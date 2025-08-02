---
layout: my-post
title: "GitHub Actionsを使ったECSへの自動デプロイ"
date: 2025-08-02 00:00:00 +0000
categories: aws ecs
page_name: automated-deployment-to-ecs-using-github-actions
lang: ja
---

この記事では、GitHub Actionsを利用してAmazon ECSへのデプロイを自動化する方法について解説します。

## 参考
- [Deploying to Amazon Elastic Container Service](https://docs.github.com/en/actions/how-tos/deploy/deploy-to-third-party-platforms/amazon-elastic-container-service)
- [Configuring OpenID Connect in Amazon Web Services](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws)
- [Configuring a role for GitHub OIDC identity provider](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html#idp_oidc_Create_GitHub)
- [GitHub Actions から AWS へアクセスする](https://qiita.com/yh1224/items/2a8223201b48a5c41e7a)

## 前提
- AWSアカウント
- IAM Identity Centerに登録されたユーザー  
詳しくは[こちらの手順](/aws/iam/create-user-with-administrative-access-in-iam-identity-center)を参照してください。
- GitHubリポジトリ

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- Laravel 11

## デプロイ手順
1. [プロジェクトをデプロイする](#1-プロジェクトをデプロイする)
2. [OIDC IDプロバイダを作成する](#2-oidc-idプロバイダを作成する)
3. [GitHub OIDC IDプロバイダ用のロールを作成する](#3-github-oidc-idプロバイダ用のロールを作成する)
4. [リポジトリのシークレットにロールを追加する](#4-リポジトリのシークレットにロールを追加する)
5. [リポジトリ内にタスク定義を作成する](#5-リポジトリ内にタスク定義を作成する)
6. [ワークフローを作成する](#6-ワークフローを作成する)
7. [ワークフローを確認する](#7-ワークフローを確認する)

## 1. プロジェクトをデプロイする
まず、何かプロジェクトをECSにデプロイします。  
詳細は[こちらのガイド](/aws/ecs/deploy-laravel-to-ecs-using-fargate)を参照してください。

## 2. OIDC IDプロバイダを作成する
AWSの認証にはOpenID Connect（OIDC）を使用します。  
[このページ](https://docs.github.com/en/actions/how-tos/deploy/deploy-to-third-party-platforms/amazon-elastic-container-service)によると、OpenID Connect (OIDC)を使用すると、GitHub ActionsのワークフローがAWSのリソースにアクセスできるようになり、長期保存のAWS認証情報をGitHubのシークレットとして保存する必要がなくなります。

> OpenID Connect (OIDC) allows your GitHub Actions workflows to access resources in Amazon Web Services (AWS), without needing to store the AWS credentials as long-lived GitHub secrets.

OIDC IDプロバイダを作成するには、AWSマネジメントコンソールの[IAM](https://console.aws.amazon.com/iam/)を開き、「Identity providers」をクリックします。

![IAMのサイドバー](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "IAMのサイドバー")

「Add provider」をクリックします。

![Identityプロバイダー一覧画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "Identityプロバイダー一覧画面")

「OpenID Connect」を選択し、以下の情報を入力します（[こちらのドキュメント](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws)参照）

| Provider URL | https://token.actions.githubusercontent.com |
| Audience | sts.amazonaws.com |

![Identityプロバイダー作成画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "Identityプロバイダー作成画面")

## 3. GitHub OIDC IDプロバイダ用のロールを作成する
[IAM](https://console.aws.amazon.com/iam/)の「Roles」に移動します。

![IAMのサイドバー](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "IAMのサイドバー")

「Create role」をクリックします。

![ロール一覧画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "ロール一覧画面")

「Web identity」を選択します。

![Select trusted entity画面1](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "Select trusted entity画面1")

以下の情報を入力して「Next」をクリックします。

| 項目 | 説明 |
|-------|-------------|
| Identity provider | 作成したOIDC IDプロバイダ |
| Audience | 設定したAudience |
| GitHub organization| GitHubユーザー名 |
| GitHub repository | このロールを使用するリポジトリ名。すべてのリポジトリに適用するには `*` を使用する。 |
| GitHub branch | 対象ブランチ。すべてのブランチに適用するには `*` を使用する。 |

![Select trusted entity画面2](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png "Select trusted entity画面2")

「Add permission」画面でも「Next」をクリックします。  
任意のロール名と説明を入力し、「Create role」をクリックします。

![最終画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image8.png "最終画面")

次に、必要なポリシー（ECR、ECS、IAMについて）をインラインポリシーとして追加します。  
AWSマネジメントコンソール上で、作成したロールをクリックして「Create inline policy」をクリックします。

![ロール一覧画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image9.png "ロール一覧画面")

JSONタブに切り替え、以下のポリシーを入力します。

ECRについて:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ecr:BatchGetImage",
                "ecr:CompleteLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:InitiateLayerUpload",
                "ecr:BatchCheckLayerAvailability",
                "ecr:PutImage"
            ],
            "Resource": "arn:aws:ecr:<Your repository region>:<Your Account ID>:repository/<Your repository name>"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "ecr:GetAuthorizationToken",
            "Resource": "*"
        }
    ]
}
```

「Next」をクリックし、任意のポリシー名を入力して「Create policy」をクリックします。

ECSとIAMについても同様に作成します。

ECSについて:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ecs:UpdateService",
                "ecs:RegisterTaskDefinition",
                "ecs:DescribeServices"
            ],
            "Resource": [
                "arn:aws:ecs:<Your cluster region>:<Your Account ID>:service/<Your cluster name>/*",
                "arn:aws:ecs:<Your cluster region>:<Your Account ID>:task-definition/<Your task definition name>:*"
            ]
        }
    ]
}
```

IAMについて:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::<Your Account ID>:role/<Your task execution role name>"
        }
    ]
}
```

## 4. リポジトリのシークレットにロールを追加する
GitHubリポジトリに移動し、「Settings」をクリックします

![GitHubリポジトリ](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image10.png "GitHubリポジトリ")

「Secrets and variables」 > 「Actions」をクリックします。

Click "Secrets and Variables" > "Actions".

![Settings](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image11.png "Settings")

「New repository secret」をクリックします。

![Actions secrets and variables](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image12.png "Actions secrets and variables")

名前（例：`ROLE_TO_ASSUME`）と、先ほど作成したロールのARNを入力して「Add secret」をクリックします。

![シークレット作成画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image13.png "シークレット作成画面")

## 5. リポジトリ内にタスク定義を作成する
`.aws/task-definition.json` などのファイル名で、タスク定義のJSONファイルをリポジトリ内に保存します。

## 6. ワークフローを作成する
GitHubリポジトリの「Actions」をクリックします。

![GitHubリポジトリ](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image14.png "GitHubリポジトリ")

「New workflow」をクリックします。

![Actions](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image15.png "Actions")

「Deploy to Amazon ECS」の「Configure」を選択します。

![Choose a workflow画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image16.png "Choose a workflow画面")

このワークフローでは、DockerイメージをビルドしてECRにプッシュし、タスク定義の「image」をプッシュしたものに置き換え、ECSにデプロイします。

YAMLファイル内の環境変数を実際の値に置き換えます。

```yml
AWS_REGION: MY_AWS_REGION                   # 好きなAWSリージョン 例: us-west-1
ECR_REPOSITORY: MY_ECR_REPOSITORY           # ECRリポジトリ名
ECS_SERVICE: MY_ECS_SERVICE                 # ECSサービス名
ECS_CLUSTER: MY_ECS_CLUSTER                 # ECSクラスター名
ECS_TASK_DEFINITION: MY_ECS_TASK_DEFINITION # GitHubリポジトリのECSタスク定義のパス
                                               # 例: .aws/task-definition.json
CONTAINER_NAME: MY_CONTAINER_NAME           # タスク定義の「containerDefinitionsset」にあるコンテナ名
```

OIDC用に `id-token: write` を `permissions` に追加します。

```yml
permissions:
  id-token: write # 追加
  contents: read
```

「Configure AWS credentials」ステップを以下のように修正します。

{% raw %}

```yml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v1
  with:
    role-to-assume: ${{ secrets.ROLE_TO_ASSUME }}  # 変更
    aws-region: ${{ env.AWS_REGION }}
```

{% endraw %}

`Dockerfile` がルートディレクトリ以外にある場合は `-f` オプションでパスを指定します。

例:

```yml
docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -f ./docker/web/Dockerfile .
```

ファイル名を入力し（例: `aws.yml`）、「Commit changes」をクリックします。

![Workflow生成画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image17.png "Workflow生成画面")

このワークフローは `main` ブランチに変更がプッシュされたときに実行されるため、上記のコミットで実行されたはずです。

```yml
on:
  push:
    branches: [ "main" ]
```

## 7. ワークフローを確認する
GitHubリポジトリの「Actions」をクリックします。

![GitHubリポジトリ](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image14.png "GitHubリポジトリ")

以下のような画面が表示されていればワークフローは正しく完了しています。

![Workflow画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image18.png "Workflow画面")