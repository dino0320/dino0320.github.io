---
layout: my-post
title: "WSL + Docker環境のLaravelでViteをインストールする方法"
date: 2024-06-08 00:00:00 +0000
categories: web-application-framework laravel
page_name: installing-vite-with-laravel-on-wsl-and-docker
lang: ja
image: /assets/images/web-application-framework/laravel/installing-vite-with-laravel-on-wsl-and-docker/image3.png
---

WSL + Dockerコンテナ上で動くLaravelプロジェクトでViteをインストールします。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "サムネイル")

## 参考ページ
- [Asset Bundling (Vite) - Laravel 11.x](https://laravel.com/docs/11.x/vite)
- [Vite](https://ja.vitejs.dev/)
- [DockerでLEMP環境を作成した際にLaravel Viteがエラーになった件 - Qiita](https://qiita.com/asaii-tone/items/88dcb4d40522dea415b0)
- [Laravel 9 + VITEの開発環境をdockerで実現する方法 - Qiita](https://qiita.com/hitotch/items/aa319c49d625c2a9b65e)
- [Docker + Viteな環境でホットリロードが効かない時の対策 - Zenn](https://zenn.dev/hctaw_srp/articles/1f7f67de03d710)

## Viteとは
Viteはフロントエンドビルドツールで、高速な開発サーバーや、CSSやJavascriptファイルをビルドするコマンドなどを提供します。  
開発環境では動的にCSSやJavascriptファイルを更新してくれる機能でスムーズに開発でき、本番環境ではCSSやJavascriptファイルをビルドする機能で最適化されたアセットを使用することができます。

## 環境
- Windows 10 64ビット
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- Amazon Linux 2023(DockerコンテナのOS)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 11

## 前提
- WSL上にLaravelプロジェクトを動かすDockerコンテナがある。(Docker Compose使用)  
LaravelプロジェクトをNGINXで動かす方法については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。

## インストールの流れ
1. [Node.jsをインストールする。](#1-nodejsをインストールする)
2. [Viteをインストールする。](#2-viteをインストールする)
3. [Viteを設定する。](#3-viteを設定する)
4. [Dockerコンテナを設定する。](#4-dockerコンテナを設定する)
5. [Viteの動作確認の準備をする。](#5-viteの動作確認の準備をする)
6. [Viteの動作を確認する。](#6-viteの動作を確認する)

## 1. Node.jsをインストールする
Node.jsのインストールについては[こちら](/programming/javascript/installing-node-on-docker)をご覧ください。

## 2. Viteをインストールする
Viteをインストールします。

Laravelプロジェクトを作成したときにデフォルトで `package.json` がプロジェクトのルートディレクトリに作成されると思います。  
その `package.json` にはViteがすでに含まれているため、それを使ってインストールします。  

Laravelプロジェクトのルートディレクトリで以下のコマンドを実行します。

```
npm install
```

これでViteがインストールされます。

ちなみに自分の環境では以下のような `package.json` が作成されていました。

`package.json` :
```json
{
    "private": true,
    "type": "module",
    "scripts": {
        "dev": "vite",
        "build": "vite build"
    },
    "devDependencies": {
        "axios": "^1.6.4",
        "laravel-vite-plugin": "^1.0",
        "vite": "^5.0"
    }
}
```

## 3. Viteを設定する
Dockerコンテナ上で動かすためのViteの設定をします。

Laravelプロジェクトのルートディレクトリにある `vite.config.js` を開き、`server` の設定を追加します。

`vite.config.js` :
```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
    // 以下を追加
    server: {
        host: true,
        hmr: {
            host: 'localhost',
        },
    },
});
```

`server.host` にはサーバーがリッスンすべきIPアドレスを指定します。  
デフォルトでは `localhost` しかリッスンしませんが、`0.0.0.0` または `true` にすることですべてのアドレスをリッスンするようになります。  
これによりDockerのホストからコンテナ内のサーバーにアクセスできるようになります。

`server.hmr.host` にはHMR(Hot Module Replacement)で通信するサーバーのIPアドレスを指定します。  
Dockerコンテナのアドレスなので、`localhost` を指定します。  
HMRは動的にCSSやJavascriptファイルを更新するための機能です。ブラウザをリロードすることなくそれらのファイルを更新することができます。

### WindowsファイルシステムにLaravelプロジェクトを置いている場合
WSLの `/mnt/c` ディレクトリ以下など、WindowsファイルシステムにLaravelプロジェクトを置いている場合、`watch` の設定も追加する必要があります。  
これはHTMLやJavascriptのファイルの変更を検知してもらうための設定です。

`vite.config.js` :
```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
    server: {
        host: true,
        hmr: {
            host: 'localhost',
        },
        // 以下を追加
        watch: {
            usePolling: true,
            interval: 1000,
        },
    },
});
```

`server.watch.usePolling` にはポーリングを行うかどうかを指定します。  
WSLでWindowsファイルシステムのファイルを監視するにはポーリングを行うしかないようなので、`true` にします。  
ポーリングはCPU使用率が高くなるため、`server.watch.interval` を `1000` (1秒)にすることで使用率をおさえます。

ポーリングでファイルを監視するのはパフォーマンスの観点で非推奨のようです。  
WSLのLaravelプロジェクトは `/home` ディレクトリ以下などのWindowsファイルシステム外に置くのがいいみたいです。

## 4. Dockerコンテナを設定する
Dockerコンテナのポートの設定を変更します。

LaravelプロジェクトのDockerコンテナの設定で、ViteのHMRのために5173番ポートを公開するようにします。  
`docker-compose.yml` を以下のように変更します。

`docker-compose.yml` :
```yml
services:
  web:    # Laravelプロジェクトが動くDockerコンテナ
    ～略～
    ports:
    - 8080:80    # HTTP通信のため80番ポートを公開する
    - 5173:5173    # ViteのHMRのため5173番ポートを公開する ←ここを追加
    ～略～
```

## 5. Viteの動作確認の準備をする
Viteの動作確認のための準備を行います。

`resources/js` ディレクトリにテスト用のJavascriptファイル(`test.js`)を作成します。

`test.js` :
```js
console.log('テスト');
```

`vite.config.js` を開き、エントリーポイントに `test.js` を追加します。

`vite.config.js` :
```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js',
                'resources/js/test.js', // 追加
            ],
            refresh: true,
        }),
    ],
    server: {
        host: true,
        hmr: {
            host: 'localhost',
        },
    },
});
```

`resources/views` ディレクトリにテスト用のBladeテンプレート(`test.blade.php`)を作成します。  
`@vite` で `resources/js/test.js` を読み込んでいます。

`test.blade.php` :
```php
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
  <title></title>
  <meta charset="utf-8" />
  @vite(['resources/js/test.js'])
</head>
<body>
</body>
</html>
```

以下のように `routes/web.php` などで `/` へのアクセスの定義を追加します。

`web.php` :
```php
Route::get('/', function () {
    return view('test');
});
```

## 6. Viteの動作を確認する
Viteの動作を確認します。  
開発サーバーを起動する方法と、ビルドする方法の2つを試します。

### 開発サーバーを起動する
開発サーバーを起動して動的にCSSやJavascriptファイルを更新するようにします。

Laravelプロジェクトのルートディレクトリで以下のコマンドを実行して開発サーバーを起動します。  

```
npm run dev
```

ブラウザなどで `/` にアクセスします。  
ブラウザのデベロッパーツールなどで、`resources/js/test.js` で実行した `console.log('テスト')` の結果が確認できると思います。

![test.jsの結果](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "test.jsの結果")

`resources/js/test.js` を以下のように変更すると、ブラウザをリロードしなくても動的に結果が更新されることが分かります。

`test.js` :
```js
console.log('てすと');
```

![更新後のtest.jsの結果](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "更新後のtest.jsの結果")

### ビルドする
CSSやJavascriptをビルドして最適化したアセットを使用するようにします。

Laravelプロジェクトのルートディレクトリで以下のコマンドを実行してCSSやJavascriptファイルをビルドします。

```
npm run build
```

`public/build/assets` ディレクトリ以下にビルドされたアセットが生成されます。

ブラウザなどで `/` にアクセスします。  
ブラウザのデベロッパーツールなどで、`resources/js/test.js` で実行した `console.log('テスト')` の結果が確認できると思います。

![test.jsの結果](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "test.jsの結果")

ビルドした場合、`resources/js/test.js` を変更してもブラウザが更新されることはありません。  
変更したい場合、変更後に再びビルドする必要があります。