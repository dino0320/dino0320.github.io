---
layout: my-post
title: "Laravelのモデル作成コマンドをカスタマイズする"
date: 2025-03-21 00:00:00 +0000
categories: web-application-framework laravel
page_name: customizing-laravel-make-model-command
lang: ja
---

Laravelのモデル作成コマンド `php artisan make:model` の挙動をカスタマイズします。  
今回は `--migration` オプションをつけたときに生成されるMigrationファイルの場所を変更します。


## 参考ページ
- [Laravel 5.4 make:model コマンド改造入門](https://qiita.com/morisuke/items/93195ef8c031ca2d0976)

## 環境
- Laravel 11

## カスタマイズの流れ
1. [モデル作成コマンドのクラスを継承したクラスを作成する。](#1-モデル作成コマンドのクラスを継承したクラスを作成する)
2. [コマンドの挙動をカスタマイズする。](#2-コマンドの挙動をカスタマイズする)
3. [動作確認する。](#3-動作確認する)

## 1. モデル作成コマンドのクラスを継承したクラスを作成する
以下のArtisanコマンドでモデル作成コマンドのクラスを継承するクラスを作成します。 

```bash
php artisan make:command ModelMakeCommand
```

作成したクラスのプロパティとメソッドをすべて削除します。  

`app/Console/Commands/ModelMakeCommand.php` :
```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class ModelMakeCommand extends Command
{
}

```

モデル作成コマンドのクラスを継承するように継承先を変更します。  
継承先のクラスは今回はComposerでインストールしたパッケージの中の `vendor/laravel/framework/src/Illuminate/Foundation/Console/ModelMakeCommand.php` でした。  

`app/Console/Commands/ModelMakeCommand.php` :
```php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class ModelMakeCommand extends \Illuminate\Foundation\Console\ModelMakeCommand
{
}

```

## 2. コマンドの挙動をカスタマイズする
今回は `--migration` オプションをつけたときに生成されるMigrationファイルのパスをいじるので、継承先のクラスを見てそれっぽい関数をオーバーライドします。  
`createMigration` 関数をコピペします。

`app/Console/Commands/ModelMakeCommand.php` :
```php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class ModelMakeCommand extends \Illuminate\Foundation\Console\ModelMakeCommand
{
    /**
     * Create a migration file for the model.
     *
     * @return void
     */
    protected function createMigration()
    {
        $table = Str::snake(Str::pluralStudly(class_basename($this->argument('name'))));

        if ($this->option('pivot')) {
            $table = Str::singular($table);
        }

        $this->call('make:migration', [
            'name' => "create_{$table}_table",
            '--create' => $table,
        ]);
    }
}

```

`make:migration` を呼び出す処理に `--path` オプションを追加します。  
今回はMigrationファイルのディレクトリ構造をモデルの構造と同じにします。

`app/Console/Commands/ModelMakeCommand.php` :
```php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class ModelMakeCommand extends \Illuminate\Foundation\Console\ModelMakeCommand
{
    /**
     * Create a migration file for the model.
     *
     * @return void
     */
    protected function createMigration()
    {
        $table = Str::snake(Str::pluralStudly(class_basename($this->argument('name'))));

        if ($this->option('pivot')) {
            $table = Str::singular($table);
        }
        
        $dirName = dirname($this->getNameInput()); // 追加
        $this->call('make:migration', [
            'name' => "create_{$table}_table",
            '--create' => $table,
            '--path' => 'database/migrations' . ($dirName === '.' ? '' : "/{$dirName}"), // 追加
        ]);
    }
}

```

## 3. 動作確認する
カスタマイズした `php artisan make:model` を試してみます。

```bash
$ php artisan make:model test/Test --migration

   INFO  Model [app/Models/test/Test.php] created successfully.

   INFO  Migration [database/migrations/test/2025_03_21_075119_create_tests_table.php] created successfully.

```

モデルと同じディレクトリ構造のMigrationファイルを作成することができました。