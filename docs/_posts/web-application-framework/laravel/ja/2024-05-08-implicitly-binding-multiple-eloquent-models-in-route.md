---
layout: my-post
title: "Laravelのルート定義で複数のEloquentモデルを暗黙的にバインドする"
date: 2024-05-08 00:00:00 +0000
categories: web-application-framework laravel
title_eng: implicitly-binding-multiple-eloquent-models-in-route
lang: ja
---

Laravelのルート定義で複数のEloquentモデルを暗黙的にバインドすることについて試しました。

## 参考ページ
- [Routing - Laravel 11.x](https://laravel.com/docs/11.x/routing#implicit-binding)
- [Eloquent: Relationships - Laravel 11.x](https://laravel.com/docs/11.x/eloquent-relationships#one-to-many)

## 前提
- LaravelプロジェクトのDBとしてMySQLを使用している。  
LaravelプロジェクトからMySQLを操作する方法については[こちら](/web-application-framework/laravel/controlling-mysql-from-laravel-project)をご覧ください。

## 環境
- Laravel 11
- mysql 8.0.34 for Linux

## もくじ
1. [DBテーブルとEloquentモデルを作成する](#1-dbテーブルとeloquentモデルを作成する)
2. [DBテーブルにデータを入れる](#2-dbテーブルにデータを入れる)
3. [Route定義を作成して動作確認する](#3-route定義を作成して動作確認する)

## 1. DBテーブルとEloquentモデルを作成する
`users` と `comments` の2つのDBテーブルとEloquentモデルを作成します。  

`comments` は `users` の子テーブルです。  
子テーブルの定義は親テーブルの `id` カラムを示すカラムを持っていることで、そのカラムの名前は `<親テーブル名からsを取ったもの>_id` になるみたいです。  
今回の場合、`comments` テーブルは `user_id` カラムを持つようにします。

以下のコマンドを実行してEloquentモデルとMigrationを作成します。  
`users` テーブルはLaravelプロジェクトを作成するときに作成されていると思うので、すでにある場合は作成しなくて大丈夫です。
```bash
php artisan make:model User -m    # users テーブルがすでにある場合は実行しない
php artisan make:model Comment -m
```

`database/migrations/～_create_comments_table.php` を開き、`up` 関数で `user_id` カラムを追加します。

`～_create_comments_table.php` :
```php
public function up(): void
{
    Schema::create('comments', function (Blueprint $table) {
        $table->id();
        $table->integer('user_id'); // 追加
        $table->timestamps();
    });
}
```

`app/Models/User.php` を開き、そのユーザーのコメントを取得するための関数を追加します。  
その関数名は子テーブルの名前(複数形)にする必要があるみたいです。

`User.php` :
```php
public function comments(): HasMany
{
    return $this->hasMany(Comment::class);
}
```

## 2. DBテーブルにデータを入れる
`users` と `comments` テーブルにいくつかデータを追加します。  

`comments` テーブルの `user_id` カラムには `users` テーブルに入れたデータの `id` カラムの値を入れるようにします。  
以下のように追加します。
```
mysql> select * from users;
+----+- 　　　 -+---------------------+---------------------+
| id |  　　　  | created_at          | updated_at          |
+----+- ～略～ -+---------------------+---------------------+
|  1 |  　　　  | 2024-05-08 07:23:50 | 2024-05-08 07:23:50 |
|  2 |  　　　  | 2024-05-08 07:24:06 | 2024-05-08 07:24:06 |
+----+- 　　　 -+---------------------+---------------------+
```
```
mysql> select * from comments;
+----+---------+---------------------+---------------------+
| id | user_id | created_at          | updated_at          |
+----+---------+---------------------+---------------------+
|  1 |       1 | 2024-05-08 07:24:34 | 2024-05-08 07:24:34 |
|  2 |       1 | 2024-05-08 07:24:42 | 2024-05-08 07:24:42 |
|  3 |       2 | 2024-05-08 07:24:49 | 2024-05-08 07:24:49 |
+----+---------+---------------------+---------------------+
```

## 3. Route定義を作成して動作確認する
複数のEloquentモデルを暗黙的にバインドするRoute定義を作成します。

`routes/web.php` などに以下を追加します。  

`web.php` :
```php
Route::get('/users/{user}/comments/{comment}', function (User $user, Comment $comment) {
    return $comment;
})->scopeBindings();
```
複数のEloquentモデルを暗黙的にバインドしてもらうために、`scopeBindings` 関数を呼び出す必要があります。  

定義したRouteにアクセスして暗黙的にバインドするかどうか確認します。
ブラウザなどで `http://localhost:8080/users/1/comments/1` にアクセスします。

`user_id` カラムが1で、`id` カラムが1のCommentモデルが取得できました。
```
{"id":1,"user_id":1,"created_at":"2024-05-08T07:24:34.000000Z","updated_at":"2024-05-08T07:24:34.000000Z"}
```

次に、`http://localhost:8080/users/1/comments/3` にアクセスします。  
404のページが表示されるはずです。これは `user_id` カラムが1で、`id` カラムが3のCommentモデルは存在しないためです。

### scopeBindings 関数について
Route定義で `scopeBindings` 関数を呼び出さないと親子テーブルの関係性を考慮せず、それぞれバインドしてしまいます。  

例えば[DBテーブルにデータを入れる](#2-dbテーブルにデータを入れる)で挿入したデータの場合、`scopeBindings` 関数を呼び出さずに `/users/1/comments/3` にアクセスすると、`id` カラムが1のUserモデルと `id` カラムが3のCommentモデルを取得してしまいます。  
`id` カラムが1のユーザーは `id` カラムが3のコメントを持っていないので、これだと関係性が考慮されていません。  
`scopeBindings` 関数を呼び出すと、それらのEloquentモデルは取得されず、404のページが表示されます。

しかし、[後述](#おまけ-id-カラム以外のカラムでeloquentモデルを取得する)のように `id` カラム以外のカラムを使ってデータを取得する場合、`scopeBindings` 関数を呼び出す必要はありません。

## (おまけ) id カラム以外のカラムでEloquentモデルをバインドする
Laravelはデフォルトでは `id` カラムでEloquentモデルをバインドしています。  
それ以外のカラムで暗黙的にバインドするには、以下の変更を行います。

`comments` テーブルに `comment_id` を追加し、それを使用してデータを取得するようにします。  
`comments` テーブルとデータを以下のように変更します。

`～_create_comments_table.php` :
```php
public function up(): void
{
    Schema::create('comments', function (Blueprint $table) {
        $table->id();
        $table->integer('user_id');
        $table->integer('comment_id'); // 追加
        $table->timestamps();
    });
}
```
```
mysql> select * from comments;
+----+---------+------------+---------------------+---------------------+
| id | user_id | comment_id | created_at          | updated_at          |
+----+---------+------------+---------------------+---------------------+
|  1 |       1 |          1 | 2024-05-08 07:24:34 | 2024-05-08 07:24:34 |
|  2 |       1 |          2 | 2024-05-08 07:24:42 | 2024-05-08 07:24:42 |
|  3 |       2 |          1 | 2024-05-08 07:24:49 | 2024-05-08 07:24:49 |
+----+---------+------------+---------------------+---------------------+
```

`routes/web.php` などに以下を追加します。  

`web.php` :
```php
Route::get('/users/{user}/comments/{comment:comment_id}', function (User $user, Comment $comment) {
    return $comment;
});
```
[scopeBindings 関数について](#scopebindings-関数について)で言及したように、`id` カラム以外のカラムで取得する場合は `scopeBindings` 関数を呼び出す必要はありません。

ブラウザなどで `http://localhost:8080/users/2/comments/1` にアクセスすると、`user_id` カラムが2で、`comment_id` カラムが1のCommentモデルが取得できました。
```
{"id":3,"user_id":2,"comment_id":1,"created_at":"2024-05-08T07:24:49.000000Z","updated_at":"2024-05-08T07:24:49.000000Z"}
```