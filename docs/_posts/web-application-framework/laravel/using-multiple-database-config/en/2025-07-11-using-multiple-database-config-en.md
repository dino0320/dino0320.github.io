---
layout: my-post
title: "Using Multiple Database Configurations in Laravel"
date: 2025-07-11 00:00:00 +0000
categories: web-application-framework laravel
page_name: using-multiple-database-config-en
lang: en
---

This guide explains how to use multiple database configurations in Laravel.  
In this example, we are using Eloquent models.

## Environment
- Laravel 11

## Changing the Database Connection for an Eloquent Model
Add the `$connection` property to the Eloquent model.  
By specifying a database configuration in the `$connection` property, that configuration will be used when connecting to the database.  
If this property is not set, the default database configuration will be used.

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
     * The database connection to be used
     *
     * @var string
     */
    protected $connection = 'sqlite'; // Configuration for SQLite
}
```

The above example assumes that you have two configurations prepared in `config/database.php`: one for SQLite (sqlite) and one for MySQL (mysql).

`config/database.php` :
```php
...

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

...
```

To use the MySQL configuration above, change the `$connection` property in the Eloquent model as shown below:

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
     * The database connection to be used
     *
     * @var string
     */
    protected $connection = 'mysql'; // Configuration for MySQL
}
```