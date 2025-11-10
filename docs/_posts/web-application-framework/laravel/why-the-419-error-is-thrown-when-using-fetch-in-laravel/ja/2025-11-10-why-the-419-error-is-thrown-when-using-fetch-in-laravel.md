---
layout: my-post
title: "なぜLaravelでfetchを使うと419エラーが発生するのか?"
date: 2025-11-10 00:00:00 +0000
categories: web-application-framework laravel
page_name: why-the-419-error-is-thrown-when-using-fetch-in-laravel
lang: ja
image: /assets/images/web-application-framework/laravel/why-the-419-error-is-thrown-when-using-fetch-in-laravel/image1.png
---

Laravelプロジェクトフロント側のTypeScriptのコード内で `fetch` メソッドを使ってサーバー側にリクエストを送ろうとしたところ、419エラーが発生しました。  
この記事では、その原因と解決方法をまとめました。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 参考
- [CSRF Protection](https://laravel.com/docs/12.x/csrf)
- [クロスサイトリクエストフォージェリ](https://ja.wikipedia.org/wiki/%E3%82%AF%E3%83%AD%E3%82%B9%E3%82%B5%E3%82%A4%E3%83%88%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%E3%83%95%E3%82%A9%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%AA)

## 前提
- Laravelプロジェクトを作成済み  
LaravelプロジェクトをNGINXで動かす方法については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。  
Reactを使用する場合は[こちら](/web-application-framework/laravel/set-up-laravel-with-react-project)をご覧ください。

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
- React 19.2.0

## もくじ
- [原因](#原因)
- [解決方法](#解決方法)

## 原因
前述のように、TypeScriptで `fetch` メソッドを使って以下のようにLaravelサーバーにHTTP POSTリクエストを送信したところ、419エラーが返ってきました。

```ts
fetch('/test', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ name: 'test' })
})
.then(data => {
  console.log(data)
})
```

これはLaravelがアプリケーションを**CSRF**攻撃から保護しているためです。  
`fetch` メソッドはこの保護のためのロジックを自動的に適用しません。  
その結果、Laravelはリクエストが正当かどうかを検証できず、419エラーを投げます。

### CSRFとは?
CSRF(クロスサイトリクエストフォージェリ)とは、Webアプリケーションが信頼しているユーザーから不正なコマンドが送信される攻撃のことです。  
詳細は[こちらのページ](https://ja.wikipedia.org/wiki/%E3%82%AF%E3%83%AD%E3%82%B9%E3%82%B5%E3%82%A4%E3%83%88%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%E3%83%95%E3%82%A9%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%AA)を参照してください。

例えばあなたのアプリケーションにログインしているユーザーが、攻撃者のウェブページにアクセスしそこからフォームを送信してしまうと、そのフォームを通じて悪意あるリクエスト(例: メールアドレスやパスワードの変更など)が送信されてしまう可能性があります。

このような攻撃を防ぐためにはリクエストが信頼できる送信元からのものかどうかをトークンの確認等で検証する必要があります。

## 解決方法
この問題を解決するには、`fetch` メソッドの代わりに `axios` ライブラリを使うのが簡単です。
上記のコードを以下のように書き換えてみます。

```ts
const response = await axios.post('test', {
  name: 'test',
});
console.log(response.data)
```

[公式ドキュメント](https://laravel.com/docs/12.x/csrf)にもある通り、  
Laravelはアプリケーションが管理する各ユーザーセッションごとにCSRFトークンを自動生成します。  
このトークンはユーザーのセッションに保存され、`Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` ミドルウェアによって検証されます。  
このミドルウェアはデフォルトで `web` ミドルウェアグループに含まれており、リクエスト内のトークンがセッションに保存されたトークンと一致しているかを確認します。

`axios` ライブラリはアプリケーションによって `XSRF-TOKEN` クッキーに保存されたトークンを自動的に `X-XSRF-TOKEN` ヘッダーに含めて送信します。  
そのため `axios` を使う場合はトークンを手動でヘッダーに設定する必要はありません。

しかし `fetch` メソッドはこの処理を自動で行いません。  
つまりLaravelサーバーに `fetch` でリクエストを送る場合は次のようにヘッダーにトークンを自分で設定する必要があります。

```ts
const token = decodeURIComponent(getCookie('XSRF-TOKEN'));
fetch('/test', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-XSRF-TOKEN': token
  },
  body: JSON.stringify({ name: 'test' })
})
.then(data => {
  console.log(data)
})

function getCookie(key: string) {
  const cookies = `; ${document.cookie}`
  const splitedCookies = cookies.split(`; ${key}=`)
  if (splitedCookies.length === 2) { // 最初の要素は空文字列のはず
    const cookie = splitedCookies.pop() ?? ''
    // 取得したいクッキーの後ろに他のクッキーが含まれている可能性があるためsplitして最初の要素を取得する
    return cookie.split(';').shift() ?? ''
  }

  return ''
}
```