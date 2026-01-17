---
layout: my-post
title: "プライベートECS Fargateコンテナから外部にアクセスする方法【NAT Gateway】"
date: 2026-01-17 00:00:00 +0000
categories: aws ecs
page_name: how-to-access-external-services-from-private-ecs-container
lang: ja
image: /assets/images/aws/ecs/how-to-access-external-services-from-private-ecs-container/image1.png
---

プライベートなECS FargateコンテナからNAT Gatewayを経由して外部サービスにアクセスする方法をまとめました。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 前提
- プロジェクトがECSにデプロイされている  
LaravelプロジェクトをECSにデプロイする方法については[こちら](/aws/ecs/deploy-laravel-to-ecs-using-fargate)をご覧ください。

## 手順
1. [NAT Gatewayを作成する](#1-nat-gatewayを作成する)
2. [プライベートルートテーブルを編集する](#2-プライベートルートテーブルを編集する)

## 1. NAT Gatewayを作成する
AWSマネジメントコンソールの[NAT Gateway一覧](https://console.aws.amazon.com/vpcconsole/home#NatGateways)にアクセスします。  
「Create NAT Gateway」をクリックしてください。

![AWSマネジメントコンソールのNAT Gateway一覧画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "AWSマネジメントコンソールのNAT Gateway一覧画面")

任意の名前を入力し、VPCを選択します。  
「Availability mode」はデフォルトで「Regional」（自動設定）に設定されているはずです。特定のアベイラビリティゾーン用に設定したい場合は「Zonal」を選択して設定を行ってください。  
また、「Method of Elastic IP (EIP) allocation」はデフォルトで「Automatic」（自動設定）に設定されていると思います。事前に作成したElastic IPアドレスを使用したい場合は「Manual」を選択してください。  
「Connectivity type」が「Public」になっていることを確認し、「Create NAT Gateway」をクリックしてください。

![NAT Gateway作成画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "NAT Gateway作成画面")

## 2. プライベートルートテーブルを編集する
AWSマネジメントコンソールの[ルートテーブル一覧](https://console.aws.amazon.com/vpcconsole/home#RouteTables)にアクセスします。  
プライベートルートテーブルを選択し、「Edit routes」をクリックしてください。

![プライベートルートテーブル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "プライベートルートテーブル")

以下のルートを追加し、「Save changes」をクリックします。  
これによりすべてのアウトバウンド通信がNAT Gatewayを経由してインターネットに送信されるようになります。

|Destination|Target|
|-----------|------|
|0.0.0.0/0|NAT Gateway ([上で作成したもの](#1-nat-gatewayを作成する))|

![ルートテーブル編集画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "ルートテーブル編集画面")

これでプライベートなECSコンテナから外部APIやAmazon S3などのAWSサービスにアクセスできるようになります。  
※Amazon S3などのAWSサービスについてはNAT Gatewayの代わりにVPCエンドポイントを使用することを推奨します（コスト削減）。