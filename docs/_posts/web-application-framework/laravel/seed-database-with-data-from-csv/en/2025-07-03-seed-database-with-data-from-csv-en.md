---
layout: my-post
title: "Seed Database with Data from CSV"
date: 2025-07-03 00:00:00 +0000
categories: web-application-framework laravel
page_name: seed-database-with-data-from-csv-en
lang: en
---

This article is how to seed a database with data from a CSV file in Laravel.

## What's Seeder?
Seeder in Laravel is an ability to insert any data into a database.

## Reference
- [Database: Seeding](https://laravel.com/docs/11.x/seeding)
- [Laravel Seeder from CSV File Example](https://raviyatechnical.medium.com/laravel-advance-laravel-seeder-from-csv-file-example-cfb099e3656)

## Environment
- Laravel 11
- SQLite 3

## Workflow
1. [Create Table](#1-create-table)
2. [Create CSV File](#2-create-csv-file)
3. [Create Seeder](#3-create-seeder)
4. [Run Seeder](#4-run-seeder)

## 1. Create Table
Create a tests table. In this time, I used SQLite.  
Execute the following Artisan to create a Test Model and Migration.

```bash
php artisan make:model Test --migration
```

Edit the created Migration file as follows.

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('tests', function (Blueprint $table) {
            $table->id('id');
            $table->string('name');
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('tests');
    }
};

```

Execute a migration command.

```bash
php artisan migrate
```

## 2. Create CSV File
Create a CSV file named `tests.csv` in the `database/data` directory for data in the tests table.   
The first column is for the `id` column, the second column is for the `name` column.

```csv
1,test1
2,test2
3,test3
```

## 3. Create Seeder
Using Artisan, create a new Seeder. It should be stored in the `database/seeders` directory.

```bash
php artisan make:seeder TestSeeder
```

Edit the created Seeder file as follows.  
It reads data from the CSV file and insert it into the tests table.

```php
<?php

namespace Database\Seeders;

use App\Models\Test;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;

abstract class TestSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        $values = [];
        if (($handle = fopen(database_path('data/tests.csv'), 'r')) !== false) {
            while (($data = fgetcsv($handle, null, ',')) !== false) {
                $values[] = [
                    'id' => $data[0],
                    'name' => $data[1],
                ];
            }

            fclose($handle);
        }

        Test::upsert($values, ['id']);
    }
}

```

Also edit the DatabaseSeeder as follows.

```php
<?php

namespace Database\Seeders;

use Database\Seeders\TestSeeder;
// use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
        $this->call([
            TestSeeder::class,
        ]);
    }
}

```

## 4. Run Seeder
Execute a seed command.

```bash
php artisan db:seed
```

Check the tests table. You can see data inserted from the CSV file.

```sql
SELECT * FROM `tests`;
1|test1
2|test2
3|test3
```