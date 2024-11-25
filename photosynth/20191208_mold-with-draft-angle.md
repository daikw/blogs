# 金型用の3Dモデリングについてのうんちくと、抜き勾配

この記事は [https://qiita.com/advent-calendar/2019/akerun:title] の8日目の記事です。

最近は情報工学と3Dプリンタに興味がある、RoRエンジニアの[https://qiita.com/daikw:title] です。

今日は、金型用の3Dモデルを作るときには、3Dプリンタでの開発とは違う感覚が必要になる、ということの話をします。

## 射出成形と3Dプリンタとの違い
いわゆるものづくりでは、使用したい物質を、必要十分な精度で、目的の形状に成型する必要があります。弊社でも、その「いわゆるものづくり」を行っています。

特によく使われる素材であるプラスチックについては、昔から成型するための手法がたくさん考案されてきました。以下の二つの手法は皆さんもよく聞くのではないでしょうか。

- 金型を用いた射出成型
- 3Dプリンタによる造形（積層造形）

射出成型は古典的な手法の一つで、特に大量生産に向いていると言われます。積層造形はここ何年も盛り上がっている手法であり、個人でもプリンタが購入できるようになってからは馴染みやすいものになりました。

ここで設計者が注意すべきことがあります。それは、<b>成型手法によって物理的な制約が全く異なるため、モデルのデザイン時にすでに成型手法のことを考慮に入れる必要がある</b>、ということです。

私のような初心者メカジニアは3Dプリンタの方が馴染み深かったりするので、気がつくと3Dプリンタの制約に基づいてモデリングをしていたりしますが、金型に起こすためのモデルは全く違う方針をとることがあります。

### 3Dプリンタの制約
「積層造形」という単語からわかるかもしれませんが、3Dプリンタはモデルを高さ方向にスライスし、それを積み上げていきます。この積み上げ時に、宙ぶらりんになる部分がどうしても出てきてしまいます。

[https://i-maker.jp/blog/wp-content/uploads/2019/06/support-1.png:image=https://i-maker.jp/blog/wp-content/uploads/2019/06/support-1.png] ((https://i-maker.jp/blog/support-material-11673.html より引用))
図のように、宙ぶらりんの部分はサポート材で支えてプリントすることになります。

サポート材と触れる部分は、表面状態が変わってくることがあるため、この点に注意して設計する必要があります。

精度によっては、複数の部品に分けた方がいい場合もありますし、印刷の向きを変えて対応する場合もあります（例えばT字型なら、天地をひっくり返せばサポート材はいらなくなりますね）。

[https://i-maker.jp/blog/support-material-11673.html:embed:cite]

### 金型の制約
金型による射出成型の原理は3Dプリンタよりも単純である一方で（あるいは単純であるがゆえに）、制約は3Dプリンタより複雑です。

金型は、キャビティとコアと呼ばれる二つの金型をセットにして用います。(( [https://minsaku.com/category01/post217/?gkey=gb%E5%B0%84%E5%87%BA%E6%88%90%E5%BD%A2&OV_REFFER=&gclid=EAIaIQobChMI4IDZ-uSi5gIVBqyWCh1ExQMeEAAYASAAEgKYLfD_BwE:title] より))

[https://img.minsaku.com/wp-content/uploads/2019/02/07144123/217-2.png:image=https://img.minsaku.com/wp-content/uploads/2019/02/07144123/217-2.png]

さらに射出成型のプロセスを分解すると、 (([https://www.polyplastics.com/jp/support/mold/outline/:title] より))

[https://www.polyplastics.com/jp/support/mold/outline/inj.gif:image=https://www.polyplastics.com/jp/support/mold/outline/inj.gif]

これを単純化すると、『「二つの金型の間に、熱して溶かしたプラスチックを注入し、冷まして固化させて、取り出す」を繰り返す』必要があります。
これらの工程ごとに、金型への制約を考えることができ、

<b>二つの金型の間に樹脂を流す</b>ことから、

- 部品の一方の面はキャビティに触れていて、反対の面はコアに触れている
- 金型の分離面（parting-line）が存在して、これが部品に触れている

<b>熱して溶かしたプラスチックを注入する</b>ことから、

- 溶融プラスチックが隅々まで綺麗に流れる形状である（流路が複雑すぎない）

<b>冷まして固化させる</b>ことから、

- なるべく全体が均一な厚みである（一部だけが厚いと、離型前に冷えきらず、離型後に冷えて凹んでしまう）

<b>取り出す</b>ことから、

- 金型から十分に剥がれやすい（離型性）


## 金型用3Dモデルの設計時に気をつけるべきこと
制約について見てきましたが、金型用の3Dモデリングで3Dプリンタと違って気をつけるべきことを大雑把にまとめると、

- parting-lineの存在
- 均一な厚み
- 離型性

特に離型性は、二つの性質に分ける事ができて、

- そもそも剥がれうる形状である ~= 横方向の凸・凹形状（アンダーカット）がない
- 剥がれやすい形状である ~= 抜き勾配が十分にある

となります。

## 抜き勾配の付け方
弊社では、3D-CADソフトはFusion360を使っています。こやつは無料でも使用できるのに高機能であり、プロジェクトやファイルの共有もクラウド管理できる、便利なやつです。

最近Generative Designで何かと話題ですが、残念ながら弊社ではそんなキラキラした機能は使っていません（）。

さて、Fusionには抜き勾配（draft）機能は存在します。が、あまり便利ではありません。。。

[https://cad-kenkyujo.com/reference/model/modify/%E5%8B%BE%E9%85%8D/:embed:cite]

これは何故かというと、

- 絶対値で指定できない（元あった面からの相対的な傾きになる）
- parting-lineとしての指定ができない（基準面はある ((Fusion360には存在しないと明言されています。<br>[https://forums.autodesk.com/t5/fusion-360-design-validate/does-parting-line-function-exist-in-f360-like-solidworks/td-p/8082779:title]))
- 他のFeatureと衝突しやすい

ためです。

任意のdraftをつけるためには、自前で平面を作るしかなく、Fusion360の不便な点ではあります。
[https://forums.autodesk.com/t5/fusion-360-ri-ben-yu/gou-peinotsukekatanitsuite/td-p/8703278:embed:cite]


### 他の3D-CADだったら？
ハイエンド（数百万円くらい）の3D-CADはもちろん複雑な抜き勾配featureが用意されていますし、他にも意味のわからない機能が大量に付いています。

- [http://help.solidworks.com/2017/japanese/SolidWorks/sldworks/HelpViewerDS.aspx?version=2017&prod=SolidWorks&lang=japanese&path=sldworks%2ft_drafts_overview.htm:title]
- [Creating Drafts with Parting Elements | CATIADOC](http://catiadoc.free.fr/online/prtug_C2/prtugbt0603.htm)
- [金型設計と鋳造 > モールドおよび鋳造フィーチャーの作成 > ドラフトカーブと分離カーブ > ドラフト環境を定義するには | Creo Parametric 5.0.5.0 オンラインヘルプ](http://support.ptc.com/help/creo/creo_pma/japanese/#page/mold_and_casting%2Fmold%2FTo_Define_the_Draft_Environment.html)


同じような価格帯の3D-CADの中でも、OnShapeはparting-line指定ができるようになっています。

- [Draft | Onshape Help](https://cad.onshape.com/help/Content/draft.htm)



## 最後に
自分の勉強がてら、金型と樹脂成形についてまとめました。

大量生産の手法はそれ自体が興味深く、プロダクトの設計・開発とはまた違った楽しみがあるなと感じました。

今は、10万円程度で金型に起こしてくれるサービス((https://www.protolabs.co.jp/))も出てきています。お試しあれ。



---

株式会社フォトシンスでは、一緒にプロダクトを成長させる様々なレイヤのエンジニアを募集しています。
[https://hrmos.co/pages/photosynth:embed:cite]


Akerun Proの購入はこちらから
[https://akerun.com/:embed:cite]
