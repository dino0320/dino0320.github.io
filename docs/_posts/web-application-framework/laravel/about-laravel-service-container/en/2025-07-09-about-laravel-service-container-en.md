---
layout: my-post
title: "About Laravel's Service Container"
date: 2025-07-09 00:00:00 +0000
categories: web-application-framework laravel
page_name: about-laravel-service-container-en
lang: en
---

I researched Laravel's Service Container.

## References
- [Service Container - Laravel 11.x](https://laravel.com/docs/11.x/container)
- [【Laravel】サービスコンテナとは？２つの強力な武器を持ったインスタンス化マシーン。簡単に解説。 - Qiita](https://qiita.com/minato-naka/items/afa4b930a2afac23261b)

## Environment
- Laravel 11

## Table of Contents
- [What is a Service Container](#what-is-a-service-container)
- [Defining How to Resolve Dependencies](#defining-how-to-resolve-dependencies)
- [Defining Dependency Resolution in a Service Provider](#defining-dependency-resolution-in-a-service-provider)

## What is a Service Container
According to the [official documentation](https://laravel.com/docs/11.x/container), a Service Container is a tool that manages the dependencies of PHP classes and injects them through constructors or setters.

That explanation alone was a bit difficult to grasp, but referring to [this page](https://qiita.com/minato-naka/items/afa4b930a2afac23261b) helped clarify things. It seems like a tool that allows you to instantiate classes without worrying about their dependencies.

For example, suppose you have the following two classes:

```php
class ClassA
{
    public function __construct(ClassB $class_b)
    {
    }
}

class ClassB
{
    public function __construct()
    {
    }
}
```

Normally, to create an instance of ClassA, you would also need to create an instance of ClassB like this:

```php
$class_b = new ClassB();
$class_a = new ClassA($class_b);
```
However, using Laravel's Service Container, you can create an instance of ClassA without explicitly creating ClassB:

```php
$class_a = App::make(ClassA::class);
```

Here, the `make` function is used to create the instance.   
In practice, though, Laravel often resolves dependencies automatically when you specify them in a controller’s constructor.  
For example, if you specify ClassA in the constructor of TestController, Laravel will automatically create an instance of ClassA when it creates the controller:

```php
class TestController extends Controller
{
    public function __construct(ClassA $class_a)
    {
    }
}
```

## Defining How to Resolve Dependencies
If a class has no dependencies or only depends on other classes, the Service Container can resolve dependencies automatically (as in the [example above](#what-is-a-service-container)).  
However, you can also manually define how dependencies should be resolved.

Suppose you have the following class:

```php
class ClassA
{
    public function __construct(int $arg)
    {
    }
}
```

Since the constructor requires an int, the Service Container cannot resolve it automatically.  
So, let’s define how to resolve this dependency manually so that the `$arg` value is always 1:

```php
App::bind(ClassA::class, function () {
    return new ClassA(1);
});
```

This way, an instance of ClassA with `$arg` set to 1 will be created automatically.

To make it easier to see, let's add a log to the constructor of ClassA and then create the instance using `make`:

```php
class ClassA
{
    public function __construct(int $arg)
    {
        Log::info("arg = {$arg}");
    }
}
```

```php
$class_a = App::make(ClassA::class);
```

Upon execution, the following log will be output:

```
arg = 1
```

## Defining Dependency Resolution in a Service Provider
According to the [official documentation](https://laravel.com/docs/11.x/providers), it’s recommended to define dependency resolution inside the `register` method of a Service Provider.  
Service Providers are where you can register many aspects of a Laravel application, such as dependency resolutions and event listeners.

Let’s try defining a dependency resolution in a Service Provider.

First, create a Service Provider.  
Run the following command to create TestServiceProvider:

```
php artisan make:provider TestServiceProvider
```

Now, create ClassA and ClassB, and define the dependency resolution in TestServiceProvider.

`TestServiceProvider.php` :
```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Log;
use Illuminate\Support\ServiceProvider;

class TestServiceProvider extends ServiceProvider
{
    /**
     * Register services.
     */
    public function register(): void
    {
        // Define how to resolve ClassB
        $this->app->bind(ClassB::class, function () {
            return new ClassB(1);
        });
    }

    /**
     * Bootstrap services.
     */
    public function boot(): void
    {
        //
    }
}

// Add ClassA
class ClassA
{
    public function __construct(ClassB $class_b)
    {
        Log::info('ClassA instance is created.');
    }
}

// Add ClassB
class ClassB
{
    public function __construct(int $arg)
    {
        Log::info("ClassB instance is created. arg = {$arg}");
    }
}
```

To create an instance of ClassA, specify it as a parameter in a route closure, like in `web.php`:

```php
Route::get('/', function (ClassA $class_a) {});
```

When you access the root path in your browser, you’ll see logs like this:

```
ClassB instance is created. arg = 1
ClassA instance is created.
```

And that's how you define dependency resolution in the Service Container using a Service Provider.