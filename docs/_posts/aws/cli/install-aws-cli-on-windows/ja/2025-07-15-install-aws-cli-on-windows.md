---
layout: my-post
title: "WindowsでAWS CLIをインストールする"
date: 2025-07-15 00:00:00 +0000
categories: aws cli
page_name: install-aws-cli-on-windows
lang: ja
---

この記事では、WindowsにAWS CLIをインストールする方法を説明します。  
※64ビット版のWindowsのみがサポートされています。

## 参考
- [Installing or updating to the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

## 前提
- AWSアカウントを持っていること

## 環境
- Windows 10 64ビット

## インストール手順
1. [インストーラーのダウンロード](#1-インストーラーのダウンロード)
2. [AWS CLIの確認](#2-aws-cliの確認)

## 1. インストーラーのダウンロード
最新バージョンのインストーラーを[こちら](https://awscli.amazonaws.com/AWSCLIV2.msi)からダウンロードします。  
バージョン情報は[こちら](https://raw.githubusercontent.com/aws/aws-cli/v2/CHANGELOG.rst)で確認できます。

ダウンロードした `.msi` ファイルを開きます。

「Next」をクリックします。

![セットアップウィザードの最初の画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "セットアップウィザードの最初の画面")

ライセンス契約を読み、「I accept the terms on the License Agreement（ライセンス契約の条項に同意する）」にチェックを入れます。  
「Next」をクリックします。

![ライセンス契約画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "ライセンス契約画面")

「Next」をクリックします。

![カスタムセットアップ画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "カスタムセットアップ画面")

「Install」をクリックします。

![インストール確認画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "インストール確認画面")

「Finish」をクリックします。

![セットアップ完了画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "セットアップ完了画面")

## 2. AWS CLIの確認
AWS CLIが正しくインストールされたか確認します。

Windowsのスタートメニューを開き、「cmd」と入力します。  
表示された「コマンドプロンプト」をクリックします。

以下のコマンドを実行して、AWS CLIのバージョンを確認します。

```
>aws --version
aws-cli/2.27.50 Python/3.13.4 Windows/10 exe/AMD64
```

バージョンが表示されれば、AWS CLIのインストールは成功しています。

AWS CLIの使用を開始するには、この[記事](/aws/cli/configure-sso-session-and-profile-with-aws-cli)を参照してください。