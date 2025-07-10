---
layout: my-post
title: "Customizing the Laravel make:model Command"
date: 2025-07-10 00:00:00 +0000
categories: web-application-framework laravel
page_name: customizing-laravel-make-model-command-en
lang: en
---

This post explains how to customize the behavior of Laravel's `php artisan make:model` command.  
In this case, we'll modify where the migration file is generated when using the `--migration` option.

## Reference
- [Laravel 5.4 make:model コマンド改造入門](https://qiita.com/morisuke/items/93195ef8c031ca2d0976)

## Environment
- Laravel 11

## Customization Steps
1. [Create a Class That Extends the Built-in Model Generation Command](#1-create-a-class-that-extends-the-built-in-model-generation-command)
2. [Customize the Behavior of the Command](#2-customize-the-behavior-of-the-command)
3. [Verify the Changes](#3-verify-the-changes)

## 1. Create a Class That Extends the Built-in Model Generation Command
Generate a new Artisan command that will extend the built-in model creation command: 

```bash
php artisan make:command ModelMakeCommand
```

Then remove all properties and methods from the generated class:

`app/Console/Commands/ModelMakeCommand.php` :
```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class ModelMakeCommand extends Command
{
}

```

Next, update the class to extend Laravel’s original `ModelMakeCommand`.  
In my environment, the original class is located at:
`vendor/laravel/framework/src/Illuminate/Foundation/Console/ModelMakeCommand.php` 

`app/Console/Commands/ModelMakeCommand.php` :
```php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class ModelMakeCommand extends \Illuminate\Foundation\Console\ModelMakeCommand
{
}

```

## 2. Customize the Behavior of the Command
We'll customize the location where the migration file is generated when using the `--migration` option.  
To do this, inspect the parent class and override the appropriate method.  
In this case, copy and override the `createMigration` method.

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

Now, add a `--path` option to customize the migration file’s location.  
We’ll use the same directory structure as the model.

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
        
        $dirName = dirname($this->getNameInput()); // Added
        $this->call('make:migration', [
            'name' => "create_{$table}_table",
            '--create' => $table,
            '--path' => 'database/migrations' . ($dirName === '.' ? '' : "/{$dirName}"), // Added
        ]);
    }
}

```

## 3. Verify the Changes
Try out the customized `php artisan make:model` command:

```bash
$ php artisan make:model test/Test --migration

   INFO  Model [app/Models/test/Test.php] created successfully.

   INFO  Migration [database/migrations/test/2025_03_21_075119_create_tests_table.php] created successfully.

```

The migration file was successfully created in a directory structure that mirrors the model's namespace.