---
layout: my-post
title: "LaravelプロジェクトをFargateを使用してECSにデプロイする"
date: 2025-07-31 00:00:00 +0000
categories: aws ecs
page_name: deploy-laravel-to-ecs-using-fargate
lang: ja
---

Fargateを利用してAmazon ECSにLaravelプロジェクトをデプロイします。

## 参考
- [Pushing a Docker image to an Amazon ECR private repository](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html)
- [Learn how to create an Amazon ECS Linux task for the Fargate launch type](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/getting-started-fargate.html)
- [Amazon ECS task execution IAM role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html)

## 前提
- AWSアカウント
- IAM Identity Centerにユーザーが作成済み  
詳しくは[こちらの手順](/aws/iam/create-user-with-administrative-access-in-iam-identity-center)を参照してください。
- AWS CLIがインストール済み  
詳しくは[こちらの手順](/aws/cli/install-aws-cli-on-linux)を参照してください。
- AWS CLI用のSSOとプロファイルが設定済み  
詳しくは[こちらの手順](/aws/cli/configure-sso-session-and-profile-with-aws-cli)を参照してください。

## 環境
- Ubuntu 22.04.3 LTS

## デプロイ手順
1. [Laravelプロジェクトの作成](#1-laravelプロジェクトの作成)
2. [リポジトリの作成](#2-リポジトリの作成)
3. [Dockerイメージのプッシュ](#3-dockerイメージのプッシュ)
4. [クラスターの作成](#4-クラスターの作成)
5. [ECSタスク実行用IAMロールの作成](#5-ecsタスク実行用iamロールの作成)
6. [タスク定義の作成](#6-タスク定義の作成)
7. [サービスの作成](#7-サービスの作成)
8. [Laravelプロジェクトの確認](#8-laravelプロジェクトの確認)

## 1. Laravelプロジェクトの作成
Laravelプロジェクトを作成します。  
詳細は[こちらの記事](/web-application-framework/laravel/running-laravel-project-on-nginx)を参照してください。

上記の例を使う場合、`Dockerfile` を以下のように変更してください。  
（プロジェクトソースをコンテナにコピーする処理と、いくつかのファイルのコピー元のパスを調整するための変更です）

`Dockerfile`:

```Dockerfile
FROM amazonlinux:2023

# （追加）プロジェクトソースをコンテナにコピー
COPY . /srv/example.com

# （変更）ファイルコピー処理のパスを調整
# 例: nginx/nginx.repo -> docker/web/nginx/nginx.repo

# NGINXインストール
RUN yum -y install yum-utils
COPY docker/web/nginx/nginx.repo /etc/yum.repos.d/nginx.repo
RUN yum -y install nginx-1.24.0
COPY docker/web/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf

# php-fpmとphp-mysqlndインストール
RUN yum -y install php8.2-fpm php8.2-mysqlnd
# 事前にディレクトリを作成しておかないとエラーになる
RUN mkdir /run/php-fpm
RUN mkdir /var/run/php
COPY docker/web/php/php-fpm.d/zzz-www.conf /etc/php-fpm.d/zzz-www.conf
```

ローカルでコンテナを起動する場合は `docker-compose.yml` も修正してください。  
コンテナをビルドするパスをプロジェクトルートにし、`Dockerfile` のパスを指定する必要があります。

`docker-compose.yml`:

```yml
services:
  web:
    # "build" を変更
    build:
      context: .
      dockerfile: ./docker/web/Dockerfile
    volumes:
      - .:/srv/example.com
    ports:
      - "8080:80"
    command: bash -c "chmod 755 /srv/example.com/docker/web/start.sh && /srv/example.com/docker/web/start.sh"
```

## 2. リポジトリの作成
AWS管理コンソールから[ECR](https://console.aws.amazon.com/ecr/)に移動し、Createボタンをクリックします。

![AWS管理コンソールのECR画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "AWS管理コンソールのECR画面")

任意のリポジトリ名を入力し（例：`laravel-project-repository`）、Createをクリックします。

![リポジトリ作成画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "リポジトリ作成画面")

## 3. Dockerイメージのプッシュ
以下の形式でDockerイメージをビルドします。

```bash
$ docker build -t <AWSアカウントID>.dkr.ecr.<ECRリポジトリリージョン>.amazonaws.com/laravel-project-repository .
```

`Dockerfile` がプロジェクトルートにない場合は `-f` オプションでファイルのパスを指定します。

```bash
$ docker build -t <AWSアカウントID>.dkr.ecr.<ECRリポジトリリージョン>.amazonaws.com/laravel-project-repository -f <Dockerfileのパス> .
```

すでにイメージがある場合は、以下のようにタグを付け直します。

```bash
$ docker tag <DockerイメージリポジトリまたはID>:<Dockerイメージタグ> <AWSアカウントID>.dkr.ecr.<ECRリポジトリリージョン>.amazonaws.com/laravel-project-repository:latest
```

AWS CLIを使ってAmazon ECRにログインします。

```bash
$ aws ecr get-login-password --region <ECRリポジトリリージョン> | docker login --username AWS --password-stdin <AWSアカウントID>.dkr.ecr.<ECRリポジトリリージョン>.amazonaws.com
```

その後、イメージをECRにプッシュします。

```bash
$ docker push <AWSアカウントID>.dkr.ecr.<ECRリポジトリリージョン>.amazonaws.com/laravel-project-repository:latest
```

管理コンソールでイメージが確認できます。

![Dockerイメージ](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "Dockerイメージ")

## 4. クラスターの作成
管理コンソールの[ECS](https://console.aws.amazon.com/ecs/)にアクセスし、Create Clusterをクリックします。

![AWS管理コンソールのECS画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "AWS管理コンソールのECS画面")

クラスター名を入力し、「AWS Fargate (serverless)」を選択します。  
例として `laravel-project-cluster` と指定し、Createをクリックします。

![クラスター作成画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "クラスター作成画面")

## 5. ECSタスク実行用IAMロールの作成
コンテナやFargateエージェントにAWSサービスへのアクセス権限を与えるために、ECSタスク実行用のIAMロールを作成します。

管理コンソールの[IAM](https://console.aws.amazon.com/iam/)に移動し、「Roles」をクリックします。

![AWS管理コンソールのIAM画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image8.png "AWS管理コンソールのIAM画面")

Create roleをクリックします。

![ロール一覧画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image9.png "ロール一覧画面")

「AWS Service」をクリックし、「Elastic Container Service Task」を選択します。  
Nextをクリックします。

![Select Trusted Entityスクリーン1](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image10.png "Select Trusted Entityスクリーン1")

![Select Trusted Entityスクリーン2](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image11.png "Select Trusted Entityスクリーン2")

ポリシー一覧から `AmazonECSTaskExecutionRolePolicy` を選択し、必要に応じてポリシー（例：S3アクセス）を追加します。  
Nextをクリックします。

![Add Permissionsスクリーン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image12.png "Add Permissionsスクリーン")

ロール名に `ecsTaskExecutionRole`（任意の名前）を入力し、Create roleをクリックして完了です。

![最終スクリーン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image13.png "最終スクリーン")

## 6. タスク定義の作成
タスク定義はECSタスクを作成する際に使用されます。  
ここでは前節の[Laravelプロジェクト](#1-laravelプロジェクトの作成)用にタスク定義を作成します。

管理コンソールのTask definitionsにアクセスします。

![AWS管理コンソールのECS](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "AWS管理コンソールのECS")

「Create new task definition with JSON」を選択します。

![タスク定義一覧画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png "タスク定義一覧画面")

`family` に `laravel-project-fargate`、コンテナ名を `web` とします。  
`image` にプッシュしたイメージのURIを指定し、`portMappings` と `command` を追加します。

`task definition`:

```json
{
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "family": "laravel-project-fargate",
    "containerDefinitions": [
        {
            "name": "web",
            "image": "<Your AWS account ID>.dkr.ecr.<Your ECS repository region>.amazonaws.com/laravel-project-repository:latest",
            "portMappings": [
                {
                    "containerPort": 80,
                    "protocol": "tcp"
                }
            ],
            "essential": true,
            "command": [
                "/bin/sh",
                "-c",
                "chmod 755 /srv/example.com/docker/web/start.sh && /srv/example.com/docker/web/start.sh"
            ]
        }
    ],
    "volumes": [],
    "networkMode": "awsvpc",
    "memory": "3 GB",
    "cpu": "1 vCPU",
    "executionRoleArn": "ecsTaskExecutionRole"
}
```

Createをクリックします。

## 7. サービスの作成
再びECSクラスター画面に戻り、対象のクラスターを選択します。

![クラスター一覧画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image14.png "クラスター一覧画面")

Createをクリックします。

![サービス一覧画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image15.png "サービス一覧画面")

タスク定義ファミリーを入力すると、最新リビジョンとサービス名が自動入力されます（例ではサービス名を変更しています）。

![サービス作成画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image16.png "サービス作成画面")

VPC、サブネット、セキュリティグループを選択します。  
この例ではパブリックサブネットと、任意のIPからのHTTPアクセスをインバウンド・アウトバウンドルールに定義し、任意のIPからのHTTPSアクセスをアウトバウンドルールに定義したセキュリティグループを選択しています。  
（任意のデバイスからLaravelのWebページを見るため(HTTP)と、ECSサービスにECRリポジトリからのイメージのプルを許可するため(HTTPS)）

「Public IP」をオンにして、Createをクリックします。

![サービス作成画面のネットワーク](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image17.png "サービス作成画面のネットワーク")

## 8. Laravelプロジェクトの確認
クラスター画面からサービスを選びます。

![サービス一覧画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image18.png "サービス一覧画面")

稼働中のタスクをクリックします。

![タスク一覧画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image19.png "タスク一覧画面")

タスク詳細の「Configuration」からパブリックIPアドレスを取得し、ブラウザで `http://<パブリックIP>` にアクセスします。

LaravelのWebページが表示されれば、ECSへのデプロイは成功です。

![LaravelのWebページ](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image20.png "LaravelのWebページ")