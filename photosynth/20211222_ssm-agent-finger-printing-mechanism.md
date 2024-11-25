# amazon-ssm-agent のハードウェアフィンガープリントの仕組み

この記事は [https://qiita.com/advent-calendar/2021/akerun:title] の 22日目の記事です。

どうもおなじみ [https://qiita.com/daikw:title] です。

Amazon Systems Manager の動作を調べているとき、「ハードウェアフィンガープリント」という言葉で厨二病を悪化させてしまいました。

つい時間をかけて調べてしまったため、記事にまとめて懺悔します。

[:contents]


# 契機
弊チームでは、raspiマシンの複製や、バックアップからの復元をすることがよくあります。

Hybrid Activation 済のマシンの復元をしようとしている最中に、ふと 「Systems Manager は混乱しないのかな ...？」と疑問に思ったことがきっかけで調べ始めました。


# 調査
概ね 抽象 -> 具体 の流れで書いてあります。あたかもこの通り綺麗に調べたように見えますが、実際にはあっちに行ったりこっちに行ったりと迷いながらやっています。探索的な調査は割と好きな daikw です。

## ドキュメント
まずは最も抽象的なドキュメントから当たって行きましょう。テクニカルリファレンスにそれっぽい記述がありました！

[SSM Agent technical reference - AWS Systems Manager#fingerprint-validation](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent-technical-details.html#fingerprint-validation) より、

> When running on-premises servers, edge devices, and virtual machines (VMs) in a hybrid environment, SSM Agent gathers a number of system attributes (referred to as the hardware hash) and uses these attributes to compute a fingerprint. The fingerprint is an opaque string that the agent passes to certain Systems Manager APIs. This unique fingerprint associates the caller with a particular on-premises managed node. The agent stores the fingerprint and hardware hash on the local disk in a location called the Vault.

> If the current machine attributes don't match the stored hardware hash, SSM Agent computes a new fingerprint, ~. This causes `RequestManagedInstanceRoleToken` to fail, and the agent won't be able to obtain a role token to connect to the Systems Manager service.
> This failure is by design and is used as a verification step to prevent multiple on-premises managed nodes from communicating with the Systems Manager service as the same managed node.


内容をざっくりまとめると、

- `hardware hash` なるものを使い、サーバ固有の `fingerprint` が計算される
- `hardware hash` と `fingerprint` は `Vault` に保管される
- `fingerprint` が `Vault` の値と一致していれば、 `RequestManagedInstanceRoleToken` なるリクエストを成功として捌いてくれる

ということがわかります。ここでさらに次のような疑問が湧きます。

- `hardware hash` と `fingerprint` の実体は何か？
- `Vault` とはどういうものか？
- 構成変更によって `fingerprint` はどの程度変わるのか？（`similarityThreshold` はどうチューンされるのか）


## フィンガープリント

[フィンガープリント（fingerprint）とは - IT用語辞典 e-Words](https://e-words.jp/w/%E3%83%95%E3%82%A3%E3%83%B3%E3%82%AC%E3%83%BC%E3%83%97%E3%83%AA%E3%83%B3%E3%83%88.html) より、

> フィンガープリントとは、指紋（を採る）、拇印という意味の英単語で、IT分野では人物や端末などの識別や同定、真正性の確認に用いられる短いデータ列などを指すことが多い。

です。

「マシンのフィンガープリントをする」というのは、「マシン同士の識別・同定が可能な情報を採取する」という意味になります。

<figure class="figure-image figure-image-fotolife">
[https://www.saitolab.org/fp_site/img/catchy3.png:image=https://www.saitolab.org/fp_site/img/catchy3.png]
<figcaption>[Fingerprint解説サイト](https://www.saitolab.org/fp_site/) より</figcaption>
</figure>


## 静的解析
実装が公開されていたので、疑問の回答を見つけることができるかもしれません。 [GitHub - aws/amazon-ssm-agent](https://github.com/aws/amazon-ssm-agent) を見ると Go で開発されていました。じっくり読んでいきましょう。

まず、 `vendor` と `extra` は vendoring されたソースコードに見えるため、除外しながら調べます。ソースコードは主に `agent` / `common` / `core` の三つのディレクトリに収められているようです。

雑に `fingerprint` でgrepすると、 [`InstanceFingerprint`](https://github.com/aws/amazon-ssm-agent/blob/mainline/agent/managedInstances/fingerprint/fingerprint.go#L54-L70) という関数が目につきました。
これは内部関数 `generateFingerprint` にキャッシュとリソースロックの機能をつけるラッパー関数で、他モジュールに対し公開されています。

```go
func InstanceFingerprint(log log.T) (string, error) {
	if isLoaded() {
		return fingerprint, nil
	}

	lock.Lock()
	defer lock.Unlock()

	var err error
	fingerprint, err = generateFingerprint(log)
	if err != nil {
		return "", err
	}

	loaded = true
	return fingerprint, nil
}
```


また、同じファイルを眺めると、先頭に核心となる構造体 [`hwInfo`](https://github.com/aws/amazon-ssm-agent/blob/mainline/agent/managedInstances/fingerprint/fingerprint.go#L36-L40) があるのが目につきます。この構造体の全てのフィールドが説明できるようになれば、当初の疑問の答えになりそうです。

```go
type hwInfo struct {
	Fingerprint         string            `json:"fingerprint"`
	HardwareHash        map[string]string `json:"hardwareHash"`
	SimilarityThreshold int               `json:"similarityThreshold"`
}
```

さて、 `generateFingerprint` を詳しく読む前に、試しに参照元の一覧を追ってみましょう。

```shell
┬─[daiki~@photosyth~:~/g/g/a/amazon-ssm-agent]─[18:41:09]─[G:mainline=]
╰─>$ rg -g !extra -g !vendor InstanceFingerprint
agent/managedInstances/registration/instance_info.go
80:     return fingerprint.InstanceFingerprint(log)

agent/managedInstances/fingerprint/fingerprint_test.go
33:func ExampleInstanceFingerprint() {
58:     val, _ := InstanceFingerprint()

agent/managedInstances/fingerprint/fingerprint.go
44:     vaultKey            = "InstanceFingerprint"
54:func InstanceFingerprint(log log.T) (string, error) {
178:            _ = log.Warnf("Could not read InstanceFingerprint file: %v", err)
```

これと同様にコードジャンプやgrepで頑張って追っていくと、 `amazon-ssm-agent` の `main`まで辿っていけます。

```
-> `fingerprint.InstanceFingerprint` (`agent/managedInstances/fingerprint/fingerprint.go`)
-> `onpremRegistation#Fingerprint` (`agent/managedInstances/registration/instance_info.go`)
-> `onpremCredentialsProvider#Retrieve` (`agent/managedInstances/registration/instance_info.go`) *ここでやや複雑なことをしている
-> `credentialsRefresher#retrieveCredsWithRetry` (`core/app/credentialrefresher/credentialrefresher.go`)
-> `credentialsRefresher#credentialRefresherRoutine` (`core/app/credentialrefresher/credentialrefresher.go`)
-> `credentialsRefresher#Start` (`core/app/credentialrefresher/credentialrefresher.go`)
-> `SSMCoreAgent#Start` (`core/app/agent.go`)
-> `start` (`core/agent.go`)
-> `run` (`core/agent.go`)
-> `main` (`core/agent_unix.go`)
```

---

`agent/managedInstances/fingerprint/fingerprint.go` に戻りましょう。`generateFingerprint` の骨子を抜き出して抜粋すると以下のようになります。

```go
func generateFingerprint(log log.T) (string, error) {
	var hardwareHash map[string]string
	var savedHwInfo hwInfo
	var err error
	var hwHashErr error

	for attempt := 1; attempt <= 3; attempt++ {
		hardwareHash, hwHashErr = currentHwHash()
		savedHwInfo, err = fetch(log)
		<omit>
	}

	<omit>

	uuid.SwitchFormat(uuid.CleanHyphen)
	new_fingerprint := ""

	if !hasFingerprint(savedHwInfo) {
		new_fingerprint = uuid.NewV4().String()
	} else if !isSimilarHardwareHash(log, savedHwInfo.HardwareHash, hardwareHash, savedHwInfo.SimilarityThreshold) {
		new_fingerprint = uuid.NewV4().String()
	} else {
		return savedHwInfo.Fingerprint, nil
	}

	updatedHwInfo := hwInfo{
		Fingerprint:         new_fingerprint,
		HardwareHash:        hardwareHash,
		SimilarityThreshold: savedHwInfo.SimilarityThreshold,
	}

	// save content in vault
	if err = save(updatedHwInfo); err != nil {
		log.Errorf("Error while saving fingerprint data from vault: %s", err)
	}
	return new_fingerprint, err
}
```

ここで `fingerprint` の正体がわかりました。ただの **`uuid`** です。
`hardwareHash` と `fingerprint` は生成タイミングが同じであること以外に共有する情報はありません。

確かに考えてみると、`fingerprint`を再計算して、一致するかを調べる必要は特になく（`hardwareHash`の比較で十分である）、これらの生成タイミングが同じであるのを保証できれば良いはずです。わかりやすい。

さらに掘っていきます。 `hwInfo` のうち、`HardwareHash` が何者なのかわかっていません。


[`agent/managedInstances/fingerprint/hardwareInfo_unix.go`](https://github.com/aws/amazon-ssm-agent/blob/mainline/agent/managedInstances/fingerprint/hardwareInfo_unix.go) に実装があります。

```go
const (
	systemDMachineIDPath = "/etc/machine-id"
	upstartMachineIDPath = "/var/lib/dbus/machine-id"
	dmidecodeCommand     = "/usr/sbin/dmidecode"
	hardwareID           = "machine-id"
)

var currentHwHash = func() (map[string]string, error) {
	hardwareHash := make(map[string]string)
	hardwareHash[hardwareID], _ = machineID()
	hardwareHash["processor-hash"], _ = processorInfoHash()
	hardwareHash["memory-hash"], _ = memoryInfoHash()
	hardwareHash["bios-hash"], _ = biosInfoHash()
	hardwareHash["system-hash"], _ = systemInfoHash()
	hardwareHash["hostname-info"], _ = hostnameInfo()
	hardwareHash[ipAddressID], _ = primaryIpInfo()
	hardwareHash["macaddr-info"], _ = macAddrInfo()
	hardwareHash["disk-info"], _ = diskInfoHash()

	return hardwareHash, nil
}


func processorInfoHash() (value string, err error) {
	value, _, err = commandOutputHash(dmidecodeCommand, "-t", "processor")
	return
}

<omit>
```

```go
func commandOutputHash(command string, params ...string) (encodedValue string, value string, err error) {
	var contentBytes []byte
	if contentBytes, err = exec.Command(command, params...).Output(); err == nil {
		value = string(contentBytes) // without encoding
		sum := md5.Sum(contentBytes)
		encodedValue = base64.StdEncoding.EncodeToString(sum[:])
	}
	return
}
```


実装から、以下のようなことがわかります。

- 特定のコマンドを実行したり、特定のファイルの値を持ってきた結果の [`md5.Sum`](https://pkg.go.dev/crypto/md5#Sum) が `hardware hash` の値となる
- `currentHwHash` の `hash` は、 [ハッシュマップ](https://ja.wikipedia.org/wiki/%E3%83%8F%E3%83%83%E3%82%B7%E3%83%A5%E3%83%86%E3%83%BC%E3%83%96%E3%83%AB) とも [チェックサム](https://ja.wikipedia.org/wiki/%E3%83%81%E3%82%A7%E3%83%83%E3%82%AF%E3%82%B5%E3%83%A0) とも取れる
- `/usr/sbin/dmidecode` を主に使っていて、`/etc/machine-id` といったファイルも使う（ちなみに [`hardwareinfo_windows.go`](https://github.com/aws/amazon-ssm-agent/blob/mainline/agent/managedInstances/fingerprint/hardwareInfo_windows.go) の場合は `wmic.exe`）


## dmidecode
[`dmidecode`](https://linux.die.net/man/8/dmidecode) は [SMBIOS - Wikipedia](https://ja.wikipedia.org/wiki/SMBIOS) を取得するための *nix 系OSに提供されるユーティリティです。

適当な EC2 サーバで叩くとこんな感じの出力がとれます。

```sh
[ec2-user@ip-172-31-47-131 ~]$ sudo dmidecode -t
dmidecode: option requires an argument -- 't'
Type number or keyword expected
Valid type keywords are:
  bios
  system
  baseboard
  chassis
  processor
  memory
  cache
  connector
  slot

[ec2-user@ip-172-31-47-131 ~]$ sudo dmidecode -t processor
# dmidecode 3.2
Getting SMBIOS data from sysfs.
SMBIOS 2.7 present.

Handle 0x0401, DMI type 4, 35 bytes
Processor Information
	Socket Designation: CPU 1
	Type: Central Processor
	Family: Other
	Manufacturer: Intel
	ID: F2 06 03 00 FF FB 8B 17
	Version: Not Specified
	Voltage: Unknown
	External Clock: Unknown
	Max Speed: 2394 MHz
	Current Speed: 2394 MHz
	Status: Populated, Enabled
	Upgrade: Other
	L1 Cache Handle: Not Provided
	L2 Cache Handle: Not Provided
	L3 Cache Handle: Not Provided
	Serial Number: Not Specified
	Asset Tag: Not Specified
	Part Number: Not Specified
```

試しに raspi で叩いてみたら、うまく動作しません。
[dmidecode - Raspberry Pi Forums](https://forums.raspberrypi.com/viewtopic.php?t=10741) に、 `It is not available on non-x86 architectures.` との記述がありました。

```sh
pi@raspberrypi:~ $ sudo dmidecode -t processor
# dmidecode 3.3
# No SMBIOS nor DMI entry point found, sorry.
```

似たような情報を得るには、`/proc` や `/sys` の情報をうまく利用する必要があるようです (([dmidecodeを使わずにCPUやメモリ、マザーボードのベンダーや型番といった情報を取得する | 俺的備忘録 〜なんかいろいろ〜](https://orebibou.com/ja/home/201706/20170626_001/) より))。

```sh
pi@raspberrypi:~ $ cat /proc/cpuinfo | grep Hardware -A10
Hardware	: BCM2835
Revision	: a020d3
Serial		: 000000009cc7694f
Model		: Raspberry Pi 3 Model B Plus Rev 1.3
```

## 複製されたraspiマシン同士の区別

複製されたraspiが区別されることは [以前の調査](https://akerun.hateblo.jp/entry/2021/12/raspi_operations_with_aws_ssm) でわかっています。

従って、 `dmidecode` が使えないとなると、raspiの区別をしている部分は別にあるようです。`agent/managedInstances/fingerprint/hardwareInfo_unix.go` をよく読むと、

```go
func diskInfoHash() (value string, err error) {
	value, _, err = commandOutputHash("ls", "-l", "/dev/disk/by-uuid")
	return
}
```

がありました。これだ！

```sh
pi@raspberrypi:~ $ ls -l /dev/disk/by-uuid
total 0
lrwxrwxrwx 1 root root 15 Dec 20 01:17 568caafd-bab1-46cb-921b-cd257b61f505 -> ../../mmcblk0p2
lrwxrwxrwx 1 root root 15 Dec 20 01:17 C839-E506 -> ../../mmcblk0p1

<omit>

pi@raspberrypi-2:~ $ ls -l /dev/disk/by-uuid
total 0
lrwxrwxrwx 1 root root 15 Dec  8 06:17 7616-4FD8 -> ../../mmcblk0p1
lrwxrwxrwx 1 root root 15 Dec  8 06:17 87b585d1-84c3-486a-8f3d-77cf16f84f30 -> ../../mmcblk0p2
```

複製を作る場合、ほぼ確実にディスクを変更することになるので、この出力が変わって区別されるのだと考えられます。


## Vault
[vaultの意味・使い方・読み方 | Weblio英和辞書](https://ejje.weblio.jp/content/vault) より、

> アーチ形天井、アーチ形天井のようなおおい、(食料品・酒類などの)地下貯蔵室、(地下)金庫室、(銀行などの)貴重品保管室、(教会・墓所の)地下納骨所

金庫室という意味があるので、暗号化や鍵のかかったイメージを持ってソースコードを読んでみましょう。[`agent/managedInstances/vault/fsvault/fsvault.go`](https://github.com/aws/amazon-ssm-agent/blob/mainline/agent/managedInstances/vault/fsvault/fsvault.go) に実装があります。

```go
func Store(key string, data []byte) (err error) {

	lock.Lock()
	defer lock.Unlock()

	if err = ensureInitialized(); err != nil {
		return
	}

	p := filepath.Join(storeFolderPath, key)

	if err = fs.HardenedWriteFile(p, []byte(data)); err != nil {
		return fmt.Errorf("Failed to write data file for %s. %v\n", key, err)
	}

	manifest[key] = p
	if err = saveManifest(); err != nil {
		delete(manifest, key)
		return fmt.Errorf("Failed to save manifest when storing %s. %v\n", key, err)
	}

	return
}
```

`fs.HardenedWriteFile` がそれらしい名前ですね。さらに追ってみましょう。 [`agent/fileutil/harden.go`](https://github.com/aws/amazon-ssm-agent/blob/mainline/agent/fileutil/harden.go#L11) を見ると

```go
const (
	RWPermission = 0600
)

// HardenedWriteFile calls ioutil.WriteFile and guarantees a hardened permission
// control. If the file already exists, it hardens the permissions before
// writing data to it.
func HardenedWriteFile(filename string, data []byte) (err error) {

	if _, err = os.Stat(filename); err != nil {
		if os.IsNotExist(err) {
			f, err := os.Create(filename)
			if err != nil {
				return fmt.Errorf("Failed to create the file, %s", err)
			}
			defer f.Close()
		} else {
			return
		}
	}

	if err = Harden(filename); err != nil {
		return
	}

	if err = ioutil.WriteFile(filename, data, RWPermission); err != nil {
		return
	}

	return
}
```

これで `harden` の意味がわかりました。ファイルパーミッションを適切に設定すること（ここでは `600`）ですね。

`saveManifest`  の方も追ってみましたが、 [`json.Marshal`](https://pkg.go.dev/encoding/json#Marshal) でシリアライズしているだけでした。

つまり `Vault` は、暗号化なしの、パーミッション `600` が維持されるただのファイルである、ということがわかりました。


## Similarity

最後に、 [`isSimilarHardwareHash`](https://github.com/aws/amazon-ssm-agent/blob/mainline/agent/managedInstances/fingerprint/fingerprint.go#L232) を、ログやコメントを一部抜粋して引用します。

```go
func isSimilarHardwareHash(log log.T, savedHwHash map[string]string, currentHwHash map[string]string, threshold int) bool {
	var totalCount, successCount int
	isSimilar := true

	// similarity check is disabled when threshold is set to -1
	if threshold == -1 {
		return true
	}

	// check input
	if len(savedHwHash) == 0 || len(currentHwHash) == 0 {
		return false
	}

	// check whether hardwareId (uuid/machineid) has changed
	// this usually happens during provisioning
	if currentHwHash[hardwareID] != savedHwHash[hardwareID] {
		isSimilar = false
	} else {
		// check whether ipaddress is the same - if the machine key and the IP address have not changed, it's the same instance.
		if currentHwHash[ipAddressID] == savedHwHash[ipAddressID] {
		} else {
			// identify number of successful matches
			for key, currValue := range currentHwHash {
				if prevValue, ok := savedHwHash[key]; ok && currValue == prevValue {
					successCount++
				}
			}

			// check if the changed match exceeds the minimum match percent
			totalCount = len(currentHwHash)
			if float32(successCount)/float32(totalCount)*100 < float32(threshold) {
				isSimilar = false
			}
		}
	}

	return isSimilar
}
```

ロジックはドキュメントそのままでしたが、以下の点が新たにわかりました。

- `threshold` は -1 ~ 100 の 整数であること
- `hash` を取得する項目の総数に占める、一致した `hash` の数の割合を一致率 (`similarity`) としていること

この `similarityThreshold` を設定できる機能は、ソースを読まないと使うの難しそうですね。ただ、使うことは滅多になさそうです。


# まとめ
冒頭の疑問に回答する形でまとめます。

- `hardware hash` と `fingerprint` の実体は、 `/usr/sbin/dmidecode` 等の出力のチェックサムである。
- `Vault` は暗号化されていない、パーミッション 600 のファイルである。
- 構成変更による `similarity` の差の出方は、アーキテクチャやOSによってかなり異なる。 `similarityThreshold`のチューンは、結構使い込んでみないと難しいと予想される。

`nmap` (( [OS 検出 | Nmap](https://nmap.org/man/ja/man-os-detection.html) より)) や `hping` (( [パケット生成が簡単にできるhpingコマンド - 無題の備忘録](https://stacktrace.hatenablog.jp/entry/2016/01/10/040353) より))で使われるような、TCP の実装差を利用した特殊な fingerprinting はなくて、割と単純な仕組みになっていることがわかりました。


# あとがき
windows と linux のビルドスイッチが見つけられなかったです。悲しい。Golang力が足りませんでした。


# 参考リンク

- [Working with SSM Agent - AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html)
- [SSM Agent technical reference - AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent-technical-details.html)

---

株式会社フォトシンスでは、一緒にプロダクトを成長させる様々なレイヤのエンジニアを募集しています。
[https://hrmos.co/pages/photosynth:embed:cite]

Akerun Proの購入はこちらから
[https://akerun.com/:embed:cite]
