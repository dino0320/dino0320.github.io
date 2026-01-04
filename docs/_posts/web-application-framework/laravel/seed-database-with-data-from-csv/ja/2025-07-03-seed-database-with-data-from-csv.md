---
layout: my-post
title: "LaravelでCSVファイルからデータベースにデータをシードする方法"
date: 2025-07-03 00:00:00 +0000
categories: web-application-framework laravel
page_name: seed-database-with-data-from-csv
lang: ja
image: /assets/images/web-application-framework/laravel/seed-database-with-data-from-csv/image1.png
---

この記事では、LaravelでSeederを使ってCSVファイルからデータを読み込み、データベースに挿入する方法について解説します。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## Seederとは？
LaravelにおけるSeederとは、任意のデータをデータベースに挿入するための機能です。

## 参考
- [Database: Seeding](https://laravel.com/docs/11.x/seeding)
- [Laravel Seeder from CSV File Example](https://raviyatechnical.medium.com/laravel-advance-laravel-seeder-from-csv-file-example-cfb099e3656)

## 環境
- Laravel 11
- SQLite 3

## 作業の流れ
1. [テーブルを作成する](#1-テーブルを作成する)
2. [CSVファイルを作成する](#2-csvファイルを作成する)
3. [Seederを作成する](#3-seederを作成する)
4. [Seederを実行する](#4-seederを実行する)

## 1. テーブルを作成する
`tests` テーブルを作成します。今回はSQLiteを使用しています。  
以下のArtisanコマンドを実行して、`Test` モデルとマイグレーションを作成します。

```bash
php artisan make:model Test --migration
```

生成されたマイグレーションファイルを以下のように編集します。

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

マイグレーションを実行します。

```bash
php artisan migrate
```

## 2. CSVファイルを作成する
`tests` テーブル用のCSVファイルを `database/data` ディレクトリに `tests.csv` という名前で作成します。  
1列目が `id` カラム、2列目が `name` カラムになります。

```csv
1,test1
2,test2
3,test3
```

## 3. Seederを作成する
以下のArtisanコマンドでSeederクラスを作成します。  
ファイルは `database/seeders` に生成されます。

```bash
php artisan make:seeder TestSeeder
```

作成された `TestSeeder` を次のように編集します。  
このコードはCSVファイルからデータを読み込み、`tests` テーブルに挿入します。

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

次に、`DatabaseSeeder.php` を以下のように編集します。

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

## 4. Seederを実行する
以下のコマンドでSeederを実行します

```bash
php artisan db:seed
```

`tests` テーブルを確認すると、CSVファイルの内容が挿入されていることが確認できます。

```sql
SELECT * FROM `tests`;
1|test1
2|test2
3|test3
```