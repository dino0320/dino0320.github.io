---
layout: my-post
title: "Use a Test Database in PHPUnit"
date: 2025-08-28 00:00:00 +0000
categories: web-application-framework laravel
page_name: use-test-database-in-phpunit-en
lang: en
---

Use a test database when running PHPUnit.  
In this example, we will use MySQL as the database.

## Reference
- [Database Testing](https://laravel.com/docs/11.x/database-testing)

## Environment
- Amazon Linux 2023(OS of the Docker container)
- PHP 8.2.15 (fpm-fcgi)
- Laravel 11
- MySQL 8.0.34

## Prerequisites
- MySQL is set up
- Laravel project can connect to and control MySQL  
For more details, refer to [this page](/web-application-framework/laravel/controlling-mysql-from-laravel-project-en).

## Setting Steps
1. [Create Test Database](#1-create-test-database)
2. [Edit phpunit.xml](#2-edit-phpunitxml)
3. [Create Test](#3-create-test)
4. [Add RefreshDatabase Trait](#4-add-refreshdatabase-trait)

## 1. Create Test Database
Connect to MySQL.

```bash
mysql -u <Your MySQL username> -p
```

Input your MySQK user's password, then create a new database for testing:

```
CREATE DATABASE test_database;
```

## 2. Edit phpunit.xml
By default, the `phpunit.xml` file exists in the Laravel project root directory.  
Add the `DB_DATABASE` environment variable to specify the database used for testing.  
If you need to change other database settings, add the corresponding environment variables as well.  
In this example, we will also add the `DB_CONNECTION` environment variable.

Edit it as follow:

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
        <!-- Add DB_CONNECTION and DB_DATABASE -->
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

## 3. Create Test
In the Laravel project root directory, run the following command to create a new test class:

```bash
php artisan make:test DatabaseTest
```

This generates a `DatabaseTest.php` file in the `tests/Feature` directory.

## 4. Add RefreshDatabase Trait
The `RefreshDatabase` trait resets the database before each test by internally running the `artisan migrate:fresh` command.

Edit the `tests/Feature/DatabaseTest.php` file:

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class DatabaseTest extends TestCase
{
    use RefreshDatabase; // Added

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

If you want to add options to the `artisan migrate` command, override the `migrateFreshUsing` method as follows:

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
            '--path' => 'database/migrations/user', // Add as many options as needed
            ], $this->traitMigrateFreshUsing());
    }

    ...
```