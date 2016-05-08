# [公式チュートリアルではじめる]CircleCI＋ECS+ECRを使ったCD(継続的デプロイ)

コンニチハ、千葉です。

[公式チュートリアル](https://circleci.com/docs/continuous-deployment-with-aws-ec2-container-service/)にそって、CircleCIとECS+ECRを使ったCD(継続的デプロイ)をやってみました。

- Dockerを復習したい方は[公式チュートリアルで始めるDocker](http://dev.classmethod.jp/tool/docker/start-docker-tutorials/)
- ECSを復習したい方は[AWS Blackbelt 2015シリーズ Amazon EC2 Container Service (Amazon ECS)](http://www.slideshare.net/AmazonWebServicesJapan/aws-blackbelt-2015-ecs)
- ECRを復習したい方は[Amazon EC2 Container Registry(ECR)でECS/Elastic BeanstalkのDockerイメージをホストする(1/12アップデート)](http://dev.classmethod.jp/cloud/ecr-on-ecs-eb/)

今回は公式ドキュメントにそり、ECS+ECR+Dockerコンテナ環境おける、Go言語サンプルWebアプリケーションのビルド・テスト・デプロイを行ってみます。

## AWS環境の構築

まずは、AWS環境のセットアップを行います。Getting Started with Amazon EC2 Container Service (ECS)より、サンプルの環境を作ります。

[こちら](https://console.aws.amazon.com/ecs/home?region=us-east-1#/firstRun)から、ECS+ECRの環境をセットアップします。

手順については、[こちら](http://dev.classmethod.jp/cloud/ecr-on-ecs-eb/)を参考に。

[bash]
# Building on top of Ubuntu 14.04. The best distro around.
FROM ubuntu:14.04

COPY ./go-ecs-ecr /opt/
EXPOSE 8080

ENTRYPOINT ["/opt/go-ecs-ecr"]
[/bash]

##
