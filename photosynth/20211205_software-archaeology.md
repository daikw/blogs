# 考古学者の心得と、遺跡調査に使える道具

この記事は [https://qiita.com/advent-calendar/2021/akerun:title] の5日目の記事です。

ご無沙汰しています、ハードウェア開発チームに来たはいいものの、実は低レイヤ開発はあまりしていない [https://qiita.com/daikw:title] です。最近は主に考古学者をしていました。

Akerunサービスにおいて、ファームウェアの開発は大きなボリュームを占めるものの一つ、当然重要です。
しかし、作ったそばから陳腐化・レガシー化・遺跡化していくものばかり。

[https://twitter.com/yamada_theta/status/1219907602303176705?s=20:embed]

> 「ドキュメント？ [そんなものはない](https://dic.nicovideo.jp/a/%E3%81%9D%E3%82%93%E3%81%AA%E3%82%82%E3%81%AE%E3%81%AF%E3%81%AA%E3%81%84)」

人手があったとしても、これは避けられないのかもしれない。 (( そもそも全てが IaC化 / 仕組み化されていれば、こんなことを考える必要はない...。 弱者の論理ではある。 ))

従って、考古学の力が試されるなぁ、と感じる日々を過ごしています。
今日は記事を書きながら、遺跡調査の道具箱を整理しよう！


[:contents]


# 言葉の定義
さて、まず智慧を司るトト神の不興を買わないように、未定義語 (( [Undefined (mathematics) - Wikipedia](https://en.wikipedia.org/wiki/Undefined_(mathematics) )) を減らす努力をします。
誠意を持った祈りは届くのです 🙏 📿

- 陳腐化: 一般的には、古くなり、当初の価値を減じた状態のこと。((  [陳腐化とは - コトバンク](https://kotobank.jp/word/%E9%99%B3%E8%85%90%E5%8C%96-22407) )) 本記事で特定のソースコードやシステムに対して使った場合、誰にも読まれなくなり、誰も把握していない状態を指す。
- 遺跡・ダンジョン: 本記事では、陳腐化した状態の特定のシステム（ネットワーク・サーバ等の集合）を指す。
- 考古学・遺跡調査: 本記事では、遺跡に対しての活動全般を指す。調査をした上で、何か変更をすることが多い。 実は [Software archaeology - Wikipedia](https://en.wikipedia.org/wiki/Software_archaeology) という分野があり、伊達や酔狂とも言い切れない。
- 道具箱: 本記事では、主にコマンドラインツールの集合を指す。ヘッドレスサーバを扱うことが多いため、デスクトップアプリケーションはたまにしか出てこない。
- トト神: 知恵の神、書記の守護者、時の管理人、楽器の開発者、創造神、または森羅万象を把握しているスーパーエンジニアのこと。 / [トート - Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%88%E3%83%BC%E3%83%88) より。


# 考え方
## 考古学者の心得リスト
まず、考古学マインドセットを内面化するための、心得リストを用意します。

今にも崩れそうな遺跡に苦しむ子羊たちを導かんと、各地のトト神による [ヒエログリフ解読方法](https://qiita.com/t_nakayama0714/items/9348e215f172562974e6) や [託宣](https://yakst.com/ja/posts/3601)、[黒魔術](https://qiita.com/kanga/items/979b0bdc4653e025483a) といったコンテンツがインターネット上に存在します。

その中で、`1. 特に抽象度が高い` または `2. レビュー論文的性質がある` ものを心得リストとしました。

- [インフラエンジニアと謎のサーバ - Qiita](https://qiita.com/kanga/items/979b0bdc4653e025483a)
- [ノーヒントサーバ調査 (Linux編) - Qiita](https://qiita.com/t_nakayama0714/items/9348e215f172562974e6)
- [6万ミリ秒でできるLinuxパフォーマンス分析 | Yakst](https://yakst.com/ja/posts/3601)
- [Software archaeology - Wikipedia](https://en.wikipedia.org/wiki/Software_archaeology)
- [DevOps考古学でプロダクションを理解する](https://www.infoq.com/jp/news/2018/06/devOps-archeology/)
- [なぜ「歴史」を学ぶと未来を予測できるのか？ | 知的戦闘力を高める 独学の技法 | ダイヤモンド・オンライン](https://diamond.jp/articles/-/150373)

## 心得リストからの教訓
例えば [ノーヒントサーバ調査 (Linux編) - Qiita # 調査の心構え](https://qiita.com/t_nakayama0714/items/9348e215f172562974e6#%E8%AA%BF%E6%9F%BB%E3%81%AE%E5%BF%83%E6%A7%8B%E3%81%88) より抜粋すると、

> - サーバの今がわかる
>   - 「現在サーバがどうなっているか」に答えられるようになる
>   - 現状調査の方法 理解が必要
>     - サーバスペック / サーバ構成 / リソース状況 / ...
> - サーバの過去がわかる
>   - 「過去サーバがどうなっていたか」に答えられるようになる
>   - 主に ログの読み方 理解が必要
> - サーバの未来がわかる
>   - 「今後サーバがどうなりそうか」に答えられるようになる
>   - もちろんここは 推論 が入る

---

> - 事実と推論を分けて整理する
>     - ノーヒントであるがゆえに、当然ファーストステップは推論になるのですが、推論に推論を重ねていくと意味がわからなくなるので、推論のあとは事実確認するようなことが望ましいです
>     - そういう意味では一人でガリガリ調べ続けるよりは、別の人も巻き込んでワイワイ(?)やるのがいいですね
> - 調査事故を起こさない
>     - たまに障害調査で 二次障害 を引き起こすケースがあります
>         - 安易なroot作業や、調査作業によるリソース占有など
>     - 「今自分がなにをやろうとしているか？」についてはいつも 気をつけましょう
>     - やらかしてしまった場合に備える意味でも ターミナルログ は必ず取りましょう
>         - 多分どんなターミナルでもログは取れますが、もし取れないものを使っているとしたらそれは窓から投げ捨てましょう(ﾎﾟｲｰ


他の心得リストからも、似たような教訓が得られます。概ね、以下の2つに大別されます。

- 歴史に学べ
- 記録せよ

### 歴史に学ぶ
手がかりが本当に全くない、という状況は少なく、探索的調査によって明らかにできます（強い言葉を(ry 。

そもそも遺跡が存在していること自体がヒントになり、歴史は社内の情報源のクロールによって明らかに（きっと）できます。
歴史を知った段階で、ダンジョン化の要因を把握し取り除くことができればより好ましく、価値のある活動に。

資源配分・ITへの無理解 などの構造的問題や、単なるサボり・技術力不足といった小さい問題のこともあります。

### 記録する
これまで記録がなかったから、ダンジョン化しているわけであって、悲劇を繰り返す必要はありません。
憎しみの連鎖は自分の代で断つ、という覚悟が重要です。

記録し、公開し、騒いで、多くの目に触れさせることで、徐々に良くしていきましょう。
ただし、政治的活動や緻密な戦略が必要なことも、、、

[https://qiita.com/hirokidaichi/items/c66682a64ac2fc59cdf3:embed:cite]
[https://zenn.dev/tmknom/articles/93f227ad5e55aa:embed:cite]


# 道具
[https://dic.nicovideo.jp/a/%E3%81%9D%E3%82%93%E3%81%AA%E8%A3%85%E5%82%99%E3%81%A7%E5%A4%A7%E4%B8%88%E5%A4%AB%E3%81%8B%3F:embed:cite]

「歴史に学」び、「記録」するために、十分リッチな道具を用意します。各記事からざっくり抜粋し、まとめました。

記載のコマンドは全て、manページをざっくり読み、オプションを含めた概要を掴んでおくことが推奨されます。
といってもdistro間の細かい違いまで抑えるのはやはり困難です。都度、必要な道具を探し、見つけ、学ぶことが肝要そう。

> 「一番いいのを頼む」

```sh
# ターミナル操作ログを残す。自動で残るようにしておくと便利。
script $file_name

# 外部から、又は外部環境ごと調べる
dig
nslookup
ping
traceroute
nmap
telnet
arp
nc
curl
wget

# 証跡やログ
history
last
journalctl
ls /var/log
less /var/log/messages

# リソース
uptime

cat /proc/cpuinfo
cat /proc/meminfo

dmesg | tail

free -mh
df -h
vmstat 1 5

[h]top

## `sysstat` が必要
mpstat -P ALL 1
pidstat 1
iostat -xz 1
sar -n DEV 1
sar -n TCP,ETCP 1

# ペリフェラル
ls -l /dev

# 固有情報
uname -a
lsb_release
ls /etc/*release
cat /etc/os-release

# プロセス
ps -A o user,command --no-header --sort command | grep -v -e '\s\['
ps aux[f]
pgrep $name
strace -p [-f] $pid [-e trace=$type]

# Cron / Daemon
ls /var/spool/cron/
ls /etc/crontab/
ls /etc/cron.d/
ls /etc/cron.hourly/
ls /etc/cron.daily/
ls /etc/cron.weekly/
ls /etc/cron.monthly/

systemctl list-units --type=service
systemctl list-unit-files --type=service

# ネットワーク
ip a
ip a s $nic
netstat -lntup
ss

# パッケージリスト
dpkg -l
dpkg-query -l
zgrep install /var/log/dpkg.log*

apt list --installed
zgrep Commandline /var/log/apt/history.log.*

comm -23 <(apt-mark showmanual | sort -u) <(gzip -dc /var/log/installer/initial-status.gz | sed -n 's/^Package: //p' | sort -u)
### ref: https://askubuntu.com/questions/2389/how-to-list-manually-installed-packages

rpm -qa --last
yum list installed

snap list
flatpak list

# 特徴的なファイルを見つける・中身を探る
du -m / --max-depth=3 --exclude="/proc*" | sort -k1 -n -r
find /etc -maxdepth 3 -type f -printf "%p %TY-%Tm-%Td\n" | sort -k2 -r
find /home/pi/ -type f -exec du -sm {} \; | sort -nrk1 | head
find /var/log/ -type f -exec du -sm {} \; | sort -nrk1 | head

file $file_name
stat $file_name
```

### 参考リンク
- [UbuntuとDebianでインストール済みパッケージ一覧を表示する方法 | TECH+](https://news.mynavi.jp/article/20190222-775519/)
- [apt - How to list manually installed packages? - Ask Ubuntu](https://askubuntu.com/questions/2389/how-to-list-manually-installed-packages)
- [【Mac】ターミナル操作をログに残す方法 - 会社辞めてニートになった元プログラマーの雑記帳](https://hira98.hatenablog.com/entry/2019/05/29/190700)
- [strace コマンドの使い方をまとめてみた - sonots:blog](http://blog.livedoor.jp/sonots/archives/18193659.html)
- [psコマンドについて詳しくまとめました 【Linuxコマンド集】](https://eng-entrance.com/linux-command-ps#aux)
- [tcpdumpコマンドで覚えておきたい使い方4個 | 俺的備忘録 〜なんかいろいろ〜](https://orebibou.com/ja/home/201505/20150525_001/)
- [sar(sysstat)によるボトルネック特定 - Qiita](https://qiita.com/kidach1/items/07637a5baa0da7d52e6a)
- [いまさら聞けないLinuxとメモリの基礎＆vmstatの詳しい使い方 - Qiita](https://qiita.com/kunihirotanaka/items/70d43d48757aea79de2d)

---

株式会社フォトシンスでは、一緒にプロダクトを成長させる様々なレイヤのエンジニアを募集しています。
[https://hrmos.co/pages/photosynth:embed:cite]

Akerun Proの購入はこちらから
[https://akerun.com/:embed:cite]
