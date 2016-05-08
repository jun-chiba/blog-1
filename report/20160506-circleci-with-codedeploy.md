# [公式チュートリアルではじめる]CircleCI＋CodeDeployを使ったCD(継続的デプロイ)

コンニチハ、千葉です。

[公式チュートリアル](https://circleci.com/docs/continuous-deployment-with-aws-codedeploy/)にそって、CircleCIとCodeDeployを使ったCD(継続的デプロイ)をやってみました。

CodeDeploy単体だとS3のファイルをEC2インスタンスへデプロイするのみですが、CircleCIを利用することで自動でアプリケーションのリビジョンを作成、S3へアップロードしデプロイするところまでを自動化することが可能です。つまり、GitHubへのコミットをトリガーに、ビルド・テスト・デプロイまでを自動化することができます。。CircleCI、CodeDeployが初めての人でもこれをやってみると雰囲気がつかめると思います。

CodeDeployを復習したい方は[CodeDeploy入門](http://dev.classmethod.jp/cloud/aws/cm-advent-calendar-2015-aws-re-entering-codedeploy/)が参考になると思います。

## AWS環境の構築

継続的インテグレーションの最初のステップとして、CodeDeployの設定をします。具体的には、

- EC2インスタンスの作成
- CodeDeoloyにてデプロイメントグループ、ロールを設定
- EC2インスタンスへタグの設定とCodeDeploy agentをインストール

今回はチュートリアルなので、[[新サービス] AWS CodeDeployを触ってみた #reinvent](http://dev.classmethod.jp/cloud/codedeploy-ataglance/)を参考にサンプルアプリをデプロイする環境を構築しました。EC2が3台起動し、サンプルアプリがデプロイされている状態になっていればokです。

★1
★2

### デプロイ用S3バケットの作成

アプリを格納するS3バケットを作成します。今回は```chiba-app```というバケットを作成しました。

### CircleCI用のIAMユーザの作成

CircleCI用のIAMユーザの作成と、アクセスキーを取得します。今回は、「circleci-user」を作成しアクセスキーを取得しました。また、S3とCodeDeployに対する最小権限を与えます。

#### S3用ポリシー

※backet名は適宜変更

[bash]
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::chiba-app/*"
      ]
    }
  ]
}
[/bash]

#### CodeDeploy用ポリシー

※arnは適宜変更

[bash]
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "codedeploy:RegisterApplicationRevision",
        "codedeploy:GetApplicationRevision"
      ],
      "Resource": [
        "arn:aws:codedeploy:ap-northeast-1:XXXXXXXXXXXX:application:DemoApplication"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "codedeploy:CreateDeployment",
        "codedeploy:GetDeployment"
      ],
      "Resource": [
        "arn:aws:codedeploy:ap-northeast-1:XXXXXXXXXXXX:deploymentgroup:DemoApplication/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "codedeploy:GetDeploymentConfig"
      ],
      "Resource": [
        "arn:aws:codedeploy:ap-northeast-1:XXXXXXXXXXXX:deploymentconfig:CodeDeployDefault.OneAtATime",
        "arn:aws:codedeploy:ap-northeast-1:XXXXXXXXXXXX:deploymentconfig:CodeDeployDefault.HalfAtATime",
        "arn:aws:codedeploy:ap-northeast-1:XXXXXXXXXXXX:deploymentconfig:CodeDeployDefault.AllAtOnce"
      ]
    }
  ]
}
[/bash]


## CircleCIの設定

アプリケーションのリビジョン作成を自動化、またビルド成功時にS3へのアップロード+デプロイの自動化を行います。

### CircleCIへIAMユーザを登録

まずは、今回用のCircleCIプロジェクトを追加します。事前にGitHub上に「circleci-codedeploy-test」というレポジトリを作成しておきます。

★3

Project Settings > AWS keysより、先ほど取得したIAMユーザのアクセスキーを登録します。

★4
★5

###  デプロイパラメータの設定

CircleCI用のパラメータファイルを```circle.yml```作成します。(デモなので今回はテストは強制的に```exit 0```にしてスキップしています。)

[bash]
test:
  override:
    - exit 0
deployment:
  staging:
    branch: master
    codedeploy:
      DemoApplication:
        application_root: /
        region: ap-northeast-1
        revision_location:
          revision_type: S3
          s3_location:
            bucket: chiba-app
            key_pattern: DemoApplication-{BRANCH}-{SHORT_COMMIT}
        deployment_group: DemoFleet
        deployment_config: CodeDeployDefault.AllAtOnce
  [/bash]

AWS環境構築時にデプロイした[デモアプリ](https://s3-ap-northeast-1.amazonaws.com/aws-codedeploy-ap-northeast-1/samples/latest/SampleApp_Linux.zip)もダウンロードして配置しておきます。コミットするファイルは以下のようになります。

[bash]
$ tree
.
├── LICENSE.txt
├── appspec.yml
├── circle.yml
├── index.html
└── scripts
    ├── install_dependencies
    ├── start_server
    └── stop_server
[/bash]


## 動作確認

では、コミットしてみます。デプロイ状況が分かるようにデプロイする```index.html```を変更します。

[bash]
echo "hello CircleCI" > index.html
[/bash]

コミットした後に動作確認をしました。

CircleCI上はSuccessです！
★6

S3のアプリ格納用バケットを確認したところファイルがアップロードされていました。
★7

CodeDeployはどうでしょう。

★8

こちらも、コミットしたタイミングでデプロイ完了しているようです。

最後にブラウザでアクセスしてみます。

★9

問題なくデプロイされています！

## 最後に

設定項目も少なく、簡単にCircleCI+CodeDeployによるデプロイ連携ができました。これを機会に触ってみてはいかがでしょうか。とても便利に感じました。
