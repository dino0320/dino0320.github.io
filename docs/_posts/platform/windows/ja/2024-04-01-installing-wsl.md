---
layout: my-post
title: "WSLをインストールする"
date: 2024-04-01 00:00:00 +0000
categories: platform windows
lang: ja
---

WSLをインストールします。

## WSLとは
Windows Subsystem for Linuxの略で、Windows上でLinux環境を使うことができます。

## 環境
- Windows 10 64ビット

## インストールの流れ
1. [WSLとUbuntuをインストールする。](#1-wslとubuntuをインストールする)
2. [PCを再起動する。](#2-pcを再起動する)
3. [UNIXユーザーを作成する。](#3-unixユーザーを作成する)
4. [インストールの確認をする。](#4-インストールの確認をする)

## 1. WSLとUbuntuをインストールする
WSLとUbuntuをインストールします。    
コマンドプロンプトかPowerShellを管理者として開き、以下を実行します。  
WSLコマンドとUbuntuがインストールされます。
```
> wsl --install
```
WSLがインストール済みでUbuntuがインストールされてない場合、以下を実行します。
```
> wsl --install -d Ubuntu
```

## 2. PCを再起動する
PCを再起動します。  
再起動するとUbuntuが起動するので、少し待ちます。  
![Ubuntu起動画面](/assets/images/platform/windows/installing-wsl/image1.png "Ubuntu起動画面")

## 3. UNIXユーザーを作成する
待っているとUNUXユーザーを作成するように言われます。  
任意のユーザー名とパスワードを入力してUNUXユーザーを作成します。  
![UNIXユーザー作成画面](/assets/images/platform/windows/installing-wsl/image2.png "UNIXユーザー作成画面")  
Ubuntuが起動するまで待ちます。起動したらUbuntuを閉じて大丈夫です。  

## 4. インストールの確認をする
インストールできたかを確認するために、Linuxディストリビューションの一覧を確認します。  
コマンドプロンプトかPowerShellを開き、以下を実行します。
```
> wsl -l
Linux 用 Windows サブシステム ディストリビューション:
Ubuntu (既定)
```
インストールの確認ができました。