---
layout: my-post
title: "LaravelプロジェクトからMySQLを操作する"
date: 2024-04-09 00:00:00 +0000
categories: web-application-framework laravel
title_eng: controlling-mysql-from-laravel-project
---

LaravelプロジェクトからMySQLを操作するための設定を行います。  
Dockerを使用して環境を構築しています。

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- Laravel 11

## 前提
- Laravelプロジェクトを動かすDockerコンテナがある。(Docker Compose使用)  
LaravelプロジェクトをNGINXで動かす方法については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。

## 設定の流れ
1. [MySQLのDockerコンテナを作成する。](#1-mysqlのdockerコンテナを作成する)
2. [Laravelの設定を変更する。](#2-laravelの設定を変更する)
3. [設定の確認をする。](#3-設定の確認をする)

## 1. MySQLのDockerコンテナを作成する
MySQLのDockerコンテナを作成します。  
`docker-compose.yml` にMySQLコンテナの設定を追加します。

`docker-compose.yml` :
```yml
services:
～略～

  # MySQLコンテナの設定を追加
  db:    # サービス名
    image: mysql:8.0.34
    environment:
      MYSQL_ROOT_PASSWORD: root    # rootユーザーのパスワード
      MYSQL_DATABASE: database    # デフォルトのデータベース名
```
今回は今後AWSにデプロイすることを考えて、Amazon Auroraに合わせてMySQLサーバーのバージョンを `8.0.34` にしています。

## 2. Laravelの設定を変更する
MySQLに接続するようにLaravelの設定を変更します。  
`.env` のデータベースに関する環境変数を以下のように変更します。

`.env` :
```
DB_CONNECTION=mysql
DB_HOST=db    # MySQLのDockerコンテナのサービス名を指定
DB_PORT=3306
DB_DATABASE=database    # デフォルトのデータベース名を指定
DB_USERNAME=root    # rootユーザー名を指定
DB_PASSWORD=root    # rootユーザーのパスワードを指定
```

## 3. 設定の確認をする
MySQLを操作できるか確認します。  
新しいテーブルを作成し、Laravelプロジェクトからインサートします。

Dockerコンテナを起動します。  
`docker-compose.yml` のあるディレクトリに移動し、以下を実行します。
```bash
$ docker compose up -d --build
```
Laravelプロジェクトを動かすコンテナに接続します。  
`<サービス名>` はLaravelプロジェクトを動かすコンテナのサービス名です。
```bash
$ docker compose exec <サービス名> bash
```
コンテナ内でLaravelプロジェクトのルートディレクトリに移動します。  
```
# cd <Laravelプロジェクトのルートディレクトリ>
```
`tests` テーブルのModelとMigrationを作成します。
```
# php artisan make:model Test -m
```
`tests`テーブルに `name` カラムを追加します。  
`<Laravelプロジェクトのルートディレクトリ>/database/migrations/` ディレクトリにある `tests` テーブルのMigrationファイルのスキーマ定義を以下のように変更します。

`～_create_tests_table.php` :
```php
Schema::create('tests', function (Blueprint $table) {
    $table->id();
    $table->string('name'); // 追加
    $table->timestamps();
});
```
`<Laravelプロジェクトのルートディレクトリ>/app/Models/Test.php` に以下のプロパティを追加します。  
`create` 関数で割り当て可能なカラムを定義するためです。
```php
protected $fillable = [
    'name',
];
```
MySQLに `tests` テーブルを作成するために以下を実行します。
```
# php artisan migrate
```
アクセスするPHPファイル( `index.php` あたり)に以下を追加します。  
```php
Test::create([
    'name' => 'test',
]);
```
上記のPHPファイルにブラウザ等でアクセスします。

MySQLの中を確認します。  
Laravelプロジェクトを動かすコンテナから抜け、MySQLのコンテナに接続します。
```
# exit    # Laravelプロジェクトを動かすコンテナから抜ける
$ docker compose exec db bash    # MySQLコンテナに接続
```
MySQLに接続します。パスワードには `root` を入力します。
```
# mysql -u root -p
```
データベースを選択します。
```
mysql> use database
```
`tests` テーブルを確認します。
```
mysql> select * from tests;
+----+------+---------------------+---------------------+
| id | name | created_at          | updated_at          |
+----+------+---------------------+---------------------+
|  1 | test | 2024-04-09 08:39:10 | 2024-04-09 08:39:10 |
+----+------+---------------------+---------------------+
```
`id` カラムと `created_at` カラム、`updated_at` カラムは自動で値が割り当てられます。

Laravelプロジェクトからインサートできたことを確認できました。