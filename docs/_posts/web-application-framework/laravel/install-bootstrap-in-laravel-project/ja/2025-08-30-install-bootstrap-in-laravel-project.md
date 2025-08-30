---
layout: my-post
title: "LaravelプロジェクトにBootstrapをインストールする"
date: 2025-08-30 00:00:00 +0000
categories: web-application-framework laravel
page_name: install-bootstrap-in-laravel-project
lang: ja
---

この記事では、LaravelプロジェクトにBootstrapをインストールする方法を解説します。

## 参考
- [Bootstrap & Vite](https://getbootstrap.com/docs/5.2/getting-started/vite/)

## 環境
- Amazon Linux 2023(DockerコンテナOS)
- Laravel 11
- Vite 5.4.19
- Bootstrap 5.3.8
- Sass 1.91.0

## 前提条件
- LaravelプロジェクトにすでにViteがインストールされていること  
詳しくは[この記事](/web-application-framework/laravel/installing-vite-with-laravel-on-wsl-and-docker)を参照してください。

## Installation Steps
1. [BootstrapとSassをインストールする](#1-bootstrapとsassをインストールする)
2. [vite.config.jsを編集する](#2-viteconfigjsを編集する)
3. [Bootstrapをインポートする](#3-bootstrapをインポートする)
4. [Bootstrapを試す](#4-bootstrapを試す)

## 1. BootstrapとSassをインストールする
以下のコマンドを実行してBootstrapとSassをインストールします。

```bash
npm i --save bootstrap @popperjs/core
npm i --save-dev sass
```

## 2. vite.config.jsを編集する
`vite.config.js` ファイルにエイリアスを追加して、インポートをシンプルにします。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import * as path from 'path'; // 追加

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/scss/app.scss', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
    server: {
        host: true,
        hmr: {
            host: 'localhost',
        },
    },
    // 以下を追加
    resolve: {
        alias: {
            '~bootstrap': path.resolve(__dirname, 'node_modules/bootstrap'),
        }
    },
});
```

## 3. Bootstrapをインポートする
`bootstrap.scss` ファイルを作成し、Bootstrapをインポートします。

```css
@use '~bootstrap/scss/bootstrap';
```

`resources/scss/app.scss` ファイルを編集して `bootstrap.scss` をインポートします。

```css
@use './bootstrap';
```

## 4. Bootstrapを試す
`resources/views` 配下に `test.blade.php` というBladeテンプレートを作成します。  
`@vite` を使ってBootstrapを読み込み、`btn btn-primary` クラスを持つボタンを追加します。

`test.blade.php` :
```php
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
  <title></title>
  <meta charset="utf-8" />
  @vite(['resources/scss/bootstrap.scss'])
</head>
<body>
  <button class="btn btn-primary">Primary button</button>
</body>
</html>
```

`routes/web.php` に以下のルートを追加して、`/` にアクセスするとテストビューが表示されるようにします。

`web.php` :
```php
Route::get('/', function () {
    return view('test');
});
```

開発サーバーを起動してSCSSやJavaScriptファイルを動的にコンパイルします。  
Laravelプロジェクトのルートディレクトリで以下を実行してください。

```
npm run dev
```

ブラウザで `/` にアクセスします。  
Bootstrapが適用されたボタンが表示されるはずです。

![Primary button](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Primary button")