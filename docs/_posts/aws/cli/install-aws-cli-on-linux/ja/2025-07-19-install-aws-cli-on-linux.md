---
layout: my-post
title: "UbuntuにAWS CLI v2をインストールする方法"
date: 2025-07-19 00:00:00 +0000
categories: aws cli
page_name: install-aws-cli-on-linux
lang: ja
image: /assets/images/aws/cli/install-aws-cli-on-linux/image1.png
---

この記事では、UbuntuにAWS CLIをインストールする方法を説明します。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 参考
- [Installing or updating to the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

## 環境
- Ubuntu 22.04.3 LTS

AWS CLIをインストールするには、`unzip` ユーティリティが必要です。  
お使いの環境に `unzip` がインストールされていない場合は、次のコマンドを実行してください。

```bash
$ sudo apt install unzip
```

次に、以下のコマンドでAWS CLIをインストールします。

```bash
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ unzip awscliv2.zip
$ sudo ./aws/install
```

インストールされたか確認するために、次のコマンドを実行します。

```bash
aws --version
aws-cli/2.27.55 Python/3.13.4 Linux/6.6.87.2-microsoft-standard-WSL2 exe/x86_64.ubuntu.22
```

AWS CLIのバージョン情報が表示されれば、インストールは成功しています。