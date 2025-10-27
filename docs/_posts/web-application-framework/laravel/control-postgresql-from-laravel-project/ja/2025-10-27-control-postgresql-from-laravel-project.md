---
layout: my-post
title: "LaravelプロジェクトからPostgreSQLを操作する"
date: 2025-10-27 00:00:00 +0000
categories: web-application-framework laravel
page_name: control-postgresql-from-laravel-project
lang: ja
image: /assets/images/web-application-framework/laravel/control-postgresql-from-laravel-project/image1.png
---

LaravelプロジェクトからPostgreSQLを操作するための設定を行います。  
Dockerを使用して環境を構築しています。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 環境
- Windows 11
- Ubuntu 24.04.3 LTS (WSLで起動している)
- Docker Engine 28.4.0
- PHP 8.2.29
- Laravel 12

## 前提
- Laravelプロジェクトを動かすDockerコンテナがある。(Docker Compose使用)  
LaravelプロジェクトをNGINXで動かす方法については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。

## 設定の流れ
1. [PostgreSQLのDockerコンテナを作成する](#1-postgresqlのdockerコンテナを作成する)
2. [php-pgsqlをインストールする](#2-php-pgsqlをインストールする)
3. [Laravelの設定を更新する](#3-laravelの設定を更新する)
4. [設定を確認する](#4-設定を確認する)

## 1. PostgreSQLのDockerコンテナを作成する
PostgreSQL用のDockerコンテナを作成します。  
`docker-compose.yml` ファイルに以下の設定を追加してください。

`docker-compose.yml` :

```yml
services:
...

  # PostgreSQLコンテナ設定の追加
  db:                                             # サービス名
    image: postgres:17.6
      - ./initdb.d:/docker-entrypoint-initdb.d    # 起動時に実行されるSQLファイルを含むディレクトリ
    environment:
      POSTGRES_DB: database                       # デフォルトデータベース名
      POSTGRES_PASSWORD: superuser                # スーパーユーザーパスワード
```

この構成では、将来的にAWSのAmazon RDSへデプロイすることを見据えて、PostgreSQLのバージョン17.6を使用しています。

次に、`docker-compose.yml` と同じディレクトリに `initdb.d/init.sql` ファイルを作成し、以下の内容を記述します。

`init.sql`:

```sql
CREATE USER "user" WITH PASSWORD 'password';
GRANT CREATE ON SCHEMA public TO "user";
```

このファイルは、`user` という名前のユーザーを作成し、パスワード `password` を設定します。  
このユーザーはLaravelプロジェクトから使用されます。  
さらに、このユーザーには `public` スキーマに対する `CREATE` 権限を付与します。  
マイグレーションを実行するためにはこの権限が必要です。

## 2. php-pgsqlをインストールする
PHPからPostgreSQLに接続するため、LaravelプロジェクトのDockerコンテナに `php-pgsql` モジュールをインストールします。  
Laravelコンテナの `Dockerfile` を次のように編集してください。

`Dockerfile`:

```Dockerfile
RUN yum -y install php8.2-pgsql
```

## 3. Laravelの設定を更新する
LaravelからPostgreSQLデータベースに接続できるように設定を変更します。  
`.env` ファイル内のデータベース関連の環境変数を以下のように編集してください。

`.env` :

```
DB_CONNECTION=pgsql
DB_HOST=db           # PostgreSQLコンテナのサービス名を指定
DB_PORT=5432
DB_DATABASE=database # デフォルトデータベース名
DB_USERNAME=user     # PostgreSQLユーザー
DB_PASSWORD=password # PostgreSQLユーザーのパスワード
```

## 4. 設定を確認する
LaravelからPostgreSQLに正しく接続できるか確認します。  
テーブルを作成し、Laravelプロジェクトからデータを挿入してみます。

まず、Dockerコンテナを起動します。  
`docker-compose.yml` のあるディレクトリで以下を実行してください。

```bash
$ docker compose up -d --build
```

次に、Laravelプロジェクトを実行しているコンテナに接続します。  
`<サービス名>` にはLaravelコンテナのサービス名を指定してください。

```bash
$ docker compose exec <サービス名> bash
```

コンテナ内で、Laravelプロジェクトのルートディレクトリに移動します。

```
# cd <Laravelプロジェクトのルートディレクトリ>
```

`tests` テーブル用のモデルとマイグレーションを作成します。

```
# php artisan make:model Test -m
```

次に、`tests` テーブルに `name` カラムを追加します。  
`database/migrations` ディレクトリ内のマイグレーションファイルを次のように編集してください。

`*_create_tests_table.php` :

```php
Schema::create('tests', function (Blueprint $table) {
    $table->id();
    $table->string('name'); // 追加
    $table->timestamps();
});
```

次に、`app/Models/Test.php` を更新し、マスアサインメントに使用するカラムを指定します。

```php
protected $fillable = [
    'name',
];
```

マイグレーションを実行して、PostgreSQLに `tests` テーブルを作成します。

```
# php artisan migrate
```

次に、ブラウザからアクセスできるPHPファイル（例: `index.php` など）に以下のコードを追加します。

```php
Test::create([
    'name' => 'test',
]);
```

ブラウザでこのファイルにアクセスしてコードを実行します。

次に、PostgreSQLデータベースの内容を確認します。  
Laravelコンテナから一度抜け、PostgreSQLコンテナに接続してください。

```
# exit                           # Laravelコンテナから退出
$ docker compose exec db bash    # PostgreSQLコンテナに接続
```

`user` のパスワード（`password`）を使用してPostgreSQLに接続します。

```
# psql -U user -d database
```

`tests` テーブルの内容を確認します。

```
database=> select * from tests;
 id | name | created_at          | updated_at          
----+------+---------------------+---------------------
  1 | test | 2024-04-09 08:39:10 | 2024-04-09 08:39:10 
```

`id`、`created_at`、`updated_at` カラムは自動的に値が設定されています。

これで、LaravelプロジェクトからPostgreSQLへのデータ挿入が正常に行われたことが確認できました。