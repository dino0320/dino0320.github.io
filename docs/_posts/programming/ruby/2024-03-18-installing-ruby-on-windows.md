---
layout: my-post
title: "WindowsでRubyをインストールする"
date: 2024-03-18 00:00:00 -0000
categories: programming ruby
---

Windows環境でRubyをインストールします。

## 環境
- OS: Windows 10
- システムの種類: 64 ビット オペレーティング システム

## インストールの流れ
1. [RubyInstallerをダウンロードする。](#1-rubyinstallerをダウンロードする)
2. [Rubyをインストールする。](#2-rubyをインストールする)
3. [インストールの確認をする。](#3-インストールの確認をする)

## 1. RubyInstallerをダウンロードする
[こちら](https://rubyinstaller.org/downloads/)からRubyInstallerをダウンロードします。  
DevKitありならWITH DEVKIT、なしならWITHOUT DEVKITのインストーラーをダウンロードします。  
今回はWITH DEVKITの推奨バージョンをダウンロードします。  
![RubyInstaller WITH DEVKITの推奨バージョン](/assets/images/programming/ruby/2024-03-18-installing-ruby-on-windows/image1.png "RubyInstaller WITH DEVKITの推奨バージョン")

DevKitはMSYS2 toolchainのことで、MSYS2はRubyのネイティブC/C++拡張をビルドするのに必要とのことです。  
DevKitはRuby on Railsなどに必要です。  
以下は[RubyInstaller for Windows](https://rubyinstaller.org/downloads/)に記載されている説明の引用です。
> MSYS2 is required in order to build native C/C++ extensions for Ruby and is necessary for Ruby on Rails. Moreover it allows the download and usage of hundreds of Open Source libraries which Ruby gems often depend on.

### 2. Rubyをインストールする
ダウンロードしたRubyInstallerを実行します。  

#### ライセンス同意画面
ライセンス同意画面です。
1. ライセンス契約を読み、「I accept the License」を選択する。
2. 「Next」をクリックする。

![ライセンス同意画面](/assets/images/programming/ruby/2024-03-18-installing-ruby-on-windows/image2.png "ライセンス同意画面")

#### インストール先とオプションの設定画面
インストール先とオプションの設定画面です。
1. インストール先を設定する。  
インストール先を設定します。デフォルトから変更する場合、「Browse」をクリックしてパスを変更します。
2. オプションを設定する。  
「Add Ruby executables to your PATH」を選択すると、環境変数のPathにRuby実行ファイルのパスを追加します。  
rubyコマンドを実行できるようになります。(フルパスを入力せずに)  
「Associate .rb and .rbw files with this Ruby installation」を選択すると、「.rb」と「.rbw」のファイルをクリックしたときにRuby実行ファイルで開きます。
3. 「Install」をクリックする。

![インストール先とオプションの設定画面](/assets/images/programming/ruby/2024-03-18-installing-ruby-on-windows/image3.png "インストール先とオプションの設定画面")

#### コンポーネント選択画面
コンポーネント選択画面です。
1. 「Next」をクリックします。

![コンポーネント選択画面](/assets/images/programming/ruby/2024-03-18-installing-ruby-on-windows/image4.png "コンポーネント選択画面")

#### インストール完了画面
インストール完了画面です。
1. 「ridk install」を実行するかどうかを選択する。  
「Run 'ridk install' to set up MSYS2 and development toolchain.MSYS2 is required to install gems with C extensions.」を選択すると、DevKitのMSYS2をセットアップします。
2. 「Finish」をクリックする。  
「Run 'ridk install' to set up MSYS2 and development toolchain.MSYS2 is required to install gems with C extensions.」を選択した場合、MSYS2のセットアップが始まります。  
コマンドプロンプトが表示され、どのコンポーネントをインストールするか聞かれますが、デフォルトでいいはずなのでEnterを押します。

![インストール完了画面](/assets/images/programming/ruby/2024-03-18-installing-ruby-on-windows/image5.png "インストール完了画面")  
![MSYS2のセットアップ](/assets/images/programming/ruby/2024-03-18-installing-ruby-on-windows/image6.png "MSYS2のセットアップ")

### 3. インストールの確認をする
インストールできたかを確認するために、Rubyのバージョンを確認します。  
コマンドプロンプトかPowerShellを開き、以下を実行します。
```
> ruby -v
ruby 3.2.3 (2024-01-18 revision 52bb2ac0a6) [x64-mingw-ucrt]
```
インストールの確認ができました。