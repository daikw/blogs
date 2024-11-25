# 施解錠遅延の限度見本

この記事は [https://qiita.com/advent-calendar/2021/akerun:title] の 12日目の記事です。

最近は越境活動が板についてきた [https://qiita.com/daikw:title] です。

部署や役職を跨いで共通の認識を作ることで、プロジェクトが円滑に回る、というのはあるあるです。
今日は、この共通認識を作るために造ったものを、一つ取り上げます。


[:contents]


# 結論
3行でまとめると、

- 「包括的なドキュメントよりも動くソフトウェアを」の実践は、越境的活動に良い効果があった。
- 生産現場では、「動くソフトウェア」に近い概念の「限度見本」を使っている。
- ngrok は良い、vercel も良い。


# 限度見本とは

僕はソフトウェア技術者でして、なるべくテストコードを書きながら開発しています。
いちいち手動でテストするのは面倒だけど、作ったものが意図通りに動くことは確かねばならないわけでして、
つまり **ソフトウェアの品質** を保つために、テストコードを書いて自動で走らせます。

一方で、製造した **製品の品質** を保つための活動として、検査治具を設けたり、外観検査をしたりします。
外観検査ではヒトの五感を使うため、感覚によるバラツキが発生しないようなわかりやすい判定基準を設けるのが望ましく、基準となるサンプルを用意して比較対象とします。

この基準となるサンプルを、「限度見本」と一般的に呼んでいるようです。
いくつかweb上のコンテンツから引用すると、

[外観検査を始める前に | 外観検査の基本 | 外観検査.com | キーエンス](https://www.keyence.co.jp/ss/products/vision/visual-inspection/basic/standard.jsp) によれば、

> 目視による外観検査は、人間に依存する割合が高く、ヒューマンエラーや検査員によるばらつきが発生しやすい工程です。それらを防ぐためには、良品と不良品の境界線を明確に決めておくことが大切です。
> 仕様書・検査基準書のような文字と写真ではなく、合否の判定基準を目で見てわかるようにする「限度見本」

[限度見本の意味と定義とは｜限度見本を英語でいうと](https://www.toishi.info/car/gendomihon.html) によれば、

> ある製品を品質上合格とするか、不合格とするかの限度を示した製品見本
> boundary sampleやlimit sampleもしくはcriteria sampleといった表現がよく使われます。


# ソフトウェア限度見本

つまり限度見本は、「検査員」と「設計者・技術者」との間のコミュニケーションツールとして機能しています。
しかも文字ベースの情報伝達と比べて、ディティールを書き下す・読む必要がなく、とても低コストです。

ここで [アジャイルソフトウェア開発宣言](https://agilemanifesto.org/iso/ja/manifesto.html) より、「包括的なドキュメントよりも動くソフトウェアを」を引用しましょう。

御託を並べるより、動くものをスパッと見せてやれば、コミュニケーション齟齬は一発で無くなったりするものです。

よってここでは、「限度見本」の定義を少し緩和して、「ソフトウェア限度見本」を

「品質を敢えて落とした製品・サービスであって、品質基準の検討のための見本となるもの」

と定義し、それを満たすようなものを作ってみました。


# スコープ・要件

ここでは試みとして、弊社スマートロックサービスの「施解錠速度」の品質基準検討に使えるようなものを目指しましょう。

要件をざっくり洗い出し、実装中に目標を見失わないようにします。

- Akerunに遠隔でコマンドを送信する
  - コマンドの種類
    - 施錠
    - 解錠
    - 状態取得
  - コマンドの性質
    - コマンドの実行完了を検知できる
    - コマンドの予定の実行時間を変更できる
    - コマンドの実際の実行時間を測定できる


# 実装

なんやかんやあってできました！
施解錠遅延によって、どの程度違和感が発生するのか、それを体感してもらうのが目的です。

右上の delay input でどのくらいの遅延が発生したか、を模擬します。

<figure class="figure-image figure-image-fotolife" title="限度見本">[f:id:photosynth-inc:20211210130353p:plain]<figcaption>限度見本</figcaption></figure>


## 設計ざっくり

技術的に新しいところは一切なく、Web開発で使いそうな基本的な技術だけで作っています。
Akerun Remote 内部に Akerunの管理コンソール画面 [Akerun Connect](https://connect.akerun.com/) の機能縮小版を作ったようなものです。

- 通信の流れは、ブラウザ -http-> Remote -ble-> Akerun
- Remote 実機内部にAPIサーバ / フロントエンドサーバ を立てて、ブラウザからの解錠指示をAPIサーバで受け取り、Akerunへのコマンド送信を直接行う
- ブラウザで遅延時間の調整ができる
- ngrok を活用して、ブラウザへの公開を簡単にする
- ついでに雑にレスポンシブにして、適当なサイズのスマホと、PCブラウザどちらからも操作可

## 作り方

追加で作るものは、ざっくり2つだけでした。

- APIサーバ
- フロントエンドサーバ

あとは、APIサーバからAkerunのコマンド送信を直接行うツールが必要です。今回はCLIツールを事前にご用意しました。3分クッキングでは定番です。


## APIサーバ
こちらは複雑さをCLIに押し込んだので、とても簡単な構成です。

- 僕がセットアップに慣れている [`flask`](https://flask.palletsprojects.com/en/2.0.x/) を使い、
- レスポンスはフロントエンドで扱いやすいように json 化し、
- 事前に用意したCLIに繋ぎこみました。

```shell
┬─[daiki~@photosyth~:~/g/s/w/r/api]─[15:40:51]─[G:main=]
╰─>$ tree -I '__pycache__|venv'
.
├── README.md
├── app.py
├── appstart.sh
├── docs
│   └── package.json
└── requirements.txt

1 directory, 5 files
```

`app.py` はごく単純で

```python
import os
import json
import subprocess as sp

from flask import abort
from flask import Flask
from flask import jsonify

from flask_cors import CORS

app = Flask(__name__)
CORS(app)

akerun_id = os.environ.get("AKERUN_ID")
cwd = os.environ.get("CLI_DIR")

# remote_cli's bleMode proxys
def bleMode_call(cmd):
    if akerun_id:
        command = ["node", "cli", "ble", cmd, "-a", akerun_id]
    else:
        command = ["node", "cli", "ble", cmd]

    res = sp.run(command, cwd=cwd, stdout=sp.PIPE, stderr=sp.STDOUT)
    print(res.stdout)
    return json.loads(res.stdout.decode('utf-8'))


@app.route('/ble/toggle', methods=['POST'])
def bleMode_toggle():
    res = bleMode_call('toggle')

    if res["level"] == "error":
        return abort(500, res)
    else:
        return jsonify(res)


@app.route('/ble/infopro', methods=['POST'])
def bleMode_infopro():
    res = bleMode_call('infopro')

    if res["level"] == "error":
        return abort(500, res)
    else:
        return jsonify(res)
```

フロントエンドからのリクエストを、Akerun本体にプロキシするような作りになっています。簡単ですね。


## フロントエンドサーバ

- 僕が謎にセットアップに慣れている [`nextjs`](https://nextjs.org/) を使い、
- さらに僕が謎になんとなく好きな [material-ui](https://v4.mui.com/) を使って

作りました。単一ページで、少しコンポーネントに分けていますが、ごく単純な設計です。

```shell
┬─[daiki~@photosyth~:~/g/s/w/j/t/remote-instructor]─[15:11:25]─[G:master=]
╰─>$ tree -I node_modules
.
├── README.md
├── jest.config.js
├── next-env.d.ts
├── next.config.js
├── package.json
├── public
│   └── favicon.ico
├── src
│   ├── components
│   │   ├── large_text_field.tsx
│   │   ├── popup.tsx
│   │   ├── progress_bar.tsx
│   │   ├── pseudo_delay_input.tsx
│   │   ├── record_table.tsx
│   │   └── timer.tsx
│   ├── layouts
│   │   └── main.tsx
│   ├── modules
│   │   ├── logger.ts
│   │   └── utils.ts
│   ├── pages
│   │   ├── _app.tsx
│   │   ├── _document.tsx
│   │   └── index.tsx
│   └── styles
│       └── theme.ts
├── tsconfig.json
└── yarn.lock
```

`package.json` には特に特徴はなく、

```json
{
  "name": "remote-instructor",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "DEBUG=remote-instructor-* next dev",
    "debug": "NODE_OPTIONS='--inspect' next dev",
    "test": "jest",
    "build": "next build",
    "start": "DEBUG=remote-instructor-audit-* next start -p 80"
  },
  "dependencies": {
    "@material-ui/core": "^4.11.4",
    "@material-ui/icons": "^4.11.2",
    "@mui/x-data-grid": "^4.0.1",
    "debug": "^4.3.1",
    "next": "10.0.8",
    "react": "17.0.1",
    "react-dom": "17.0.1"
  },
  "devDependencies": {
    "@types/debug": "^4.1.5",
    "@types/jest": "^26.0.23",
    "@types/node": "^14.14.37",
    "@types/react": "^17.0.3",
    "@types/react-dom": "^17.0.3",
    "dotenv": "^8.2.0",
    "jest": "^26.6.3",
    "ts-jest": "^26.5.5",
    "typescript": "^4.2.3"
  }
}
```

`nextjs` に依存しているため、`pages/index.tsx` にほとんど全て詰まっています（省略するところがほとんどなかった ... ）。

```typescript
// Framework
import React, { useState, useRef } from "react"

// UI
import { makeStyles } from "@material-ui/styles"

import Grid from "@material-ui/core/Grid"

import CircularProgress from "@material-ui/core/CircularProgress"
import Icon from "@material-ui/core/Icon"
import IconButton from "@material-ui/core/IconButton"

import { sleep } from "@/modules/utils"
import { Timer, timeMeasureAround, getTime } from "@/components/timer"
import { PseudoDelayInput } from "@/components/pseudo_delay_input"
import { RecordTable, pushToTable } from "@/components/record_table"

const useStyles = makeStyles({ ... })

const LockState = {
  Unknown: 0, // default
  Indeterminate: 1,
  Open: 2,
  Close: 3,
} as const
type LockState = typeof LockState[keyof typeof LockState]

const LockStateKeys = {
  0: "unknown",
  1: "indeterminate",
  2: "open",
  3: "close",
}

export default function Home(props: any) {
  const classes = useStyles(props)

  const timer = useRef<any>() // reference to the Timer Element
  const table = useRef<any>() // reference to the RecordTable Element

  const [pseudoDelay, setPseudoDelay] = useState<number>(0) // [sec]. input by the user
  const [lockState, setLockStateRaw] = useState<LockState>(LockState.Unknown)

  const lockStateTimeLimit = useRef(null)
  const setLockState = (input: LockState) => {
    setLockStateRaw(input)

    clearTimeout(lockStateTimeLimit.current)
    if (input === LockState.Open || input === LockState.Close) {
      lockStateTimeLimit.current = setTimeout(() => {
        setLockState(LockState.Unknown) // clear lockState after 30 sec
      }, 30 * 1000)
    }
  }

  const processing = useRef(false)
  const iconClickAround = (func: () => Promise<LockState>) => {
    return async () => {
      if (processing.current) return
      processing.current = true
      setLockState(LockState.Indeterminate)

      const nextState = await timeMeasureAround(async () => {
        await sleep(pseudoDelay * 1000) // wait for `pseudoDelay`, to simulate the Akerun Remote delay time
        return await func()
      }, timer)()

      processing.current = false
      setLockState(nextState as LockState)
    }
  }

  const onReplayIconClick = iconClickAround(async () => {
    const url = `/ble/infopro`
    console.log(url)

    return await fetch(url, { method: "POST" })
      .then((resp) => {
        if (!resp.ok) {
          console.error(resp)
          pushToTable({ state: "unknown", cmd: "infopro", time: getTime(timer) }, table)
          throw new Error("server error")
        }
        return resp.json()
      })
      .then((body) => {
        const lock_state = body["message"]["lock_state"]["data"][0]
        const nextState = {
          "0": LockState.Open,
          "1": LockState.Close,
        }[lock_state]

        pushToTable({ state: LockStateKeys[nextState], cmd: "infopro", time: getTime(timer) }, table)
        return nextState
      })
      .catch((err) => {
        console.error(err)
        return LockState.Unknown
      })
  })

  const onLockIconClick = iconClickAround(async () => {
    const url = `/ble/toggle`
    console.log(url)

    return await fetch(url, { method: "POST" })
      .then((resp) => {
        if (!resp.ok) {
          console.error(resp)
          pushToTable({ state: "unknown", cmd: "infopro", time: getTime(timer) }, table)
          throw new Error("server error")
        }
        pushToTable({ state: "open", cmd: "toggle", time: getTime(timer) }, table)
        return LockState.Open
      })
      .catch((err) => {
        console.error(err)
        return LockState.Unknown
      })
  })

  const onUnlockIconClick = iconClickAround(async () => {
    const url = `/ble/toggle`
    console.log(url)

    return await fetch(url, { method: "POST" })
      .then((resp) => {
        if (!resp.ok) {
          console.error(resp)
          pushToTable({ state: "unknown", cmd: "infopro", time: getTime(timer) }, table)
          throw new Error("server error")
        }
        pushToTable({ state: "close", cmd: "toggle", time: getTime(timer) }, table)
        return LockState.Close
      })
      .catch((err) => {
        console.error(err)
        return LockState.Unknown
      })
  })

  return (
    <Grid container direction="row">
      <Grid container direction="row" className={classes.indicatorContainer}>
        <Timer ref={timer} />
        <PseudoDelayInput
          value={pseudoDelay}
          handleChange={(ev: React.ChangeEvent<HTMLInputElement>) => {
            setPseudoDelay(Number(ev.target.value))
          }}
        />
      </Grid>

      <Grid container direction="column" className={classes.iconContainer}>
        {(() => {
          switch (lockState) {
            case LockState.Open:
              return (
                <IconButton onClick={onUnlockIconClick}>
                  <Icon className={classes.bigIcon}>lock_open</Icon>
                </IconButton>
              )
            case LockState.Close:
              return (
                <IconButton onClick={onLockIconClick}>
                  <Icon className={classes.bigIcon}>lock</Icon>
                </IconButton>
              )
            case LockState.Indeterminate:
              return <CircularProgress />
            case LockState.Unknown:
            default:
              return (
                <IconButton onClick={onReplayIconClick}>
                  <Icon className={classes.bigIcon}>replay</Icon>
                </IconButton>
              )
          }
        })()}
      </Grid>

      <Grid container direction="column" className={classes.tableContainer}>
        <RecordTable ref={table} />
      </Grid>
    </Grid>
  )
}
```

少々複雑ですが、保持する状態は少なく、フロントエンド特有のものです。仕方ない。
ただ一応、 [react-hook](https://ja.reactjs.org/docs/hooks-overview.html) を使って多少ｲｲｶﾝｼﾞに書いています。素敵！！


## ngrok

[ngrok](https://dashboard.ngrok.com/user/signup) は、ローカルPC（この場合、Akerun Remote内部）で動作させているTCPアプリケーションを簡単に外部公開できるサービスです。

[プログラマ3大美徳](https://freelance-start.com/articles/575) を黙らせてくれる大変便利なツール。

- ただのデモアプリだろうと、関係者はまとめて触るのが当然（傲慢）
- 本番相当のホスト環境を用意するなんて面倒なことをするはずが（怠惰）
- そもそもそんなことに時間をかけるのはいやだ（短気）

このデモを作った際に個人的に課金しました。

<figure class="figure-image figure-image-fotolife" title="左: nextjsの動作環境 / 右: ngrokでサービス提供">[f:id:photosynth-inc:20211210153523p:plain]<figcaption>左: nextjsの動作環境 / 右: ngrokでサービス提供</figcaption></figure>


え？ [vercel](https://vercel.com/) を使えばいいじゃないかって？ｿｳﾃﾞｽﾈ .....


## CORS対策

さて、実はこのままでは動かないです…

nextjs と別の APIサーバ を共有するのに、クロスオリジン制約を突破する必要があります。

[nextjs の `rewrite`](https://nextjs.org/docs/api-reference/next.config.js/rewrites) を使うか、[nextjs API Route](https://nextjs.org/docs/api-routes/introduction)で一旦受け取ってプロキシすると良いです。そのうち追記するかも。。


# 限度見本が与えた影響

目の前に動くものがあることで、障害発生時の振る舞いを（開発・サポート・営業の誰もが）想像しやすくなり、コミュニケーションが簡単になりました。
おかげで、部署・関係者間の齟齬なく、やるべきことだけに取り組めるようになりました。


副次的な効果として、アドベントカレンダーのネタになったことに感謝しつつ、今日は終わり。


# 参考リンク

- [アジャイルソフトウェア開発宣言](https://agilemanifesto.org/iso/ja/manifesto.html)
- [ngrokが便利すぎる - Qiita](https://qiita.com/mininobu/items/b45dbc70faedf30f484e)
- [Next.jsを使うべき5つの理由 + 実装Tips - Qiita](https://qiita.com/Yuki_Oshima/items/5c0dfd8f7af8fb76af8f)
- [フック早わかり – React](https://ja.reactjs.org/docs/hooks-overview.html)
- [Welcome to Flask — Flask Documentation (2.0.x)](https://flask.palletsprojects.com/en/2.0.x/)
- [About this documentation | Node.js v17.2.0 Documentation](https://nodejs.org/docs/latest/api/documentation.html)


---

株式会社フォトシンスでは、一緒にプロダクトを成長させる様々なレイヤのエンジニアを募集しています。
[https://hrmos.co/pages/photosynth:embed:cite]

Akerun Proの購入はこちらから
[https://akerun.com/:embed:cite]
