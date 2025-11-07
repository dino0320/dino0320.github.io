---
layout: my-post
title: "Visual Studio CodeでWSLに接続する"
date: 2024-04-01 01:00:00 +0000
categories: ide vscode
page_name: connecting-to-wsl-with-vscode
lang: ja
image: /assets/images/ide/vscode/connecting-to-wsl-with-vscode/image7.png
---

Visual Studio Code(VSCode)でWSLに接続します。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png "サムネイル")

## 環境
- Windows 10 64ビット
- Visual Studio Code 1.87.2

## 前提
- Visual Studio Codeをインストールしている。

## 設定の流れ
1. [WSLとUbuntuをインストールする。](#1-wslとubuntuをインストールする)
2. [VSCodeの拡張機能のWSLをインストールする。](#2-vscodeの拡張機能のwslをインストールする)
3. [リモートウィンドウを開く。](#3-リモートウィンドウを開く)

## 1. WSLとUbuntuをインストールする
WSLとUbuntuのインストールについては[こちら](/platform/windows/installing-wsl)をご覧ください。

## 2. VSCodeの拡張機能のWSLをインストールする
VSCodeを開き、「拡張機能」をクリックします。  
![拡張機能を開くボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "拡張機能を開くボタン")

検索バーに「wsl」と入力し、検索結果の「WSL」をインストールします。  
![拡張機能のWSL](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "拡張機能のWSL")

## 3. リモートウィンドウを開く
VSCodeの左下にある、リモートウィンドウを開くボタンをクリックします。  
![リモートウィンドウを開くボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "リモートウィンドウを開くボタン")

VSCodeの上部にオプションが表示されるので、「WSLへの接続」をクリックします。
![リモートウィンドウを開くオプション](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "リモートウィンドウを開くオプション")

WSLのデフォルトのディストリビューションであるUbuntuに接続されます。

WSLに接続したままWindows側のフォルダーを開く場合、「フォルダーを開く」ボタンをクリックします。  
![フォルダーを開くボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "フォルダーを開くボタン")

Windows側のフォルダーはWSLのUbuntuの `/mnt/` ディレクトリにマウントされているので、`/mnt/` ディレクトリ以下でフォルダを探します。開きたいフォルダーがCドライブの場合、`/mnt/c/` ディレクトリ以下にあります。  
例:  
![フォルダーのパス入力バー](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "フォルダーのパス入力バー")

フォルダーのパスを入力し、「OK」ボタンをクリックすとフォルダーが開かれます。