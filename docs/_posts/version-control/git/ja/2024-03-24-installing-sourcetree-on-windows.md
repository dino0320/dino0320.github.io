---
layout: my-post
title: "WindowsでSourcetreeをインストールする"
date: 2024-03-24 00:00:00 +0000
categories: version-control git
lang: ja
---

WindowsでSourcetreeをインストールします。

## Sourcetreeとは
Sourcetreeは無料のGitクライアントです。  
GUIでファイルの差分が見やすく、Gitの操作も簡単になります。

## 環境
- Windows 10 64ビット
- Git for Windows 2.44.0

## 前提
- Gitをインストールしている。

## インストールの流れ
1. [Sourcetreeをダウンロードする。](#1-sourcetreeをダウンロードする)
2. [Sourcetreeをインストールする。](#2-sourcetreeをインストールする)
3. [インストールの確認をする。](#3-インストールの確認をする)

## 1. Sourcetreeをダウンロードする
[Sourcetreeの公式サイト](https://www.sourcetreeapp.com/)にアクセスし、「Download for Windows」をクリックします。  
![Download for Windowsボタン](/assets/images/version-control/git/installing-sourcetree-on-windows/image1.png "Download for Windowsボタン")   
ライセンス契約とプライバシーポリシーに目を通し、「I agree to the Atlassian Software License Agreement and Privacy Policy.」にチェックを入れます。「Download」ボタンをクリックします。  
![ライセンス等同意ダイアログ](/assets/images/version-control/git/installing-sourcetree-on-windows/image2.png "ライセンス等同意ダイアログ")  

## 2. Sourcetreeをインストールする
ダウンロードしたインストーラーを実行します。今回は `SourceTreeSetup-3.4.17.exe` を実行します。  

### Bitbucket Cloudアカウント登録画面
Bitbucket Cloudアカウントの登録ができますが、今回はスキップします。「スキップ」ボタンをクリックします。  
![Bitbucket Cloudアカウント登録画面](/assets/images/version-control/git/installing-sourcetree-on-windows/image3.png "Bitbucket Cloudアカウント登録画面")  

### ツール選択画面
必要なツールを選択して「次へ」ボタンをクリックします。  
![ツール選択画面](/assets/images/version-control/git/installing-sourcetree-on-windows/image4.png "ツール選択画面")

|項目|説明|
|----|----|
|Git|Gitをインストールするかどうかです。今回は事前にインストールしていたので、自動的に選択されています。|
|Mercurial|Mercurialをインストールするかどうかです。Mercurialはバージョン管理システムです。今回は必要ないのでチェックを外しています。|
|改行の自動処理を設定する|チェックアウトするとき自動的に改行コードをLFからCRLFに変換するかどうかです。今回はチェックを入れています。|
|Configure Global Ignore|IDEなどの無視すべきファイルを設定したグローバルな無視設定ファイルを使うかどうかです。今回はチェックを外しています。|

### プリファレンス画面
コミット時に利用されるユーザー名とメールアドレスを入力して「次へ」ボタンをクリックします。  
![プリファレンス画面](/assets/images/version-control/git/installing-sourcetree-on-windows/image5.png "プリファレンス画面")

インストールが始まります。  
SSHキーを読み込むかどうかを聞かれますが、今回は「いいえ」ボタンをクリックします。  
![SSHキーを読み込むかどうかのダイアログ](/assets/images/version-control/git/installing-sourcetree-on-windows/image6.png "SSHキーを読み込むかどうかのダイアログ")

## 3. インストールの確認をする
インストールが完了すると自動的にSourcetreeが開きます。  
試しにリポジトリを追加してみます。「Add」ボタンをクリックします。  
![リポジトリ追加ボタン](/assets/images/version-control/git/installing-sourcetree-on-windows/image7.png "リポジトリ追加ボタン")  
リポジトリのパスを入力し、「追加」ボタンをクリックします。  
![リポジトリ追加画面](/assets/images/version-control/git/installing-sourcetree-on-windows/image8.png "リポジトリ追加画面")  
リポジトリを追加できました。