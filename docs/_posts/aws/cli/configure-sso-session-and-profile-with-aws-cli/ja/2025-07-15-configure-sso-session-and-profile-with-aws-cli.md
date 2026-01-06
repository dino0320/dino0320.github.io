---
layout: my-post
title: "AWS CLIでIAM Identity Center (SSO) を設定してプロファイルを作成する"
date: 2025-07-15 00:00:00 +0000
categories: aws cli
page_name: configure-sso-session-and-profile-with-aws-cli
lang: ja
image: /assets/images/aws/cli/configure-sso-session-and-profile-with-aws-cli/image3.png
---

この記事では、AWS CLIを使用してSSOとプロファイルを設定する方法を説明します。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "サムネイル")

## 参考
- [Configuring IAM Identity Center authentication with the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)

## 前提
- AWS アカウントを持っていること
- IAM Identity Centerにユーザーが存在していること  
詳細は[こちらの手順](/aws/iam/create-user-with-administrative-access-in-iam-identity-center)をご確認ください。
- AWS CLIがすでにインストールされていること  
インストール手順は[こちら](/aws/cli/install-aws-cli-on-windows)をご覧ください。

## 環境
- Windows 10 64ビット

## 設定手順
1. [SSO Start URLとSSO Regionの確認](#1-sso-start-urlとsso-regionの確認)
2. [SSOとプロファイルの設定](#2-ssoとプロファイルの設定)
3. [SSOサインインとサインアウト](#3-ssoサインインとサインアウト)

## 1. SSO Start URLとSSO Regionの確認
AWSアクセスポータルにアクセスします。  
詳細は[こちらの記事](/aws/iam/create-user-with-administrative-access-in-iam-identity-center)をご参照ください。

「Access Keys」をクリックします。

![AWSアクセスポータル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "AWSアクセスポータル")

「Windows」を選択します。  
「AWS IAM Identity Center credentials (Recommended)」セクションに、SSO start URLとSSO Regionが表示されます。

![資格取得画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "資格取得画面")

## 2. SSOとプロファイルの設定
コマンドプロンプトを開き、以下のコマンドを実行します。

```
>aws configure sso
```

以下の項目を入力して、SSOを設定します。

```
SSO session name (Recommended): my-sso  # 任意の名前
SSO start URL [None]: <取得したSSO start URL>
SSO region [None]: <取得したSSO Region>
SSO registration scopes [sso:account:access]: sso:account:access  # デフォルト値でいい
```

デフォルトのブラウザが起動し、サインイン画面が表示されます。  
IAM Identity Centerのユーザーとしてサインインし、AWS CLIにデータへのアクセスを許可します。

サインイン後、コマンドプロンプトに戻ります。  
以下の情報を入力してプロファイルを作成します。

| 項目 | 説明 |
|-------|-------------|
| AWS account | 利用するAWSアカウントを選択します。アカウントが1つだけの場合、自動で選択されます。 |
| IAM role | ユーザーに割り当てられた IAMロールを選択します。ロールが1つだけの場合、自動で選択されます。 |
| Default client Region | デフォルトのリージョンを入力します。AWS CLI実行時に、何も指定しなければこのリージョンが使われます。 |
| CLI default output format | 出力形式を入力します。形式の選択肢は[こちら](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html#cli-config-output)を参照してください。 |
| Profile name | 任意の名前を入力します。`default` と入力すれば、AWS CLIの実行時に `--profile` オプションを省略できます。 |

以下は入力例です。

```
The only AWS account available to you is: <あなたのAWSアカウントID>
Using the account ID <あなたのAWSアカウントID>
The only role available to you is: AdministratorAccess
Using the role name "AdministratorAccess"
Default client Region [None]: ap-northeast-1
CLI default output format (json if not specified) [None]: json
Profile name [AdministratorAccess-507911341149]: my-dev-profile
```

次のコマンドを実行して、SSOとプロファイルが正しく動作しているか確認します。  
（プロファイル名に `default` を指定した場合は、`--profile` オプションは不要です）

```
aws sts get-caller-identity --profile my-dev-profile
{
    "UserId": "<あなたのユーザーID>",
    "Account": "<あなたのAWSアカウントID>",
    "Arn": "<あなたのユーザーのArn>"
}
```

ユーザー情報が表示されれば、SSOとプロファイルの設定は完了です。

## 3. SSOサインインとサインアウト
次回以降は、以下のコマンドでSSOサインインできます。  
（プロファイル名に `default` を指定した場合は、`--profile` は省略可能です）

```
>aws sso login --profile my-dev-profile
```

デフォルトのブラウザが起動し、サインイン画面が表示されます。  
IAM Identity Centerのユーザーとしてサインインしてください。

その後、次のコマンドを実行してサインインできているか確認します。

```
aws sts get-caller-identity --profile my-dev-profile
{
    "UserId": "<あなたのユーザーID>",
    "Account": "<あなたのAWSアカウントID>",
    "Arn": "<あなたのユーザーのArn>"
}
```

ユーザー情報が表示されれば、サインインに成功しています。

サインアウトするには、以下のコマンドを実行します。

```
>aws sso logout
```