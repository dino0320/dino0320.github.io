---
layout: my-post
title: "LaravelのService Containerについて"
date: 2024-04-16 00:00:00 +0000
categories: web-application-framework laravel
title_eng: about-laravel-service-container
---

LaravelのService Containerについて調査しました。

## 参考ページ
- [Service Container](https://laravel.com/docs/11.x/container)
- [【Laravel】サービスコンテナとは？２つの強力な武器を持ったインスタンス化マシーン。簡単に解説。](https://qiita.com/minato-naka/items/afa4b930a2afac23261b)

## 環境
- Laravel 11

## もくじ
- [Service Containerとは](#service-containerとは)
- [依存関係の解決方法を定義する](#依存関係の解決方法を定義する)
- [Service Providerで依存関係の解決方法を定義する](#service-providerで依存関係の解決方法を定義する)

## Service Containerとは
[公式ページ](https://laravel.com/docs/11.x/container)によると、Service ContainerはPHPのクラスの依存関係を管理し、コンストラクタやセッターを通して依存関係を注入するツールとのことです。

上記の説明だけだとよく分からなかったのですが、[こちらのページ](https://qiita.com/minato-naka/items/afa4b930a2afac23261b)を参考にさせていただくと、あるクラスのインスタンスを作成したいときに、依存関係を意識しないで作成できるツールのようです。

例えば以下の2つのクラスが定義されているとします。
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
通常ClassAのインスタンスを作成したいとき、以下のようにClassBのインスタンスも作成する必要があります。
```php
$class_b = new ClassB();
$class_a = new ClassA($class_b);
```
LaravelのService Containerを使うと、以下のようにClassBのインスタンスの作成を意識せずにClassAのインスタンスを作成することができます。このとき内部ではClassBのインスタンスも作成されます。
```php
$class_a = App::make(ClassA::class);
```
上記では `make` 関数でインスタンスを作成していますが、通常はControllerなどの引数に指定すると、自動でインスタンスを作成してくれます。  
例えば以下のようにTestControllerのコンストラクタの引数にClassAを指定すると、TestControllerのインスタンス作成時に自動でClassAのインスタンスを作成してくれます。 
```php
class TestController extends Controller
{
    public function __construct(ClassA $class_a)
    {
    }
}
```

## 依存関係の解決方法を定義する
クラスに依存関係が無かったり、他のクラスにのみ依存する場合、Service Containerは自動で依存関係を解決します。([上記の例](#service-containerとは)など)  
しかし、依存関係の解決方法を自分で定義することもできます。  
例えば以下のクラスが定義されているとします。
```php
class ClassA
{
    public function __construct(int $arg)
    {
    }
}
```
コンストラクタにintの引数があるため、Service Containerは自動で依存関係を解決できません。  
そこで必ず `$arg` に1を入れてClassAのインスタンスを作成するように、解決方法を以下のように定義します。
```php
App::bind(ClassA::class, function () {
    return new ClassA(1);
});
```
これによって `$arg` に1が入ったClassAのインスタンスが自動で作成されるようになります。  

わかりやすいようにClassAのコンストラクタにログを入れ、`make` 関数でインスタンスを作成します。
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
実行すると、以下のようなログが出力されます。
```
arg = 1
```

## Service Providerで依存関係の解決方法を定義する
[公式ページ](https://laravel.com/docs/11.x/providers)によると、依存関係の解決方法はService Providerの `register` 関数で定義すると良いみたいです。  
Service ProviderはLaravelアプリケーションの様々なもの(Service Containerの依存関係の解決方法やevent listenerなど)を登録するところです。

試しにService Providerに依存関係の解決方法を定義してみます。  

Service Providerを作成します。  
以下のコマンドを実行して、TestServiceProviderを作成します。
```
php artisan make:provider TestServiceProvider
```
ClassAとClassBを作成し、TestServiceProviderに依存関係の解決方法を定義します。

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
        // 依存関係の解決方法の定義を追加
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

// ClassAを追加
class ClassA
{
    public function __construct(ClassB $class_b)
    {
        Log::info('ClassA instance is created.');
    }
}

// ClassBを追加
class ClassB
{
    public function __construct(int $arg)
    {
        Log::info("ClassB instance is created. arg = {$arg}");
    }
}
```

ClassAのインスタンスを作成するため、Controllerを作成し、コンストラクタの引数にClassAを指定します。  
以下のコマンドを実行して、TestControllerを作成します。
```
php artisan make:controller TestController
```
TestControllerにコンストラクタとViewを返す関数を作成します。

`TestController` :
```php
<?php

namespace App\Http\Controllers;

use App\Providers\ClassA;
use Illuminate\Contracts\View\View;

class TestController extends Controller
{
    // コンストラクタを追加。引数にClassAを指定。
    public function __construct(ClassA $class_a)
    {
    }

    // 適当なViewを返す関数を作成
    public function show(): View
    {
        return view('welcome');
    }
}
```
TestControllerのインスタンスを作成するため、適当なRouteを定義します。  
`web.php` などに以下を定義します。
```php
Route::get('/', [TestController::class, 'show']);
```
ブラウザなどでルートにアクセスすると、以下のようなログが出力されます。
```
ClassB instance is created. arg = 1
ClassA instance is created.
```
Service ProviderでService Containerの依存関係の解決方法を定義しました。