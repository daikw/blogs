# Web開発チームから組込み開発チームに移った話

この記事は [https://qiita.com/advent-calendar/2020/akerun:title] の2日目の記事です。

どうも、1日目に続き[https://qiita.com/daikw:title]です。

1日目に「いつの間にかハードウェア開発に参加していた」とか書いてましたが、僕個人のモチベーションや会社からの期待値について触れておくいい機会かなと思ったので、記事にまとめようかと思います。trailblazerたれ。

# モチベーション
仕事を定義したり、何か情報を他人に伝える時は、モチベーションや契機についてのセクションを一番最初に持ってくるのが僕の最近のトレンドです。これが伝わらないとやる気も起きないし、細かい配慮などできないですよね。

さて、僕が Photosynth に入ろうと思ったのは、ハードウェアとソフトウェアのインターフェースをよりよく知るきっかけになると思ったからでした。ハードウェアもソフトウェアも両方開発しているイイ感じの企業、ないかな〜と思って偶然見つけたのがここ。最近気がついたのですが、パタヘネのサブタイがドンピシャです。

[asin:4822298426:detail]

つまり、急に組込み開発にモチベーションを持ち始めたのではなく、初めから持っていたということになります。1年半くらいかかりましたが、Web開発も楽しめたのでそれは良かったかもしれません。おかげで面白い人たちにも巡り合えました。


# キャリアについて聞かれるが
頻りに未来のことを気にされる方がいます。例えば、「ハードウェア開発をやってそのあとどうするの？」とか。ここではより良く稼ぐための方法を考えるのがキャリア戦略で、それについてどう考えているかを問われていると仮定します。

これを聞かれるとかなり回答に窮するのですが…というのも。

未来予測は元来難しいタスクであり、投資家の平均的パフォーマンスがインデックスに劣ったり [1]、

> 　投資信託ファンドは、経験豊富なうえに猛烈に働くプロフェッショナルが運用しており、彼らは巧みな売り買いを通じて、顧客のために望みうる最高の結果を達成できると考えられている。にもかかわらず、五〇年間にわたる調査の結果には議論の余地がない ── 彼らの運用成績は、ポーカーよりもサイコロ投げに近いのである。少なくとも投信ファンド三件に二件は、どの年をとっても、市場全体のパフォーマンスを下回っていた。

> 　さらに重要なのは、ファンドの運用成績は、どの年をとっても前年実績との相関性がきわめて小さく、ゼロをほんのわずか上回る程度だということである。つまり、ある年にうまくいったファンドは、ほとんど幸運のおかげなのだ。サイコロの目がよかったということである。

[政治・経済の専門家が政治経済動向の予測を全くできなかったり](https://www.theatlantic.com/magazine/archive/2019/06/how-to-predict-the-future/588040) [2] します。

>　Tetlock decided to put expert political and economic predictions to the test. With the Cold War in full swing, he collected forecasts from 284 highly educated experts who averaged more than 12 years of experience in their specialties. To ensure that the predictions were concrete, experts had to give specific probabilities of future events. Tetlock had to collect enough predictions that he could separate lucky and unlucky streaks from true skill. The project lasted 20 years, and comprised 82,361 probability estimates about the future.

>　**The result: The experts were, by and large, horrific forecasters.** Their areas of specialty, years of experience, and (for some) access to classified information made no difference. They were bad at short-term forecasting and bad at long-term forecasting. They were bad at forecasting in every domain. When experts declared that future events were impossible or nearly impossible, 15 percent of them occurred nonetheless. When they declared events to be a sure thing, more than one-quarter of them failed to transpire.

未来予測をしようとすることは良い投資とは言えないように見えます。大きな方向性だけ決めて、あとはその時々で面白そうだと思ったことに取り組むのが良いのかなと。

また、拝金駆動は一見わかりやすい動機ですが、燃料（欲望）をくべ続けること自体にエネルギーが要るなという印象があります。個人の欲望には上限があり、割となんでも揃ってしまう先進国に住んでいる人間がハングリー精神を持ち続けるのは難しい。

目の前で取り組んでいることそのものに対してモチベーションを持つことができる、好奇心に駆動されるのが、燃料供給が最も長続きする気がします。長期パフォーマンスが最も高い。

なので、細かいキャリアは考えていません。面白そうなことを一生懸命やります。


# 期待値
会社からの期待値は概ね「バックエンド・エッジ両者のインターフェースを知ることで、より見通し良く生産性の高いアーキテクチャを考案する」とかそんな感じです。サーバとエッジの両方の、特に開発者の気持ちがわかれば、みんなが幸せになるようなやり方を考えられるようになるはず…という思惑です。

「3D-CADもVerilogも電気電子物理もGitもOSもOOPも各種DBもコンテナオーケストレーションもセキュリティもwebAPIも少しずつわかります」みたいになれば、各分野のスペシャリストと協力しやすくなるかなぁと。器用貧乏になってしまうかもしれませんが。

個人と会社が両方得をする一つの例かなと感じています。誰かの参考になれば。


## 参考・引用
- [1] ダニエル カーネマン,村井 章子. ファスト＆スロー　（上） (Japanese Edition) (Kindle Locations 5264-5265). 早川書房. Kindle Edition.
- [2] [https://www.theatlantic.com/magazine/archive/2019/06/how-to-predict-the-future/588040/:title]


未来予測の不可能性については、「カオス」「複雑系」とかの単語でひっかけると面白いです。

[https://www.kyoto-su.ac.jp/project/st/st11_01.html:title]



---

株式会社フォトシンスでは、一緒にプロダクトを成長させる様々なレイヤのエンジニアを募集しています。
[https://hrmos.co/pages/photosynth:embed:cite]


Akerun Proの購入はこちらから
[https://akerun.com/:embed:cite]
