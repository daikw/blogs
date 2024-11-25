# Go で Intel-HEX フォーマットを扱うパッケージ

この記事は [https://qiita.com/advent-calendar/2022/akerun:title] の 8 日目の記事です。

どうも、 [https://qiita.com/daikw:title] です。ソフトウェアエンジニアをしています。

今年は Go 言語を扱う機会が少しあったので、[前回の記事では Go 言語の BLE 関連パッケージも調べました](https://akerun.hateblo.jp/entry/2022/12/draw_repository_fork_graph) が、今日は Intel HEX フォーマットのデータを扱うパッケージを調べてみます。

[:contents]

# 結論

- 公開されているパッケージの中では [marcinbor85/gohex](https://pkg.go.dev/github.com/marcinbor85/gohex) が最も良い。
- [unixdj/ihex](https://pkg.go.dev/github.com/unixdj/ihex) も悪くはない。
- 単純な実装なので再発明しても良い。

# Intel HEX フォーマット

そもそも Intel HEX とはなんでしょうか？ 最近話題の ChatGPT くんに泣きついてみましょう。

[f:id:photosynth-inc:20221206221009p:plain]

ふんふん、非常にわかりやすい。エンジニアに聞くよりわかりやすいぞ。プログラムをプログラムするのか、なるほどなるほど。

で、それはどうやって Go 言語で扱うのかな？

[f:id:photosynth-inc:20221206221122p:plain]

うんうん、ファイルの入出力のやり方から教えてくれるのは丁寧だ。僕らはもう廃業かな。

[f:id:photosynth-inc:20221206221142p:plain]

うーん、「データの解析」のところが単純すぎる実装だなぁ。Go のデータ型としてどう扱ったらいいかな？

[f:id:photosynth-inc:20221206221159p:plain]

そっかー、レコードタイプも扱いたいんだけど、どうかな ...？

[f:id:photosynth-inc:20221206221414p:plain]

[f:id:photosynth-inc:20221206221811p:plain]

[f:id:photosynth-inc:20221206221824p:plain]

おおっ、なんかいい感じだ！ありがたい。それじゃ最後に、アドレスもうまく扱えたりしないかな ...？

[f:id:photosynth-inc:20221206221222p:plain]

## 助けて wiki えもん

まだしばらくは廃業しなくて済みそうでしたので、こういう時は普通に調べましょう。 [Intel HEX - Wikipedia](https://ja.wikipedia.org/wiki/Intel_HEX) より、

> Intel HEX はバイナリ情報を ASCII テキスト形式で記載したファイル形式である。マイクロコントローラや EPROM などのプログラム可能なデバイスのプログラム書き込みのために広く用いられている。

> 典型的な利用用途としてはコンパイラやアセンブラがプログラムの C 言語やアセンブリ言語などのソースコードを機械語に変換し、HEX ファイルとして出力する。

> HEX ファイルは ROM にマシン語のコードを「焼く」ために書き込み機によって読み込まれたり、対象のシステムで読み込んだり実行したりするために転送されたりする。

ああ〜、 ChatGPT くんの言う「プログラムをプログラムする」は、マシン語のコードを（プログラムを）焼く（プログラムする）ことかもしれないですね。

実際に [J-Link や Jeff Probe などを利用してマイコンに焼く](https://akerun.hateblo.jp/entry/2021/12/16/JeffProbe) ときに、この形式のファイルを使っています。

> 各行は複数の 2 進数値をエンコードする 16 進数の文字を含む。2 進数の値は行の位置や形式、長さによってデータ、メモリアドレスなどに相当する。各行はレコードと呼ばれる。

> レコード(テキストの行)は左から順に並んだ 6 つのフィールド（部分）を有する

データの構造は以下のようになり、6 種類のレコードタイプを扱う必要はありますが、比較的シンプルな実装のパーサで扱えそうですね。探索も実装も楽そうに見えます。

<figure class="figure-image figure-image-fotolife" title="wikiえもんより">[f:id:photosynth-inc:20221206222240p:plain]<figcaption>wikiえもんより</figcaption></figure>

# パッケージの選定

## 探索

さて、一通り理解が進んだところで、先週 BLE 通信パッケージを探索した時と同様に [pkg.go.dev](https://pkg.go.dev/) で探してみましょう。

関連キーワードは BLE ほどたくさん思いつかないので、 `intel hex` で検索（[intel hex - Search Results - pkg.go.dev](https://pkg.go.dev/search?q=intel+hex&m=package)）して、インターフェースの異なるパッケージが 6 つ 見つかりました。

なお [unixdj/ihexcat](https://pkg.go.dev/github.com/unixdj/ihexcat#section-readme) は選定から除きます。

## 静的比較

パッケージ比較をロジカルにやってみましょう。ざっくり利用する基準を考えてみると、

- ihex ファイルのアドレスを指定して扱う機能があること
- 読み書きの機能があること
- テストコードがあること
- 他のパッケージから利用されていること

といったところでしょうか。

では `pkg.go.dev` から取得できる情報と、ドキュメント・ソースコードをさっと見た時に得られる情報をまとめてみます。

| package                                                                        | published    | license      | imported_by | testcode |
| ------------------------------------------------------------------------------ | ------------ | ------------ | ----------- | -------- |
| [zellyn/go6502/asm/ihex](https://pkg.go.dev/github.com/zellyn/go6502/asm/ihex) | Sep 3, 2018  | GPL-3.0      | 0           | あり     |
| [tejainece/ihex](https://pkg.go.dev/github.com/tejainece/ihex)                 | Jul 12, 2016 | BSD-3-Clause | 0           | あり     |
| [marcinbor85/gohex](https://pkg.go.dev/github.com/marcinbor85/gohex)           | Mar 8, 2021  | MIT          | 20          | あり     |
| [edmccard/ihex](https://pkg.go.dev/github.com/edmccard/ihex)                   | Apr 25, 2015 | MIT          | 1           | あり     |
| [unixdj/ihex](https://pkg.go.dev/github.com/unixdj/ihex)                       | Jan 18, 2022 | ISC          | 1           | なし     |
| [littlehawk93/ihex](https://pkg.go.dev/github.com/littlehawk93/ihex)           | Jan 24, 2020 | MIT          | 0           | なし     |

その他のメモも書き下します ((はてなブログの表は横スクロールにするのが面倒。参考: [【はてなブログ】Markdown 記法で表を作成する方法 | AY3 の 6 畳細長部屋](https://www.ay3s-room.com/entry/hatena-markdown-table)))。

- `zellyn/go6502/asm/ihex`
  - MOS 6502 という CPU を扱うパッケージの一部
  - `Writer` はあるが `Reader` がない
- `tejainece/ihex`
  - アドレス指定で取り出せない
- `marcinbor85/gohex`
  - テストコードが最もちゃんと書いてある
  - 利用例がある
- `edmccard/ihex`
  - 1 レコードずつ読む・メタ情報を読むには良さそう
  - アドレス指定できない
- `unixdj/ihex`
  - ドキュメントが最も丁寧
  - テスト・利用例がなく扱いにくい
- `littlehawk93/ihex`
  - `coming soon` など書いてある割にメンテされた様子がなく、ややスメリー
  - 使えそう感は出ている

以上のことから、基準に合致するのは `marcinbor85/gohex` のほぼ一択ですね。ギリギリ良さそうな `unixdj/ihex` も、ついでに参考実装を書いてみましょう。

## 動的比較: 参考実装

以下の要件で実装してみます。

- アドレス指定でファイルに書き込むこと
- ファイルを読み込んで処理すること
- 共通の以下の実装を持つこと

```go
func cat(filepath string) error {
	data, err := os.ReadFile(filepath)
	if err != nil {
		return err
	}

	fmt.Println("----------------------------------------------------")
	os.Stdout.Write(data)
	fmt.Println("----------------------------------------------------")

	return nil
}

func main() {
	filepath := "output.hex"
	if err := dump(filepath); err != nil {
		panic(err)
	}
	if err := read(filepath); err != nil {
		panic(err)
	}

	cat(filepath)
}
```

### `marcinbor85/gohex`

こちらは参考実装がレポジトリにもあるので、さっと書けそうです。

https://go.dev/play/p/NMgScee-1ZX

```go
func dump(filepath string) error {
	file, err := os.Create(filepath)
	if err != nil {
		return err
	}
	defer file.Close()

	mem := gohex.NewMemory()
	mem.SetStartAddress(0x80008000)
	mem.AddBinary(0x10008000, []byte{0x01, 0x02, 0x03, 0x04})
	mem.AddBinary(0x20000000, make([]byte, 256))
	mem.AddBinary(0x30008000, []byte{0x04, 0x03, 0x02, 0x01})

	mem.DumpIntelHex(file, 16)

	return nil
}

func read(filepath string) error {
	file, err := os.Open(filepath)
	if err != nil {
		return err
	}

	mem := gohex.NewMemory()
	if err := mem.ParseIntelHex(file); err != nil {
		return err
	}
	for i, segment := range mem.GetDataSegments() {
		fmt.Printf("%v: {Address: 0x%00000000x, Data: %v}\n", i, segment.Address, segment.Data)
	}
	bytes := mem.ToBinary(0xFFF0, 128, 0x00)
	fmt.Printf("b: %v\n", bytes)

	return nil
}
```

割とわかりやすいんじゃないでしょうか。

```
0: {Address: 0x10008000, Data: [1 2 3 4]}
1: {Address: 0x20000000, Data: [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]}
2: {Address: 0x30008000, Data: [4 3 2 1]}
b: [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
----------------------------------------------------
:0400000580008000F7
:020000041000EA
:048000000102030472
:020000042000DA
:1000000000000000000000000000000000000000F0
:1000100000000000000000000000000000000000E0
:1000200000000000000000000000000000000000D0
:1000300000000000000000000000000000000000C0
:1000400000000000000000000000000000000000B0
:1000500000000000000000000000000000000000A0
:100060000000000000000000000000000000000090
:100070000000000000000000000000000000000080
:100080000000000000000000000000000000000070
:100090000000000000000000000000000000000060
:1000A0000000000000000000000000000000000050
:1000B0000000000000000000000000000000000040
:1000C0000000000000000000000000000000000030
:1000D0000000000000000000000000000000000020
:1000E0000000000000000000000000000000000010
:1000F0000000000000000000000000000000000000
:020000043000CA
:048000000403020172
:00000001FF
----------------------------------------------------

Program exited.
```

出力も予想通りになりました。

### `unixdj/ihex`

こちらは参考実装がないので少し苦労しました。

選定から除いた [unixdj/ihexcat](https://pkg.go.dev/github.com/unixdj/ihexcat#section-readme) で 利用されていた (https://github.com/unixdj/ihexcat/blob/v1.0.1/main.go)ので、これで雰囲気を掴みつつ書いてみます。

https://go.dev/play/p/qLSFWVED-hm

```go
func dump(filepath string) error {
	file, err := os.Create(filepath)
	if err != nil {
		return err
	}
	defer file.Close()

	ix := ihex.IHex{
		Format: ihex.Format32Bit,
	}

	ix.Chunks = []ihex.Chunk{
		ihex.Chunk{
			Addr: 0x10008000,
			Data: []byte{0x01, 0x02, 0x03, 0x04},
		},
		ihex.Chunk{
			Addr: 0x20000000,
			Data: make([]byte, 256),
		},
		ihex.Chunk{
			Addr: 0x30008000,
			Data: []byte{0x04, 0x03, 0x02, 0x01},
		},
	}

	if err := ix.WriteTo(file); err != nil {
		log.Fatal(err)
	}

	return nil
}

func read(filepath string) error {
	file, err := os.Open(filepath)
	if err != nil {
		return err
	}

	var ix ihex.IHex
	ix.ReadFrom(file)

	for i, chunk := range ix.Chunks {
		fmt.Printf("%v: {Address: 0x%00000000x, Data: %v}\n", i, chunk.Addr, chunk.Data)
	}

	return nil
}
```

`Chunk` という表現に違和感がなければ、割とわかりやすいように見えますね。

```
0: {Address: 0x10008000, Data: [1 2 3 4]}
1: {Address: 0x20000000, Data: [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]}
2: {Address: 0x30008000, Data: [4 3 2 1]}
----------------------------------------------------
:020000041000EA
:048000000102030472
:020000042000DA
:1000000000000000000000000000000000000000F0
:1000100000000000000000000000000000000000E0
:1000200000000000000000000000000000000000D0
:1000300000000000000000000000000000000000C0
:1000400000000000000000000000000000000000B0
:1000500000000000000000000000000000000000A0
:100060000000000000000000000000000000000090
:100070000000000000000000000000000000000080
:100080000000000000000000000000000000000070
:100090000000000000000000000000000000000060
:1000A0000000000000000000000000000000000050
:1000B0000000000000000000000000000000000040
:1000C0000000000000000000000000000000000030
:1000D0000000000000000000000000000000000020
:1000E0000000000000000000000000000000000010
:1000F0000000000000000000000000000000000000
:020000043000CA
:048000000403020172
:00000001FF
----------------------------------------------------

Program exited.
```

出力は予想通りで、 `marcinbor85/gohex` と同じになりました。

# その他: J-Link で他のフォーマットを扱う

https://wiki.segger.com/J-Link_Commander#LoadFile より、 J-Link では以下の形式のファイルが扱えます。

```
*.mot
*.srec
*.s37
*.s19
*.s
*.hex
*.bin
```

`mot`, `srec`, `s37`, `s19`, `s` はいずれも `Motorola S-record` というフォーマットで、 Intel HEX と同じような仕組みを持っています。
先頭が `S` で始まる点が違いますが、アドレス・データ・チェックサムを扱える点は同じです。

# 参考記事

- `ja.wikipedia` より `en.wikipedia` の記事の方が参考文献へのリンクが多い: https://en.wikipedia.org/wiki/Intel_HEX
- http://www.piclist.com/techref/fileext/hex/intel.htm
- [S-record - Wikipedia](https://ja.wikipedia.org/wiki/S-record)

---

株式会社フォトシンスでは、一緒にプロダクトを成長させる様々なレイヤのエンジニアを募集しています。
[https://hrmos.co/pages/photosynth:embed:cite]

Akerun Pro の購入はこちらから
[https://akerun.com/:embed:cite]
