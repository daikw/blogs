# 会社で遊んでいたForm3を使えるようにするまで

この記事は [https://qiita.com/advent-calendar/2020/akerun:title] の1日目の記事です。

実に１年ぶりの更新になってしまいました。

低レイヤに興味がある故にいつの間にかハードウェア開発に参画していた[https://qiita.com/daikw:title] です。


今日は、2020/01頃に購入したものの、設置環境の問題でしばらく眠っていた光造形式の3Dプリンタを使えるようにした話をします。


# モチベーション
現弊社は[G-BASE田町](https://www.shimz.co.jp/shimzdesign/topics/gbase.html)というオフィスビルに入居しています。

<figure class="figure-image figure-image-fotolife" title="https://www.shimz.co.jp/shimzdesign/topics/gbase.html より引用">[f:id:photosynth-inc:20201125135040p:plain]<figcaption>https://www.shimz.co.jp/shimzdesign/topics/gbase.html より引用</figcaption></figure>

とても先進的なビルでめちゃくちゃ良い物件なのですが、弊社のハードウェアチームが占拠しているパーティションには開放できる窓がなく、換気はダクトを介して行う必要があります。

その中で光造形式の3Dプリンターを使うのは憚られていました（<s>なぜ買ったのか</s>）。と言うのも、

- 洗浄用のアルコールはかなり揮発が激しいこと
- 液体の印刷用樹脂の臭いがそこそこすること
- 印刷用樹脂でカーペットを（かなり）汚しそうなこと

そこで、簡易なドラフトチェンバーをDIYし、なるべくその中で作業をするような環境を作りました。

# Form3・Form Washと光造形
Form3は、ミドルエンドの光造形方式の3Dプリンタです。いろいろ難点もありますが、かなり綺麗に印刷ができ、樹脂の種類も豊富に選べて便利です。

[https://formlabs.com/3d-printers/form-3/:embed:cite]

光造形方式全般に言えることですが、印刷直後のモデルは液体樹脂でベッタベタなのでよく洗浄する必要があります。

[https://ja.wikipedia.org/wiki/%E5%85%89%E9%80%A0%E5%BD%A2%E6%B3%95:title]

Formlabは洗浄機や二次硬化のための機材も作っていて、弊社でも利用しています。

[https://formlabs.com/wash-cure/:embed:cite]


プリンタと洗浄機には常に液体が入っている状態で運用するため、これらはチェンバーに設置する想定で作ります。


# 設計
大掛かりなものや、DIYでアクリル細工をしたりしたことは全然なかったのですが、必要なものは何で、何を作ればいいのかはソフトウェアの設計と大して変わりません。弊社メカジニアたちと相談しながら設計していきます。要件は洗い出せているので、デザインと寸法を考えて材料を探していきます。

<figure class="figure-image figure-image-fotolife" title="ドラフトチェンバーの設計図">[f:id:photosynth-inc:20201125173637j:plain]<figcaption>ドラフトチェンバーの設計図</figcaption></figure>

## レシピ
アクリル板を発注したり、細かい部品を揃えたり…

- [はざいや](https://www.hazaiya.co.jp/)で発注したアクリル板数枚

- 換気ダクトに接続できるフレキシブルダクト

- [https://www.monotaro.com/g/04209452/:title]（これじゃないですが、こんな感じの）

- [https://www.askul.co.jp/p/3658366/:title]

- [https://www.askul.co.jp/p/E082888/:title]

- [https://jp.misumi-ec.com/vona2/detail/223000811265/:title]

- [アイリスオーヤマの突っ張り棒(https://www.amazon.co.jp/dp/B000S6KSOO)

- その他ワッシャーやボルト・ナットなどこまごま


##  成果
組み立て作業はそこそこ大変でしたが写真が残っておらず…

完成 🎉

<figure class="figure-image figure-image-fotolife" title="ドラフトチェンバー完成（開）">[f:id:photosynth-inc:20201125173144j:plain]<figcaption>ドラフトチェンバー完成（開）</figcaption></figure>
<figure class="figure-image figure-image-fotolife" title="ドラフトチェンバー完成（閉）">[f:id:photosynth-inc:20201125173015j:plain]<figcaption>ドラフトチェンバー完成（閉）</figcaption></figure>


アクリル板の横から生えている筒が換気ダクトに接続されていて、空気を吸引しています。
空気を吸っているので、カーテンが少し内側に引っ張られているのがわかりますね。

またスチールラックの棚は水平出しが難しいため、3Dプリンタを置くスペースとしては不適当です。従って適当な足の長さの調節できるテーブルを中に含めた設計になっています。

# 振り返って
メカジニア氏はこのチェンバーのすぐ隣で作業をしているので、アルコールをそれほど吸わずにすむようになって良かったとのことでした 🎉🎉

<br />

想像して、それを実現するというのはどういう分野であっても大きく変わるものではないな、と思った私でした。


---

株式会社フォトシンスでは、一緒にプロダクトを成長させる様々なレイヤのエンジニアを募集しています。
[https://hrmos.co/pages/photosynth:embed:cite]


Akerun Proの購入はこちらから
[https://akerun.com/:embed:cite]
