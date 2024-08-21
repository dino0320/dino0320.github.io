---
layout: my-post
title: "Chrome Remote Desktopを使ってみる"
date: 2024-08-02 00:00:00 +0000
categories: web-browser google-chrome
title_eng: set-up-chrome-remote-desktop
lang: ja
---

Chrome Remote Desktopを利用してパソコンから別のパソコンを操作してみます。

## 参考ページ
- [【AV1 VS VP9】AV1とVP9の違い：次世代ビデオコーデック徹底比較（画質、圧縮率、互換性、応用…） - Digiarty WinXDVD](https://www.winxdvd.com/video-convert/av1-vs-vp9.htm)

## 環境
### 接続先PC
- Windows 11
- Google Chrome 127.0.6533.89

### 接続元PC
- Windows 10 64ビット
- Google Chrome 127.0.6533.89

## もくじ
- [接続先PCでChrome Remote Desktopの設定をする。](#接続先pcでchrome-remote-desktopの設定をする)
- [接続元PCでChrome Remote Desktopの設定をする。](#接続元pcでchrome-remote-desktopの設定をする)
- [オプションパネルについて](#オプションパネルについて)
- [接続先pcの設定を削除する](#接続先pcの設定を削除する)

## 接続先PCでChrome Remote Desktopの設定をする
接続先PCでChrome Remote Desktopを設定します。

Chromeを開き、[chrome ウェブストア](https://chromewebstore.google.com/)にアクセスしてChrome Remote DesktopをChromeに追加します。  
ChromeではGoogleアカウントにログインする必要があります。

![chrome ウェブストアのChrome Remote Desktopノページ](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image1.png "chrome ウェブストアのChrome Remote Desktopノページ")

Chromeの拡張機能からChrome Remote Desktopを開きます。

![Chromeの拡張機能一覧](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image2.png "Chromeの拡張機能一覧")

リモート アクセスの設定のダウンロードボタンをクリックします。

![インストーラのダウンロードボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image3.png "インストーラのダウンロードボタン")

利用規約とプライバシーポリシーを読み、「同意してインストール」をクリックします。

![インストールの準備完了画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image4.png "インストールの準備完了画面")

ステータスバーまたはダウンロードフォルダから `chromeremotedesktophost.msi` という名前のインストーラを実行します。

リモート アクセスの設定に戻り、「オンにする」をクリックします。

![リモートアクセスを許可するボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image5.png "リモートアクセスを許可するボタン")

任意のPC名を入力します。

![名前の選択画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image6.png "名前の選択画面")

任意のPINを入力します。

![PINの入力画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image7.png "PINの入力画面")

「起動」をクリックすると、このPCに別のPCから接続できるようになります。

![接続できるPC一覧](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image8.png "接続できるPC一覧")

## 接続元PCでChrome Remote Desktopの設定をする
接続元PCでChrome Remote Desktopを設定します。

Chromeを開き、[Chrome リモートデスクトップ](https://remotedesktop.google.com/access)にアクセスします。  
Chromeでは接続先PCでログインしたものと同じGoogleアカウントにログインする必要があります。

接続できるようにしたPCをクリックします。

![接続できるPC一覧](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image9.png "接続できるPC一覧")

接続先PCの設定で入力したPINを入力します。

![PINの入力画面](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image10.png "PINの入力画面")

以上で接続先PCを操作できるようになります。

## オプションパネルについて
PCに接続後、画面右側の「>」マークをクリックすることでオプションパネルを開くことができます。  
オプションパネルでは接続を切断したり、接続の設定を変更したりできます。

![オプションパネル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image11.png "オプションパネル")

以下は一部のオプションについてです。

### 接続の切断
オプションパネル左上の「切断」をクリックすることでPCへの接続を終了できます。

![接続の切断ボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image12.png "接続の切断ボタン")

### 動画コーデック
動画コーデック(動画の圧縮方法)はVP8、VP9、AV1の3つから選べます。  
[参考にさせていただいたページ](https://www.winxdvd.com/video-convert/av1-vs-vp9.htm)によると、以下のようなイメージみたいです。

|項目|順序|
|----|----|
|ネットワーク帯域幅の使用量|VP8 > VP9 > AV1|
|ホストでのリソース使用量|AV1 > VP9 > VP8|

AV1はPCの処理能力は求められますが、圧縮率が高いため通信量が抑えられたり通信速度が速かったりします。

VP8はPCの処理能力は求められませんが、圧縮率が低いため通信量が増えたり通信速度が遅かったりします。

## 接続先PCの設定を削除する
[こちら](#接続先pcでchrome-remote-desktopの設定をする)で設定した接続先PCの設定を削除するには、[Chrome リモートデスクトップ](https://remotedesktop.google.com/access)にアクセスしてごみ箱マークをクリックします。

![リモート接続を無効にするボタン](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.title_eng }}/image13.png "リモート接続を無効にするボタン")

おそらく接続先PCからでも接続元PCからでも操作できると思います。