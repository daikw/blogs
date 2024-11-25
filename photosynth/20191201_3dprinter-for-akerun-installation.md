# Akerunを設置するためにわざわざ3Dプリンタを買った新入webエンジニア社員の話

この記事は [https://qiita.com/advent-calendar/2019/akerun:title] の1日目の記事です。

## 誰？
初めまして、今年の一発目を飾ることになりました。

4月にRoRエンジニアとして入社した[https://qiita.com/daikw:title] です。主にサーバ・webクライアント側の開発を担当しています。

最近は情報工学と3Dプリンタに興味があります。


## 3行で
どうしてもdogfoodingをしたくて自宅にAkerunProをつけました。

サムターンの形状が少し特殊だったため、3Dプリンタを買ってスペーサを作りました。((市場にあるサムターンに合わせた形状のスペーサは、一通り用意されています。わざわざ作る必要はありません。))

たのしぃ…


## 詳しく
### Akerun Proの入手と絶望
弊社社員は、入社時の研修で渡されるAkerunPro一式をそのまま貸与されます。

個人的には、憧れのIoT代表格、スマートロック。

**これは！！！つけるしかないでしょ！！！！**

ちょうど一人暮らしを始めたところだし、新居にサクッと設置して物理鍵とオサラバしよう。


 <div class="tumblr-post" data-href="https://embed.tumblr.com/embed/post/mUspE_Tpl4ltpDKPA5-5pw/189403347301" data-did="6501d05bf463fb02e94b1026b8a19fe432fb0042"><a href="https://daikw.tumblr.com/post/189403347301/before">https://daikw.tumblr.com/post/189403347301/before</a></div>  <script async src="https://assets.tumblr.com/post.js"></script>

..., 神は死んだ。


### 3Dプリンタの入手と試し刷り
僕の部屋に神はいませんでしたが、3Dプリンタは格安で購入できることが分かりました。

造形方式はなんでもよかったのですが、

- メンテが比較的楽で ((諸説ある。FDMは安いし固体だから取り回しやすいけど、ヘッドが焦げ付きやすい。))
- [ネット上のレビューで評価が比較的高く](https://sato-ayumi.com/2019/07/11/%E3%80%90%E6%A0%BC%E5%AE%89%E3%81%AE%E5%85%89%E9%80%A0%E5%BD%A23d%E3%83%97%E3%83%AA%E3%83%B3%E3%82%BF%E3%80%91elegoo-mars-uv%E3%81%AE%E6%B5%B7%E5%A4%96%E3%83%AC%E3%83%93%E3%83%A5%E3%83%BC%E3%81%BE/)、
- それなりに安いもの

という基準で、ELEGOO MARSを選択。
[asin:B07K31VX87:detail]

樹脂はANYCUBICのグレーのやつにしました。
[asin:B07DQSRXRD:detail]

Amazonで注文すると、１週間ほどで配達。


試しに印刷してみた様子がこちら。
<figure class="figure-image figure-image-fotolife" title="印刷ステージに生えてくる様子">[f:id:photosynth-inc:20191129135441j:plain:w300]<figcaption>印刷ステージに生えてくる様子</figcaption></figure>
<figure class="figure-image figure-image-fotolife" title="洗浄後 / チェスのルーク">[f:id:photosynth-inc:20191129135529j:plain:w300]<figcaption>洗浄後 / チェスのルーク</figcaption></figure>

とても綺麗に印刷できますね。

あとサンプルモデルがイケてる。かっこいい。


### サムターンのサイズ測定
本当は一般的なご家庭にある[ノギス](https://ja.wikipedia.org/wiki/%E3%83%8E%E3%82%AE%E3%82%B9)を使えば簡単なのですが、持ち合わせがなかったので、物差しを使って測ります。

<figure class="figure-image figure-image-fotolife" title="サイズの測定">[f:id:photosynth-inc:20191129141712j:plain:w250][f:id:photosynth-inc:20191129141703j:plain:w250]<figcaption>サイズの測定</figcaption></figure>

サムターン部分のだいたいのサイズが分かれば良いです。画像から、

* 高さ: 15 mm程度
* 幅　: 30 mm程度
* 厚み: 3 mm弱

くらい

ちなみに、MIWAロックのLSPという型番らしい。
<figure class="figure-image figure-image-fotolife" title="MIWAロックのLSPという型番らしい">[f:id:photosynth-inc:20191129141722j:plain:w200]<figcaption></figcaption></figure>


### 3Dモデルの作成
サムターンとAkerunProにちゃんと嵌るように、ざっくり作るとこんな感じ。

サムターン側は、余裕を持って幅を5 mmに。

[f:id:photosynth-inc:20191129144150p:plain]


### 印刷と設置
作成した3Dモデルから、stlファイルで出力し、ELEGOO MARS用のスライサ((3Dモデルを印刷形式に変換することを、スライシングといいます。モデルをz軸方向にスライスして、層毎に印刷するため行いますが、プリント方式によって要求が異なるため、3Dプリンタメーカがそれぞれ作っています))で印刷用ファイルを作成。

印刷して取り付けると、、、？

 <div class="tumblr-post" data-href="https://embed.tumblr.com/embed/post/mUspE_Tpl4ltpDKPA5-5pw/189403397477" data-did="7b9c601363f7f8bfe1d7c7d121bd1ecbd94575b6"><a href="https://daikw.tumblr.com/post/189403397477/after">https://daikw.tumblr.com/post/189403397477/after</a></div>  <script async src="https://assets.tumblr.com/post.js"></script>

神か、、、


### しばらく使ってみると
1ヶ月ほど使ってみた後のスペーサの様子：

[f:id:photosynth-inc:20191129143554j:plain:w300]

初めは綺麗だった表面に、z軸と垂直な方向に亀裂が入っています。Additive Manufacturingの常なのかもしれませんが、レイヤ同士の結合が甘くなりがちなためです。

Akerunはかなり強いトルクを持っているので、このままだと割れてしまいそうです。設計を工夫して、応力が角に集中しないようにする工夫ができそうですね


## まとめ
すりーでぃーぷりんたすごい


---
株式会社フォトシンスでは、一緒にプロダクトを成長させる様々なレイヤのエンジニアを募集しています。
[https://hrmos.co/pages/photosynth:embed:cite]

Akerun Proの購入はこちらから
[https://akerun.com/:embed:cite]
