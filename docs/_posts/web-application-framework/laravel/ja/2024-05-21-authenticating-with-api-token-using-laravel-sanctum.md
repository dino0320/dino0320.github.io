---
layout: my-post
title: "Laravel Sanctumを使ってAPIトークンで認証を行う"
date: 2024-05-21 00:00:00 +0000
categories: web-application-framework laravel
title_eng: authenticating-with-api-token-using-laravel-sanctum
lang: ja
---

Laravel Sanctumを使ったAPIトークンでの認証を試します。

## 参考ページ
- [Laravel Sanctum - Laravel 11.x](https://laravel.com/docs/11.x/sanctum)
- [【Laravel】SanctumでAPIトークン認証の使い方とSPA認証との比較 - Qiita](https://qiita.com/104dev/items/0787a81f7dda892ce86a)

## Laravel Sanctumとは
Laravel Sanctumは、SPA(single page applications)やモバイルアプリケーション、シンプルなトークンベースのAPIなどで使われる認証システムです。  
Cookieが使えなくても、APIトークンを使って認証ができるようになります。

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- Amazon Linux 2023(DockerコンテナのOS)
- PHP 8.2.15 (fpm-fcgi)
- Composer 2.7.6
- NGINX 1.24.0
- Laravel 11

## 前提
- Laravelプロジェクトを動かすDockerコンテナがある。  
LaravelプロジェクトをNGINXで動かす方法については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。
- Laravelプロジェクトを動かすDockerコンテナにComposerをインストールしている。  
Composerのインストールについては[こちら](/programming/php/installing-composer-on-linux)をご覧ください。

## もくじ
1. [Laravel Sanctumをインストールする](#1-laravel-sanctumをインストールする)
2. [APIトークンを発行できるようにする](#2-apiトークンを発行できるようにする)
3. [Routeを定義する](#3-routeを定義する)
4. [APIトークンでの認証を試す](#4-apiトークンでの認証を試す)

## 1. Laravel Sanctumをインストールする
Laravel Sanctumをインストールします。

以下のコマンドを実行します。

```
php artisan install:api
```

`personal_access_tokens` テーブルが追加されるので、データベースのマイグレーションを実行するか聞かれるかもしれません。  
実行する場合は `yes`、しない場合は `no` を入力し、Enterをクリックします。デフォルトは `yes` です。

```
One new database migration has been published. Would you like to run all pending database migrations? (yes/no) [yes]:
```

Composerをインストールしていないと以下のようなエラーが出ます。

```
sh: line 1: exec: composer: not found
```

## 2. APIトークンを発行できるようにする
認証に使用するEloquentモデルがAPIトークンを発行できるようにします。  

今回はLaravelプロジェクト作成時に生成されるUserモデルを使用するため、`app/Models/User.php` に `Laravel\Sanctum\HasApiTokens` Traitを追加します。

`User.php` :
```php
<?php

namespace App\Models;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens; // 追加

class User extends Authenticatable
{
    // HasApiTokensを追加
    use HasApiTokens, HasFactory, Notifiable;

～略～
```

`HasApiTokens` Traitを追加することで、Userモデルは `createToken` 関数を使ってAPIトークンを発行することができるようになります。

## 3. Routeを定義する
APIトークンを発行するRouteと、APIトークンを使った認証を行うRouteを定義します。

`php artisan install:api` を実行したときに作成された `routes/api.php` に以下のRouteを追加します。  
`/user` の定義はすでにあると思います。

`api.php` :
```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');

Route::post('/register', function (Request $request) {
    // ユーザー情報をデータベースに追加
    $user = User::create([
        'name' => $request->name,
        'email' => $request->email,
        'password' => $request->password
    ]);

    // APIトークンを発行
    $token = $user->createToken('TestToken');

    return ['token' => $token->plainTextToken];
});
```

`/user` はユーザーを認証してユーザー情報を返します。  
`auth:sanctum` Middlewareを追加することで、Cookieを使用しての認証またはAPIトークンを使用しての認証を行います。  
APIトークンを使用しての認証は認証用のCookieが存在しない場合に行われます。  

`/register` はユーザー登録を行いAPIトークンを発行して返します。  
`createToken` でAPIトークンを発行しています。

## 4. APIトークンでの認証を試す
[上記のRoute](#3-routeを定義する)を利用してAPIトークンでの認証を試します。  
今回はUbuntu上で `curl` コマンドを使ってLaravelプロジェクト(`localhost:8080`)にHTTPリクエストを送ります。

まず、ユーザー登録してAPIトークンを受け取ります。  
以下のコマンドを実行して `/register` にリクエストを送ります。

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"name":"Test", "email":"test@example.com", "password":"test"}' localhost:8080/api/register
```

HTTPリクエストボディには任意の `name`、`email`、`password` を入れます。

上記のコマンドの結果、以下のようにAPIトークンが返ってくると思います。

```bash
{"token":"1|RsrpmTepTJsUTfaosnVsE097WHaucLhZ5Vn7aPrV103b57d7"}
```

APIトークンはデータベースの `personal_access_tokens` テーブルに保存されます。  
`personal_access_tokens` テーブルを見てみると、以下のようにAPIトークンが保存されていることがわかります。  
APIトークンはSHA-256でハッシュ化されて保存されます。

```
+----+-----------------+--------------+-----------+------------------------------------------------------------------+-----------+---------------------+------------+---------------------+---------------------+
| id | tokenable_type  | tokenable_id | name      | token                                                            | abilities | last_used_at        | expires_at | created_at          | updated_at          |
+----+-----------------+--------------+-----------+------------------------------------------------------------------+-----------+---------------------+------------+---------------------+---------------------+
|  1 | App\Models\User |            1 | TestToken | 0cb255c4ed077c5b0a3709b64810bb95dc97ad078793eaaef7f0c8f01bc9ef92 | ["*"]     | 2024-05-21 09:18:26 | NULL       | 2024-05-21 09:17:53 | 2024-05-21 09:18:26 |
+----+-----------------+--------------+-----------+------------------------------------------------------------------+-----------+---------------------+------------+---------------------+---------------------+
```

次に、受け取ったAPIトークンを使用してユーザー認証してもらいます。  
以下のコマンドを実行して `/user` にリクエストを送ります。

```bash
$ curl -X GET -H "Content-Type: application/json" -H "Authorization: Bearer 1|RsrpmTepTJsUTfaosnVsE097WHaucLhZ5Vn7aPrV103b57d7" localhost:8080/api/user
```

HTTPリクエストヘッダーには `Authorization` ヘッダーを設定します。  
`Authorization` ヘッダーはBearerトークンとしてAPIトークンを持ちます。

上記のコマンドの結果、送ったAPIトークンによってユーザーが認証され、以下のようにユーザー情報が返ってくると思います。

```bash
{"id":1,"name":"Test","email":"test@example.com","email_verified_at":null,"created_at":"2024-05-21T09:17:53.000000Z","updated_at":"2024-05-21T09:17:53.000000Z"}
```