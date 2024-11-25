# AWS Systems Managerを使ってraspiマシンを運用する時の注意点

この記事は [https://qiita.com/advent-calendar/2021/akerun:title] の 19日目の記事です。

今年の初めに `AWS Solution Architect Associate` を取得した [https://qiita.com/daikw:title] です 🎉

今日は `AWS Systems Manager` (`SSM`) に関する話をします。

[:contents]


# はじめに
## SSM で解決したいこと

クラウド側で全てprovisionしてくれる仮想マシンとは別に、オンプレマシン **も** 扱いたい、というのは、実は結構よくある要求ではないでしょうか。

弊社のFW開発チームも、raspiをあちこちで使っていて、正直管理が辛いところがあります。

クラウド・オンプレどちらもできるだけ同じように扱うことができると、管理コストが減ってとてもハッピーになれそうです。

ところで、 [AWS Systems Manager（運用時の洞察を改善し、実行）| AWS](https://aws.amazon.com/jp/systems-manager/) によれば、

> AWS Systems Manager は、ハイブリッドクラウド環境のための安全なエンドツーエンドの管理ソリューションです。

とあります。もしや、これを使ったらハッピーになれるのでは？？？

<figure class="figure-image figure-image-fotolife">
[https://d1.awsstatic.com/AWS%20Systems%20Manager/product-page-diagram-AWS-Systems-Manager_how-it-works.e9ba8cbeff1249c8cc24db4737d03648a1a073f6.png:image=https://d1.awsstatic.com/AWS%20Systems%20Manager/product-page-diagram-AWS-Systems-Manager_how-it-works.e9ba8cbeff1249c8cc24db4737d03648a1a073f6.png]
<figcaption>[https://aws.amazon.com/jp/systems-manager/] より</figcaption>
</figure>


## raspi x SSM
クラスメソッドさんの記事で、 raspi x SSM カテゴリのものがかなり豊富にありました。

- [Amazon EC2 Systems ManagerがRaspbian OSに対応したのでRaspberry Piにインストールしてみた | DevelopersIO](https://dev.classmethod.jp/articles/ssm-with-raspberrypi/)
- [AWS Systems ManagerをRaspberry Piで使用してみた | DevelopersIO](https://dev.classmethod.jp/articles/aws_systems_manager_raspberry_pi/)
- [[AWS Systems Manager] アクティベーションの終わったRaspberryPiのイメージをコピーして、複数起動した時の挙動を確認してみました | DevelopersIO](https://dev.classmethod.jp/articles/aws-systems-manager-activation-image-conflict/)
- [【小ネタ】1台のマネージドインスタンスを踏み台にして、多数のRasPiに選択メニューからsshするシェルを作ってみました | DevelopersIO](https://dev.classmethod.jp/articles/amazon-systems-manager-proxy-ssh/)
- [【小ネタ】 AWS Systems Manager のマネージドインスタンス（RaspberryPi）を踏み台にして、Windows10にリモートデスクトップで接続してみました。 | DevelopersIO](https://dev.classmethod.jp/articles/windows10-rdp-with-ssm/)
- [【小ネタ】 AWS Systems Managerに登録した多数のRaspberryPiに識別しやすいように名前をつける | DevelopersIO](https://dev.classmethod.jp/articles/aws-systems-manager-append-name-tag/)

本記事では、これらを参考に運用を試みた際に感じた、導入にあたって注意すべき点をまとめておきます。
また、記事の後半に用語リストをまとめてあります。参考までに。


# 運用して気がついたこと

## セッションログは、セッションの開始方法によっては残せない

AWS Console の Session Manager を経由した SSHセッションログを、S3 や Cloudwatch logs に流すことができます。

[AWS Systems Manager Session Managerのシェル操作をログ出力する | DevelopersIO](https://dev.classmethod.jp/articles/log-ssm-session-manager-for-shell-access-activity/)

しかし、 `aws ssm start-session` コマンドによるログインセッションのログは同じログストリームに流すことができません。

<figure class="figure-image figure-image-fotolife" title="[No s3 log when ssh ubuntu@i-xxxxxxxxxx · Issue #194 · aws/amazon-ssm-agent](https://github.com/aws/amazon-ssm-agent/issues/194)">[f:id:photosynth-inc:20211218133733p:plain]<figcaption>[No s3 log when ssh ubuntu@i-xxxxxxxxxx · Issue #194 · aws/amazon-ssm-agent](https://github.com/aws/amazon-ssm-agent/issues/194) より</figcaption></figure>


不便といえば不便ですが、当たり前といえば当たり前（接続元と接続先で鍵交換するので、session-manager は復号できない、ﾀﾌﾞﾝ）で、ややもどかしくもあります。


## Quick Setup を組み合わせる場合、セッションログを取得するために小細工が必要
Quick Setup は基本 EC2 インスタンスに対して行うものですが、ターゲットに Managed Instance を選択することもできます。ハイブリッド環境もまとめて扱えるのが Systems Manager のｲｲﾄｺﾛなので、まとめて選択してしまうこともあるでしょう。

デフォルトでは、Hybrid Activationの際に [`AmazonEC2RunCommandRoleForManagedInstances`](https://console.aws.amazon.com/iam/home#/roles/AmazonEC2RunCommandRoleForManagedInstances?section=trust) が Managed Instance にアタッチされます。これは Identity Provider が `ssm` になっています。
Quick Setupの後に、それらしい Role ([`AmazonSSMRoleForInstancesQuickSetup`](https://console.aws.amazon.com/iam/home#/roles/AmazonSSMRoleForInstancesQuickSetup?section=permissions))が作成されるのですが、これは Managed Instance に自動でアタッチされません。さらにこちらの Identity Provider は `ec2` になっています。

以下のポリシーを含む [`ssm-managed-instance`](https://console.aws.amazon.com/iam/home#/roles/akerun-managed-instance) role を作成して、

- `CloudWatchAgentServerPolicy`
- `AmazonSSMManagedInstanceCore`
- `AmazonSSMPatchAssociation`

`ssm-managed-instance` を Fleet Manager ([AWS Systems Manager - Managed Instances](https://ap-northeast-1.console.aws.amazon.com/systems-manager/managed-instances?region=ap-northeast-1)) から 対象の Managed Instance にアタッチし、さらにこの際、RoleのIdentity Provider を `ssm` にすることで、cloudwatch logs に ssm session のログが流せるようになります。

この辺りは IAM Role, Policy の構成をよく調べないとうまく使うことができなそうです。来れ、aws猛者たちよ。


## ssm-user の権限が広い
Session Manager を利用する場合、特に aws-console 上からのログインを許可する場合は、デフォルトで `ssm-user` が利用されます。

この `ssm-user` は `sudoers` なので、Session Manager を下手に許可してしまうと [最小権限の原則 - Wikipedia](https://ja.wikipedia.org/wiki/%E6%9C%80%E5%B0%8F%E6%A8%A9%E9%99%90%E3%81%AE%E5%8E%9F%E5%89%87) に簡単に反します。

「Session Manager を利用するのは、特権管理者である」と想定されているのかと思います。

チームで Session Manager を共有して使う場合は、コンプラという言葉が飛び交う現代IT産業界の皆様が口から泡を吹いて倒れることのないように、 `ssm-user` を sudoers から除去しておくのが無難です。

ついでに pi ユーザの `nopasswd` も除去しておきましょう。

```sh
sudo su
cd /etc/sudoers.d
echo "#User rules for ssm-user" > ssm-agent-users
rm 010_pi-nopasswd
```

参考:

- [[小ネタ]新機能Session Managerで使うssm-userの権限が気になった話 | DevelopersIO](https://dev.classmethod.jp/articles/why-does-ssm-user-use-sudo/)
- [ステップ 7: (オプション) ssm-user アカウントの管理アクセス許可を有効または無効にする - AWS Systems Manager](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager-getting-started-ssm-user-permissions.html)


## インスタンスのクローンを作るときは、再度アクティベーションする

Raspberry Pi で、既にあるマシンと同等のマシンを作りたい（複製する）場合は、 MicroSDカードからイメージを `dd` コマンド等で吸い出し、別のカードに書き込んで用意することがあります。

アクティベート済みのマシンを複製する場合、 `ssm-agent` による[ハードウェアフィンガープリント](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/ssm-agent-technical-details.html#fingerprint-validation) に引っかかるため、そのままでは使えないであろうことが予想できました。

自分でも挙動を調べたんですが、クラスメソッドさんの記事が僕よりよくまとまっていました。

[[AWS Systems Manager] アクティベーションの終わったRaspberryPiのイメージをコピーして、複数起動した時の挙動を確認してみました | DevelopersIO](https://dev.classmethod.jp/articles/aws-systems-manager-activation-image-conflict/) より引用すると、

> アクティベーションが済んだイメージを複製する場合、
> - 1台だけ起動する場合、どちらでも接続可能
> - 複数台起動した場合、後から起動したものが接続される
> - 複数台起動した場合、リブートすると、どちらに接続されるかは不定
> - 複数台起動した場合、セッションマネージャから再アクティベートすると別のインスタンスとして管理される


## session-manager を利用できるIAM Roleの設定

セッション開始には、`"ssm:StartSession"` アクションの許可が必要になります。

例えば、 [Session Manager の追加サンプル IAM ポリシー - AWS Systems Manager](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/getting-started-restrict-access-examples.html) から抜粋すると、

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:StartSession"
            ],
            "Resource": [
                "arn:aws:ec2:us-east-2:123456789012:instance/i-1234567890EXAMPLE",
                "arn:aws:ec2:us-east-2:123456789012:instance/i-abcdefghijEXAMPLE",
                "arn:aws:ec2:us-east-2:123456789012:instance/i-0e9d8c7b6aEXAMPLE"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "ssm:TerminateSession",
                "ssm:ResumeSession"
            ],
            "Resource": [
                "arn:aws:ssm:*:*:session/${aws:username}-*"
            ]
        }
    ]
}
```

## .ssh/config を適切に設定する
いろいろと試した結果、以下2点わかっています。

1. `AWS-StartSSHSession` document  は、 `ProxyCommand` を使わないと動作しない
2. `aws ssm`コマンドを利用する場合、2段以上の `ProxyCommand` チェーンは効かない


1 は、ターミナルにコマンド直打ちする時以外は特に困らないですが、 `Protocol mismatch` となります。

```
┬─[daiki~@photosyth~:~]─[15:23:10]
╰─>$ aws ssm start-session --target mi-xxxxxxxxx --document-name AWS-StartSSHSession --parameters 'portNumber=22222'

Starting session with SessionId: daiki.watanabe@photosynth.co.jp-0b95bd5ed82475bae
SSH-2.0-OpenSSH_7.9p1 Raspbian-10+deb10u2+rpt1

ls
Protocol mismatch.


Exiting session with sessionId: daiki.watanabe@photosynth.co.jp-0b95bd5ed82475bae.
```

2 は少しだけ困っていて、以下のような記法でのログインに失敗します。

```.ssh/useless-config
# SSH over Session Manager
host i-* mi-*
    ProxyCommand sh -c "aws ssm start-session --profile akerun --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"

Host target
    User pi
    ProxyCommand ssh mi-xxxxxxxxx -l pi -p 22222
```

```shell
┬─[daiki~@photosyth~:~]─[10:52:12]
╰─>$ ssh target
Pseudo-terminal will not be allocated because stdin is not a terminal.
daikiwaranabe@mi-01630ac873d5fe084: Permission denied (publickey).
kex_exchange_identification: Connection closed by remote host
```

config の設定をもっと頑張ればいけるかもしれません。ホストが増えないうちは特に困らないので、緩めのﾏｻｶﾘが来ることを期待しつつ。


# その他参考記事

- [AWS Systems Manager セッションマネージャーでSSH・SCPできるようになりました | DevelopersIO](https://dev.classmethod.jp/articles/session-manager-launches-tunneling-support-for-ssh-and-scp/)
- [AWS System Managerセッションマネージャーでポートフォワードする | DevelopersIO](https://dev.classmethod.jp/articles/port-forwarding-using-aws-system-manager-sessions-manager/)
- [send-ssh-public-key と ssm start-session の合わせ技 | 1Q77](https://blog.1q77.com/2020/11/send-ssh-public-key-and-ssm-start-session/)
- [SSM Session Manager 経由での SSH | 1Q77](https://blog.1q77.com/2020/04/ssh-connection-through-session-manager/)


# 用語リスト
SSM固有の用語が多く、概念になれるのに少し時間がかかるので、先んじてまとめておきます。

## Hybrid Activation
用意したマシンを Systems Manager に登録する作業のことを、ハイブリッドアクティベーションと呼びます。

[ステップ 4: ハイブリッド環境のマネージドインスタンスのアクティベーションを作成する - AWS Systems Manager](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/sysman-managed-instance-activation.html) より、

> ハイブリッド環境でサーバーと仮想マシン (VM) をマネージドインスタンスとして設定するには、マネージドインスタンスのアクティベーションを作成する必要があります。アクティベーションが完了するとすぐに、アクティベーションコードとアクティベーション ID が送信されます。ハイブリッド環境でサーバーと VM に AWS Systems Manager SSM Agent をインストールするときに、このコードと ID の組み合わせを指定します。このコードと ID は、マネージドインスタンスから Systems Manager サービスへの安全なアクセスを提供します。

つまり、以下を実行するだけ。

- アクティベーションコードとアクティベーションID を使って、
- ssm-agent をインストールしたマシンを登録する


```sh
sudo service amazon-ssm-agent stop
sudo amazon-ssm-agent -register -code $activation_code -id $activation_id -region $region
sudo service amazon-ssm-agent start
```

## Managed Instance
Systems Manager で管理下にある、自分で用意したマシンのことを言います。オンプレミスサーバであることもあるし、何か別の形態でプロビジョンされたサーバであることもあり得ます。


## SSM Agent
[エージェント（agent）とは - IT用語辞典 e-Words](https://e-words.jp/w/%E3%82%A8%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%B3%E3%83%88.html) によれば、

> エージェントとは、代理人、代理店、仲介人、取次業者、などの意味を持つ英単語。ITの分野では、利用者や他のシステムの代理として働いたり、複数の要素の間で仲介役として機能するソフトウェアやシステムなどを指すことが多い。

SSM Agent は ManagedInstance に常駐し、Systems Manager にインスタンスの状態を渡したり、Systems Managerからのリクエストを処理する機能を持ちます。 [SSM Agent の使用 - AWS Systems Manager](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/ssm-agent.html) によれば、

> エージェントは、 AWS クラウド で Systems Manager サービスからのリクエストを処理し、リクエストに指定されたとおりに実行します。SSM Agent は、Amazon Message Delivery Service (サービスプレフィックス: ec2messages) を使用して、Systems Manager サービスにステータスと実行情報を返します。


## Session Manager
`Systems Manager` の一つの機能で、対象ホストとのSSHログインセッションを **直接** 貼ることができるようになります。

[AWS Systems Manager Session Manager - AWS Systems Manager](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager.html) より ((AWSのドキュメントは、5回くらい読まないとちゃんと真意が掴めないのは僕だけでしょうか... ))、

> Session Manager はフルマネージド型 AWS Systems Manager 機能で、インタラクティブなワンクリックブラウザベースのシェルや AWS Command Line Interface (AWS CLI)を介して Amazon Elastic Compute Cloud (Amazon EC2) インスタンス、オンプレミスインスタンス、および仮想マシン (VM) を管理できます。Session Manager を使用すると、インバウンドポートを開いたり、踏み台ホストを維持したり、SSH キーを管理したりすることなく、監査可能なインスタンスを安全に管理できます。


---

株式会社フォトシンスでは、一緒にプロダクトを成長させる様々なレイヤのエンジニアを募集しています。
[https://hrmos.co/pages/photosynth:embed:cite]

Akerun Proの購入はこちらから
[https://akerun.com/:embed:cite]
