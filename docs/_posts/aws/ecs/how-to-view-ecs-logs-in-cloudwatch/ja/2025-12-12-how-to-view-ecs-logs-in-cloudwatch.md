---
layout: my-post
title: "CloudWatchでECSのログを確認する"
date: 2025-12-12 00:00:00 +0000
categories: aws ecs
page_name: how-to-view-ecs-logs-in-cloudwatch
lang: ja
image: /assets/images/aws/ecs/how-to-view-ecs-logs-in-cloudwatch/image1.png
---

CloudWatchでECSのログを確認する方法をまとめました。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 参考
- [Amazon ECS タスク定義の例: CloudWatch にログをルーティングする](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/specify-log-config.html)
- [インターフェイス VPC エンドポイントでの CloudWatch Logs の使用](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/cloudwatch-logs-and-interface-VPC.html)

## 前提
- プロジェクトがECSにデプロイされている  
LaravelプロジェクトをECSにデプロイする方法については[こちら](/aws/ecs/deploy-laravel-to-ecs-using-fargate)をご覧ください。

## 設定手順
1. [logConfigurationを追加する](#1-logconfigurationを追加する)
2. [VPCエンドポイントを作成する](#2-vpcエンドポイントを作成する)
3. [ECSタスク実行用IAMロールを編集する](#3-ecsタスク実行用iamロールを編集する)

## 1. logConfigurationを追加する
`task-definition.json` に `logConfiguration` を追加します。  

`task-definition.json`:

```json
{
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "family": "my-fargate",
    "containerDefinitions": [
        {
            "name": "my-container",
            ...
            // logConfigurationを追加
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-create-group": "true",
                    "awslogs-group": "my-log-group",
                    "awslogs-region": "ap-northeast-1",
                    "awslogs-stream-prefix": "my-container"
                }
            },
            ...
```

各オプションの説明は以下の通りです。

| オプション | 説明 |
|--------|-------------|
| awslogs-create-group | `true` にするとロググループが自動的に作成されます。 |
| awslogs-group | ログを送信するロググループ名を指定します。 |
| awslogs-region | ロググループがあるリージョンを指定します。 |
| awslogs-stream-prefix | ログストリームにつけるプレフィックスを指定します。 |

## 2. VPCエンドポイントを作成する
ECSのコンテナがプライベートサブネットにある場合、CloudWatch Logs用のVPCエンドポイントを作成する必要があります。

AWSマネジメントコンソールの[VPC]((https://console.aws.amazon.com/vpc/))に移動します。  
「Endpoints」をクリックします。

![AWSマネジメントコンソールのVPC](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "AWSマネジメントコンソールのVPC")

「Create endpoint」をクリックします。

![エンドポイント一覧画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "エンドポイント一覧画面")

名前を入力し、「Type」には「AWS services」を選択します。

![エンドポイント作成画面1](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "エンドポイント作成画面1")

`com.amazonaws.region.logs` を選択します。

![エンドポイント作成画面2](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "エンドポイント作成画面2")

VPC、プライベートサブネット、プライベート用のセキュリティグループを選択します。

「Create endpoint」をクリックします。

## 3. ECSタスク実行用IAMロールを編集する
ECSタスク実行用IAMロールにロググループの作成を許可するポリシーを追加します。  
このIAMロールの詳細については[こちらの記事](/aws/ecs/deploy-laravel-to-ecs-using-fargate#5-ecsタスク実行用iamロールの作成)を参照してください。

次のようなポリシーを追加します。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:<ロググループのリージョン>:<アカウントID>:log-group:*"
        }
    ]
}
```

もしECSタスク実行ロールに `logs:CreateLogStream` や `logs:PutLogEvents` が含まれていない場合は上記のポリシーに追加してください。

これでサービスをデプロイするとCloudWatchでECSのログを確認できるようになります。