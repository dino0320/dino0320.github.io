---
layout: my-post
title: "PHPUnitでテスト用データベースを使う"
date: 2025-08-28 00:00:00 +0000
categories: web-application-framework laravel
page_name: use-test-database-in-phpunit
lang: ja
---

PHPUnitを実行するときに専用のテスト用データベースを使用するようにします。  
今回はMySQLを利用します。

## 参考
- [Database Testing](https://laravel.com/docs/11.x/database-testing)

## 環境
- Amazon Linux 2023(DockerコンテナのOS)
- PHP 8.2.15 (fpm-fcgi)
- Laravel 11
- MySQL 8.0.34

## 前提条件
- MySQLがセットアップされている
- LaravelプロジェクトからMySQLに接続・操作できる  
詳しくは[こちらのページ](/web-application-framework/laravel/controlling-mysql-from-laravel-project)を参照してください。

## 設定手順
1. [テスト用データベースを作成する](#1-テスト用データベースを作成する)
2. [phpunit.xmlを編集する](#2-phpunitxmlを編集する)
3. [テストを作成する](#3-テストを作成する)
4. [RefreshDatabaseトレイトを追加する](#4-refreshdatabaseトレイトを追加する)

## 1. テスト用データベースを作成する
MySQLに接続します。

```bash
mysql -u <Your MySQL username> -p
```

MySQLユーザーのパスワードを入力し、テスト用の新しいデータベースを作成します。

```
CREATE DATABASE test_database;
```

## 2. phpunit.xmlを編集する
デフォルトで、`phpunit.xml` ファイルはLaravelプロジェクトのルートディレクトリにあります。  
テストで使用するデータベースを指定するために、`DB_DATABASE` 環境変数を追加します。  
その他のデータベース設定を変更したい場合は、対応する環境変数も追加してください。  
今回は `DB_CONNECTION` 環境変数も追加します。

次のように編集します。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true"
>
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
    </testsuites>
    <source>
        <include>
            <directory>app</directory>
        </include>
    </source>
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="APP_MAINTENANCE_DRIVER" value="file"/>
        <env name="BCRYPT_ROUNDS" value="4"/>
        <env name="CACHE_STORE" value="array"/>
        <!-- DB_CONNECTION と DB_DATABASE を追加 -->
        <env name="DB_CONNECTION" value="mysql"/>
        <env name="DB_DATABASE" value="test_database"/>
        <env name="MAIL_MAILER" value="array"/>
        <env name="PULSE_ENABLED" value="false"/>
        <env name="QUEUE_CONNECTION" value="sync"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="TELESCOPE_ENABLED" value="false"/>
    </php>
</phpunit>
```

## 3. テストを作成する
Laravelプロジェクトのルートディレクトリで、以下のコマンドを実行して新しいテストクラスを作成します。

```bash
php artisan make:test DatabaseTest
```

これにより、`tests/Feature` ディレクトリに `DatabaseTest.php` ファイルが生成されます。

## 4. RefreshDatabaseトレイトを追加する
`RefreshDatabase` トレイトは、各テストの前に内部的に `artisan migrate:fresh` コマンドを実行してデータベースをリセットします。

`tests/Feature/DatabaseTest.php` ファイルを編集します。

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class DatabaseTest extends TestCase
{
    use RefreshDatabase; // 追加

    /**
     * A basic functional test example.
     */
    public function test_basic_example(): void
    {
        $response = $this->get('/');
 
        // ...
    }
}

```

`artisan migrate` コマンドにオプションを追加したい場合は、次のように `migrateFreshUsing` メソッドをオーバーライドします。

```php
...

class DatabaseTest extends TestCase
{
    use RefreshDatabase {
        migrateFreshUsing as traitMigrateFreshUsing;
    }

    protected function migrateFreshUsing()
    {
        return array_merge([
            '--path' => 'database/migrations/user', // 任意のオプションを追加
            ], $this->traitMigrateFreshUsing());
    }

    ...
```