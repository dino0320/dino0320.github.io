---
layout: my-post
title: "Laravelで複数のデータベース接続を利用する方法"
date: 2024-07-06 00:00:00 +0000
categories: web-application-framework laravel
page_name: using-multiple-database-config
lang: ja
image: /assets/images/web-application-framework/laravel/using-multiple-database-config/image1.png
---

Laravelで複数のデータベース設定を利用する方法です。  
今回はEloquentモデルを使っています。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 環境
- Laravel 11

## Eloquentモデルのデータベース接続先を変更する
Eloquentモデルに `$connection` プロパティを追加します。  
`$connection` プロパティにデータベース設定を指定すると、データベース接続時にその設定を使うようになります。  
このプロパティを追加しない場合、デフォルトのデータベース設定が使用されます。

`app/Models/Test.php` :
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Test extends Model
{
    use HasFactory;

    /**
     * データベース接続に使用される設定
     *
     * @var string
     */
    protected $connection = 'sqlite'; // SQLite用の設定
}
```

上記は以下の `config/database.php` のような、SQLite用の設定 `sqlite` とMySQL用の設定 `mysql` の2種類を用意した場合です。

`config/database.php` :
```php
～略～

    'connections' => [

        'sqlite' => [
            'driver' => 'sqlite',
            'url' => env('DB_URL_SQLITE'),
            'database' => env('DB_DATABASE_SQLITE', database_path('database.sqlite')),
            'prefix' => '',
            'foreign_key_constraints' => env('DB_FOREIGN_KEYS_SQLITE', true),
        ],

        'mysql' => [
            'driver' => 'mysql',
            'url' => env('DB_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'laravel'),
            'username' => env('DB_USERNAME', 'root'),
            'password' => env('DB_PASSWORD', ''),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => env('DB_CHARSET', 'utf8mb4'),
            'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
            'prefix' => '',
            'prefix_indexes' => true,
            'strict' => true,
            'engine' => null,
            'options' => extension_loaded('pdo_mysql') ? array_filter([
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
            ]) : [],
        ],

～略～
```

上記のMySQL用の設定を使うときは、Eloquentモデルの `$connection` プロパティを以下のように変更します。

`app/Models/Test.php` :
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Test extends Model
{
    use HasFactory;

    /**
     * データベース接続に使用される設定
     *
     * @var string
     */
    protected $connection = 'mysql'; // MySQL用の設定
}
```