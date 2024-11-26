# aws-google-auth を Go でリライトした話

この記事は [https://qiita.com/advent-calendar/2024/akerun:title] の 1 日目の記事です。

どうも、 [https://qiita.com/daikw:title] です。ご無沙汰しています。
今年の4月から、フォトシンスの CTO になりました。機械とお話しする時間が減り、人間とお話しする時間が増えました。

## 概要

今年、[`aws-google-auth`](https://github.com/cevoaustralia/aws-google-auth) を Go でリライトしました。 Google SSO でログインしながら、 AWS CLI 用の認証情報を取得するツールです。

[Photosynth-inc/aws-google-login: Provides AWS STS credentials based on Google Apps SAML SSO auth with interactive GUI support](https://github.com/Photosynth-inc/aws-google-login)

```sh
$ brew install Photosynth-inc/tap/aws-google-login
$ aws-google-login
NAME:
   aws-google-login - Acquire temporary AWS credentials via Google SSO (SAML v2)

USAGE:
   aws-google-login [global options] [command [command options]] [arguments...]

COMMANDS:
   config   Show current configuration
   cache    Manage application's cache
   help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --profile value, -p value           AWS Profile to use (default: "akerun")
   --duration-seconds value, -d value  Session Duration (in seconds) (default: 3600)
   --sp-id value, -s value             Service Provider ID (default value is in /Users/daikiwatanabe/.aws/config)
   --idp-id value, -i value            Identity Provider ID (default value is in /Users/daikiwatanabe/.aws/config)
   --role-arn value, -r value          AWS Role Arn for assuming to, ex: arn:aws:iam::123456789012:role/role-name
   --select-role-interactivelly, -l    choose AWS Role interactively. If set, 'role-arn' will be ignored (default: false)
   --browser-timeout value, -t value   browser timeout duration in seconds (default: 60)
   --log value                         change Log level, choose from: [trace | debug | info | warn | error | fatal | panic]
   --help, -h                          show help (default: false)
```

## 契機

弊社では AWS を主に利用しています。
Google Workspace を IdP とする SSO でユーザ管理をしており、開発者が AWS CLI を利用する際には Python 製の [`cevoaustralia/aws-google-auth`](https://github.com/cevoaustralia/aws-google-auth) を利用していました。

ところが、　2024年5月ごろから `aws-google-auth` での認証が頻繁に失敗するようになりました。
Google ログインにまつわる仕様変更によるものだと思われますが、 `aws-google-auth` は数年単位でメンテナンスされておらず、修正は望めません。

もし AWS CLI が使えない状態で障害があると、実施できない調査や復旧作業が出てくるかもしれません。

## 代替案

障害を機に、初めて真面目に `aws-google-auth` の実装を読みました。仕組みは比較的簡単で、

- Google ログイン時に得られる HTML から SAML レスポンスをキャプチャし、
- [`AssumeRoleWithSAML`](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithSAML.html) API を利用して STS トークンを取得
- 得られた STS トークンを `.aws/credentials` に書き込み

の流れで認証情報 (STS) を取得します。`.aws/credentials` に記載の認証情報は「プロファイル」として他のツールで利用できるようになります。

したがって、

- ブラウザ上、デベロッパーコンソールで SAML レスポンスをキャプチャし、
- SAML レスポンスを使って `aws sts assume-role-with-saml` を実行
- 得られた STS トークンを `.aws/credentials` に書き込み

の手順を踏むことで、 `aws-google-auth` を使わなくても STS を取得できます。

この処理をシェルスクリプトで書くと、以下のようになります。

```sh
# コピーした SAML レスポンスをファイルに保存
pbpaste > resp.saml

# SAML レスポンスを使って STS トークンを取得、変換してファイルに書き込み
export account_id=your_account_id_here
export role=your_role_here
aws sts assume-role-with-saml --role-arn arn:aws:iam::$account_id:role/$role --principal-arn arn:aws:iam::$account_id:saml-provider/GSuite --duration-seconds 3600 --saml-assertion file://resp.saml | jq -r '.Credentials' | awk -F':' '
BEGIN {RS="," ; print "[akerun]"}
/:/{ gsub(/"/, "", $2) }
/AccessKeyId/{print "aws_access_key_id =" $2}
/SecretAccessKey/{print "aws_secret_access_key =" $2}
/SessionToken/{ print "aws_session_token =" $2 }
/Expiration/{ printf "aws_session_expiration =%s:%s:%s",$2,$3,$4}
' | sed 's/"//g' | sed '/}/d'
```

## リライト

怠惰な僕には、上述の作業が週に数度のゴミ出しよりも面倒なので、別の手段を考えていました。

`aws-google-auth` が壊れたのは、 どうやら Google ログイン時に得られる HTML の構造が変わったことが原因のようでした。ログインページを含む全ての HTML を BeautifulSoup でパースしていて、 HTML の構造の変化に弱いです。
二要素認証など、ページをまたがっての変更の修正はめんどくさいですし、複数のパターンの HTML 構造も懸念されます。

[`cucxabong/aws-google-login`](https://github.com/cucxabong/aws-google-login) はこの点を解決していて、Playwright でブラウザ操作し、ユーザに認証情報を自分で入力させ、結果として SAML レスポンスを含む出力をアプリケーションが取得します。これは素晴らしい。機械に難しいことは人間にやらせた方が良いでしょう。

この実装をベースに、以下のような変更を入れていきました。

- Go とパッケージのバージョンを更新
- Homebrew でインストールできるように、 GoReleaser を利用
- いくつかの機能を追加
  - `~/.aws/config` を利用する機能
  - `~/.aws/credentials` を自動修正する機能
  - 自動でアカウントエイリアスを取得し表示する機能
  - 対話的にログイン先アカウントを選択する機能: `aws-google-login --select-role-interactivelly`
  - 現在の設定を表示する機能: `aws-google-login config`
  - 内部ブラウザのセッションキャッシュの削除機能: `aws-google-login cache --clear`


元々 `aws-google-auth` を使用する際は、　`~/.aws/config` に以下のような設定を入れますが、

```ini
[profile akerun]
output = json
region = ap-northeast-1
google_config.ask_role = True
google_config.keyring = False
google_config.duration = 3600
google_config.google_idp_id = 'your_idp_id'
google_config.google_sp_id = 'your_sp_id'
google_config.u2f_disabled = False
google_config.google_username = daiki.watanabe@photosynth.co.jp
google_config.bg_response = None
google_config.role_arn = arn:aws:iam::$account_id:role/$role_id
```

`aws-google-login` もこの設定項目をそのまま利用 できます。ただ、 ~~実装がちょっと面倒だったので~~ いくつかのフィールドは使っていません。


レポジトリをハードフォークした上で、以下の PR で差分を実装しました。気がつくと元レポジトリの面影はほとんどなくなりました。

- [Upgrade Go / packages by daikw · Pull Request #1 · Photosynth-inc/aws-google-login](https://github.com/Photosynth-inc/aws-google-login/pull/1)
- [Feat/use-aws-config and others by daikw · Pull Request #2 · Photosynth-inc/aws-google-login](https://github.com/Photosynth-inc/aws-google-login/pull/2)
- [Feat/installable-via-homebrew by daikw · Pull Request #4 · Photosynth-inc/aws-google-login](https://github.com/Photosynth-inc/aws-google-login/pull/4)

初期の実装では Google Workspace 上の設定やロールの設定によって挙動差があり、細かいデバッグは、社内の SRE エンジニアをはじめとした有志に手伝っていただきました。

## まとめ

この記事を書くにあたって `aws-google-auth` を読んでいたら、気になる PR があったのでコメントをつけました。
https://github.com/cevoaustralia/aws-google-auth/issues/284#issuecomment-2496911043

このツールが誰かの役に立ったらいいなと思います。

ちなみに、個人的には `--select-role-interactivelly` が結構便利で、気に入っている機能です。

```sh
┬─[daiki@mac42:~]─[15:50:42]
╰─>$ aws-google-login -l
Please login to your Google account in the browser...
Request received, processing...
Use the arrow keys to navigate: ↓ ↑ → ←
? Select AWS Role:
    akerun-vvv (123456789012)
    akerun-www (234567890123)
  ▸ akerun-xxx (345678901234)
    akerun-yyy (456789012345)
    akerun-zzz (567890123456)
```

```sh
┬─[daiki@mac42:~]─[15:50:42]
╰─>$ aws-google-login -l
Please login to your Google account in the browser...
Request received, processing...
✔ akerun-xxx (345678901234)
Temporary AWS credentials have been saved to /Users/daikiwatanabe/.aws/credentials
```

---

株式会社フォトシンスでは、一緒にプロダクトを成長させる様々なレイヤのエンジニアを募集しています。
[https://hrmos.co/pages/photosynth:embed:cite]

Akerun Pro の購入はこちらから
[https://akerun.com/:embed:cite]
