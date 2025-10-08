---
layout: my-post
title: "LaravelとReactプロジェクトのセットアップ"
date: 2025-10-08 00:00:00 +0000
categories: web-application-framework laravel
page_name: set-up-laravel-with-react-project
lang: ja
---

この記事では、Inertia.jsを使用してLaravelプロジェクトにReactを組み込む方法を説明します。

## 参考
- [Using React or Vue](https://laravel.com/docs/12.x/frontend#using-react-or-vue)
- [Server-side setup](https://inertiajs.com/server-side-setup)
- [Client-side setup](https://inertiajs.com/client-side-setup)
- [Pages](https://inertiajs.com/pages)
- [VPS環境でlaravel+reactで環境作成時のエラー備忘録「 ERR_CONNECTION_REFUSED」](https://qiita.com/aimkbiz/items/8e30dccd8d5906e83ef5)

## Inertiaとは？
Inertiaは、LaravelとReact(またはVue)を統合して使用できるようにする仕組みです。  
以下は[Laravelのドキュメント](https://laravel.com/docs/12.x/frontend#using-react-or-vue)から抜粋して翻訳しています。

> Inertiaは、LaravelアプリケーションとReact/Vueのモダンなフロントエンドの間を橋渡しします。Laravelのルーティング、コントローラ、データバインディング、認証などの仕組みを活かしつつ、Reactや Vueを使ったモダンなフロントエンドを、単一のリポジトリ内で構築できるようにします。

## 環境
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2ディストリビューション)
- Docker Engine 28.4.0
- Amazon Linux 2023(DockerコンテナのOS)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 12
- Node.js 20.14.0
- Vite 7.1.9

## 前提
- WSL上でDocker Composeを使ってLaravelプロジェクトが動作している。  
NGINXでLaravelを動かす方法については[こちらの記事](/web-application-framework/laravel/running-laravel-project-on-nginx)を参照してください。
- Viteがすでにインストールされている。  
インストール手順は[こちらの記事](/web-application-framework/laravel/installing-vite-with-laravel-on-wsl-and-docker)を参照してください。

## セットアップ手順
1. [Composerパッケージをインストール](#1-composerパッケージをインストール)
2. [ルートテンプレートを作成](#2-ルートテンプレートを作成)
3. [ミドルウェアを設定](#3-ミドルウェアを設定)
4. [NPMパッケージをインストール](#4-npmパッケージをインストール)
5. [vite.config.jsを編集](#5-viteconfigjsを編集)
6. [メインのJavaScriptファイルを作成](#6-メインのjavascriptファイルを作成)
7. [動作確認](#7-動作確認)

## 1. Composerパッケージをインストール
以下のComposerパッケージをインストールします。  
(Composerが未インストールの場合は[こちらの記事](/programming/php/installing-composer-on-linux)を参照してください。)

```bash
composer require inertiajs/inertia-laravel
```

## 2. ルートテンプレートを作成
[公式ドキュメント](https://inertiajs.com/server-side-setup)の通り、`resources/views` ディレクトリに `app.blade.php` を作成します。  
以下公式ドキュメントの抜粋の翻訳です。

> 最初にページを読み込む際に使用されるルートテンプレートを設定します。このテンプレートには、サイトのCSSやJavaScriptアセット、@inertia および @inertiaHead ディレクティブを含めます。

`app.blade.php`:

```php
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1"> 
    @viteReactRefresh
    @vite('resources/js/app.jsx')
    @inertiaHead
  </head>
  <body>
    @inertia
  </body>
</html>
```

`@viteReactRefresh` は `@vite` の前に記述してください。公式ドキュメントにも次のように記載されています(翻訳済み)：

> Reactアプリケーションの場合、開発中のFast Refreshを有効にするため、@viteReactRefresh を @vite の前に含めることを推奨します。

このテンプレートでは `@vite` により `resources/js/app.jsx` を読み込みます。  
このファイルは後で作成するInertiaアプリのエントリーポイントになります。

## 3. ミドルウェアを設定
Artisanコマンドを使って `HandleInertiaRequests` ミドルウェアを生成します。

```bash
php artisan inertia:middleware
```

その後、生成されたミドルウェアを `bootstrap/app.php` の `web` ミドルウェアグループに追加します。

`app.php`:

```php
<?php

use App\Http\Middleware\HandleInertiaRequests;
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware): void {
        $middleware->web(append: [
            HandleInertiaRequests::class, // 追加
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions): void {
        //
    })->create();

```

## 4. NPMパッケージをインストール
以下のNPMパッケージをインストールします。

```bash
npm install @inertiajs/react @vitejs/plugin-react react react-dom
```

## 5. vite.config.jsを編集
次のように `vite.config.js` を編集します。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import tailwindcss from '@tailwindcss/vite';
import react from '@vitejs/plugin-react'; // 追加

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.jsx'], // resources/js/app.jsx を指定
            refresh: true,
        }),
        tailwindcss(),
        react(), // 追加
    ],
    server: {
        host: true,
        hmr: {
            host: 'localhost',
        },
    },
});

```

`react()` は、Bladeテンプレート内の `@viteReactRefresh` を有効にするために必要です。

補足：  
`server.host`： サーバーが待ち受けるIPアドレスを指定します。  
デフォルトでは `localhost` のみですが、`0.0.0.0` または `true` に設定すると外部からもアクセス可能になります。  
これにより、Dockerホストからコンテナ内のサーバーにアクセスできるようになります。

`server.hmr.host`： HMR（Hot Module Replacement）の通信に使用するホストを指定します。  
Dockerコンテナ内で実行しているため `localhost` に設定します。  
HMRにより、CSSやJavaScriptの変更をブラウザをリロードせずに即時反映できます。

## 6. メインのJavaScriptファイルを作成
`resources/js` ディレクトリ内にInertiaアプリを起動するための `app.jsx` を作成します。

`app.jsx`:

```jsx
import { createInertiaApp } from '@inertiajs/react'
import { createRoot } from 'react-dom/client'

createInertiaApp({
  resolve: name => {
    const pages = import.meta.glob('./Pages/**/*.jsx', { eager: true })
    return pages[`./Pages/${name}.jsx`]
  },
  setup({ el, App, props }) {
    createRoot(el).render(<App {...props} />)
  },
})

```

## 7. 動作確認
動作確認用のInertiaページを作成します。

`resources/js/Pages` ディレクトリに `Welcome.jsx` を作成します。

`Welcome.jsx`:

```jsx
import { Head } from '@inertiajs/react'

export default function Welcome() {
  return (
    <div>
      <Head title="Welcome" />
      <h1>Welcome</h1>
      <p>Hello, welcome to your first Inertia app!</p>
    </div>
  )
}

```

次に、`routes/web.php` にルートを追加して `/` にアクセスしたときにこのページが表示されるようにします。

`web.php`:

```php
<?php

use Illuminate\Support\Facades\Route;
use Inertia\Inertia;

Route::get('/', function () {
    return Inertia::render('welcome');
});

```

### 開発サーバーを起動
CSSやJavaScriptを動的に反映させるために、開発サーバーを起動します。

プロジェクトのルートディレクトリで以下を実行します。

```bash
npm run dev
```

ブラウザで `/` にアクセスすると次のようなページが表示されます。

![Inertiaテストページ](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Inertiaテストページ")

### 本番用ビルド
本番環境用に最適化されたCSSとJavaScriptをビルドするには以下を実行します。

```bash
npm run build
```

ブラウザで `/` にアクセスすると同じ結果が表示されるはずです。