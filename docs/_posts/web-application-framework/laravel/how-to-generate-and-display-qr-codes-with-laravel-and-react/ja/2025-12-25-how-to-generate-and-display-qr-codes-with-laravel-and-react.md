---
layout: my-post
title: "Laravel + ReactでQRコードを生成・表示する方法【qrcode.react】"
date: 2025-12-25 00:00:00 +0000
categories: web-application-framework laravel
page_name: how-to-generate-and-display-qr-codes-with-laravel-and-react
lang: ja
image: /assets/images/web-application-framework/laravel/how-to-generate-and-display-qr-codes-with-laravel-and-react/image1.png
---

`qrcode.react` を使ってLaravel + ReactでQRコードを生成・表示してみます。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 参考
- [qrcode.react](https://github.com/zpao/qrcode.react)

## 前提
- React環境でセットアップ済みのLaravelプロジェクトがあること。  
詳しくは[こちらの記事](/web-application-framework/laravel/set-up-laravel-with-react-project)を参照してください。

## 環境
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2 distribution)
- Docker Engine 28.4.0
- Amazon Linux 2023(OS of the Docker container)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 12
- Node.js 20.14.0
- Vite 7.1.9
- React 19.2.0
- @inertiajs/react 2.2.6

## 手順
1. [qrcode.react をインストールする](#1-qrcodereact-をインストールする)
2. [サーバーサイドを実装する](#2-サーバーサイドを実装する)
3. [クライアントサイドを実装する](#3-クライアントサイドを実装する)
4. [QRコードを確認する](#4-qrコードを確認する)

## 1. qrcode.react をインストールする
以下のコマンドを実行して `qrcode.react` をインストールします。

```bash
npm install qrcode.react
```

`package.json` に `qrcode.react` が含まれていればインストール成功です。

`package.json`:
```json
"qrcode.react": "^4.2.0",
```

## 2. サーバーサイドを実装する
次のようにルーティングを追加します。

```php
Route::get('/accessed-by-qrcode', function () {
    return Inertia::render('AccessedByQRCode');
})->name('accessed-by-qrcode');

Route::get('/issue-qrcode', function () {
    return Inertia::render('IssueQRCode', [
        'url' => URL::temporarySignedRoute('accessed-by-qrcode', now()->addMinutes(10)),
    ]);
})
```

|ルート|説明|
|-----|-----------|
|/accessed-by-qrcode|QRコードからアクセスするルート|
|/issue-qrcode|QRコードを生成するルート。QRコードに変換するURLを渡します。|

`URL::temporarySignedRoute` はURLの改ざんを防ぎ、指定した時間を過ぎると無効になるURLを生成します。  
この例ではURLは10分間有効です。  
このメソッドの第一引数にルート名が必要なので、`/accessed-by-qrcode` にルート名を設定しています。

## 3. クライアントサイドを実装する
次のInertiaページを追加します。

`AccessedByQRCode.tsx`:
```tsx
export default function AccessedByQRCode() {
  return (
    <div>
      <p>QRコードからアクセスされました</p>
    </div>
  )
}
```

`IssueQRCode.tsx`:
```tsx
import {QRCodeSVG} from 'qrcode.react';

export default function IssueQRCode({ url }: { url: string }) {
  return (
    <div>
      <QRCodeSVG value={url} />
    </div>
  )
}
```

`AccessedByQRCode.tsx` はQRコードからアクセスできたかどうか確認するためのページです。  
`IssueQRCode.tsx` はQRコードを生成するためのページです。

`QRCodeSVG` はQRコードをSVGとして描画します。  
Canvasとして描画したい場合は `QRCodeCanvas` を使用してください。

以下は `QRCodeSVG` の主なオプションです。

|オプション|説明|
|------|-----------|
|value|QRコードにエンコードする値。|
|size|QRコードのサイズ（ピクセル）。デフォルト値は `128`。|
|level|エラー訂正レベル。レベルが高いほどQRコードの一部が欠けても読み取りやすくなりますが、QRコードは複雑になります。指定可能な値は `L` = 低（約7%）、`M` = 中（約15%）、`Q` = 高（約25%）、`H` = 最高（約30%）です。デフォルト値は `L` です。|

## 4. QRコードを確認する
ブラウザで `/issue-qrcode` にアクセスしてください。  
QRコードが表示されるはずです。

![生成されたQRコード](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "生成されたQRコード")

QRコードを読み取ると `/accessed-by-qrcode` にリダイレクトされます。