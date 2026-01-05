---
layout: my-post
title: "IAM Identity Center で管理者アクセス権ユーザーを作成する手順"
date: 2025-07-05 00:00:00 +0000
categories: aws iam
page_name: create-user-with-administrative-access-in-iam-identity-center
lang: ja
image: /assets/images/aws/iam/create-user-with-administrative-access-in-iam-identity-center/image10.png
---

この記事では、AWS Management Consoleを使用して、IAM Identity Center上に管理者権限を持つユーザーを作成する手順を解説します。  
AWSアカウントのルートユーザーは通常サービスの操作に使用すべきではありません。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image10.png "サムネイル")

## 参考
- [Set up to use Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/get-set-up-for-amazon-ecs.html)

## 前提条件
- AWSアカウントのルートユーザーを保有している

## 作業の流れ
1. [ユーザーを作成する](#1-ユーザーを作成する)
2. [パーミッションセットを作成する](#2-パーミッションセットを作成する)
3. [ユーザーにパーミッションセットを割り当てる](#3-ユーザーにパーミッションセットを割り当てる)
4. [AWS Management Console にアクセスする](#4-aws-management-console-にアクセスする)

## 1. ユーザーを作成する
AWSアカウントのルートユーザーのメールアドレスでAWS Management Consoleにサインインします。

![Sign in form](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Sign in form")

![Sign in form using root user](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "Sign in form using root user")

IAM Identity Centerに移動して、有効化します。

![How to access IAM Identity Center](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "How to access IAM Identity Center")

"Users" をクリックします。

![Users in IAM Identity Center](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "Users in IAM Identity Center")

"Add User" をクリックし、"Primary Information" を入力します。  
その他の項目は任意です。  
今回は "Send an email to this user with password setup instructions."（このユーザーにパスワード設定手順をメールで送信）を選択しました。

![User Primary Information](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "User Primary Information")

"Next" をクリックすると、任意でユーザーをグループに追加できます。  
最後に "Add user" をクリックして作成を完了します。

作成後、ユーザー宛てに招待メールが届きます。  
"Accept Invitation" をクリックし、パスワードを設定します。  
その後、多要素認証（MFA）を有効にします。  
今回は "Authenticator apps" を選び、Google Authenticatorを使用しました。

セットアップ完了後、AWS access portalにアクセスできるようになります。  
次回以降は、メールに記載されたアクセス用URLを使用してログインします。

## 2. パーミッションセットを作成する
管理者アクセスを付与するために、パーミッションセット（Permission Set）を作成します。

AWSアカウントのルートユーザーでIAM Identity Centerに戻り、"Permission sets" をクリックします。

![Permission sets in IAM Identity Center](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "Permission sets in IAM Identity Center")

"Create permission set"（パーミッションセットの作成）をクリックします。  
"Predefined permission set"（定義済みパーミッションセット）から"AdministratorAccess" を選択します。

![Select permission set type](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png "Select permission set type")

他の項目は任意です。"Create" をクリックして完了します。

## 3. ユーザーにパーミッションセットを割り当てる
IAM Identity Center内の "AWS accounts" をクリックします。

![AWS accounts in IAM Identity Center](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "AWS accounts in IAM Identity Center")

自分のAWSアカウントを選択し、"Assign users or groups"（ユーザーまたはグループを割り当て）」をクリックします。  
"Users" タブで先ほど作成したユーザーを選択し、"Next" をクリックします。  
"AdministratorAccess" のパーミッションセットを選択し、"Next" をクリックします。  
最後に "Submit" をクリックして割り当てを完了します。

この操作により、作成したユーザーに管理者アクセス権が付与されます。

## 4. AWS Management Console にアクセスする
管理者アクセス権を持つユーザーでAWS Management Consoleにログインできるか確認します。

ユーザー作成時に送信されたメールに記載されたAWSアクセスポータルのリンクにアクセスします。　　
"AdministratorAccess" をクリックすると、AWS Management Consoleに入ることができます。

![AWS access portal](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image9.png "AWS access portal")
