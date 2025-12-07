---
layout: my-post
title: "LaravelのMiddlewareについて"
date: 2024-05-02 00:00:00 +0000
categories: web-application-framework laravel
page_name: about-laravel-middleware
lang: ja
image: /assets/images/web-application-framework/laravel/about-laravel-middleware/image1.png
---

LaravelのMiddlewareについて調査しました。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 参考ページ
- [Middleware - Laravel 11.x](https://laravel.com/docs/11.x/middleware)

## 環境
- Laravel 11

## Middlewareとは
LaravelのMiddlewareは、Laravelアプリケーションで処理するHTTPリクエストを検査したりフィルタリングしたりするメカニズムです。  

例えばLaravelにはユーザーがログインしているか確認するMiddlewareがあります。  
このMiddlewareはアプリケーションがHTTPリクエストを本格的に処理する前に動き、ユーザーがログインしているかどうか確認します。  
ログインしていればHTTPリクエストをアプリケーションで処理することを許可し、ログインしていなければユーザーにログインページを表示します。

LaravelはいくつかデフォルトでMiddlewareを持っていますが、自分でMiddlewareを作成することもできます。

## もくじ
- [Middlewareを作成して試す](#middlewareを作成して試す)
- [メインの処理の後にMiddlewareの処理を行う](#メインの処理の後にmiddlewareの処理を行う)
- [Middlewareの登録](#middlewareの登録)
- [Middlewareグループ](#middlewareグループ)
- [Middlewareのエイリアス](#middlewareのエイリアス)
- [Middlewareのパラメータ](#middlewareのパラメータ)

## Middlewareを作成して試す
Middlewareを作成して動きを確かめます。

Middlewareを3つ作成します。

```
php artisan make:middleware Middleware1
php artisan make:middleware Middleware2
php artisan make:middleware Middleware3
```

Middlewareは `app/Http/Middleware` ディレクトリに作成されます。  
作成した各Middlewareにログの処理を追加します。

`Middleware1.php` :
```php
class Middleware1
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        Log::info('Midlleware1'); // 追加

        return $next($request);
    }
}
```

`$next` コールバックを呼び出すことで次の処理(他のMiddlewareなど)にHTTPリクエストを渡しています。  

`bootstrap/app.php` でMiddlewareを登録します。  
Middleware1 -> Middleware2 -> Middleware3 の順に処理するように登録します。  
`append` 関数はMiddlewareのリストの後ろにMiddlewareを追加する関数です。

`app.php` :
```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // 以下を追加
        $middleware->append(Middleware1::class);
        $middleware->append(Middleware2::class);
        $middleware->append(Middleware3::class);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

`/` にアクセスしたときにログを出力するようにします。  
`routes/web.php` に以下を追加します。

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
});
```

ブラウザ等で `/` にアクセスします。  
以下のようなログが出力されると思います。

```
local.INFO: Midlleware1
local.INFO: Midlleware2
local.INFO: Midlleware3
local.INFO: main
```

`/` にアクセスしたときの処理の前にMiddlewareの処理が動くことを確認できました。  
また、Middlewareがリストに追加した順に実行されていることがわかりました。

## メインの処理の後にMiddlewareの処理を行う
アプリケーションが本格的に処理を行った後にMiddlewareを実行したい場合、以下のように `$next` コールバックの呼び出しを一番最初に持ってきます。

`Middleware1.php` :
```php
class Middleware1
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request); // 一番最初に持ってくる

        // Middlewareの処理
        Log::info('Midlleware1');

        return $response;
    }
}
```

メインの処理の後にMiddleware1が実行されるかを確認します。  

`bootstrap/app.php` でMiddlewareを登録します。  

`app.php` :
```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // 以下を追加
        $middleware->append(Middleware1::class);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

`/` にアクセスしたときにログを出力するようにします。  
`routes/web.php` に以下を追加します。

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
});
```

ブラウザ等で `/` にアクセスすると、以下のようなログが出力されると思います。

```
local.INFO: main
local.INFO: Midlleware1
```

`/` にアクセスしたときの処理より後にMiddleware1の処理が実行されることを確認できました。

## Middlewareの登録
Middlewareの登録には2種類あります。  
グローバルMiddlewareとしての登録と、Routeに紐づける登録です。

### グローバルMiddlewareとしての登録
[上記の例](#middlewareを作成して試す)のように `bootstrap/app.php` でMiddlewareを登録すると、グローバルMiddlewareとして登録されます。  
グローバルMiddlewareとして登録すると、すべてのHTTPリクエストに対してMiddlewareの処理が行われます。

### Routeに紐づける登録
Routeに紐づけてMiddlewareを登録すると、そのRouteの処理を行うときだけMiddlewareの処理が行われます。

例えば `routes/web.php` で以下のように `/` へのアクセスの定義にMiddlewareの登録処理を追加します。  
`middleware` 関数を使うことでMiddlewareを登録できます。

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
})->middleware(Middleware2::class); // Middlewareの登録処理
```

Middleware1をグローバルMiddlewareとして登録しておきます。

ブラウザ等で `/` にアクセスします。  
以下のようなログが出力されると思います。  

```
local.INFO: Midlleware1
local.INFO: Midlleware2
local.INFO: main
```

グローバルMiddlewareのMiddleware1の他に、`/` の定義に紐づけたMiddleware2も実行されていることが分かります。

以下のような別のRouteの定義を追加し、そこにアクセスするとMiddleware2は実行されません。

`web.php` :
```php
Route::get('/test', function () {
    Log::info('main');
});
```

```
local.INFO: Midlleware1
local.INFO: main
```

複数のMiddlewareをRouteに紐づけるには、`middleware` 関数の引数に配列を指定します。

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
})->middleware([Middleware1::class, Middleware2::class]); // Middlewareの登録処理
```

## Middlewareグループ
Middlewareグループを作成することで複数のMiddlewareをまとめることができます。

`bootstrap/app.php` でMiddlewareグループ `middleware-group1` を登録します。  
Middleware1、Middleware2、Middleware3をまとめます。

`app.php` :
```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // 以下を追加
        $middleware->appendToGroup('middleware-group1', [
            Middleware1::class,
            Middleware2::class,
            Middleware3::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

`routes/web.php` で以下のように `/` へのアクセスの定義にMiddlewareグループの登録処理を追加します。  

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
})->middleware('middleware-group1'); // Middlewareグループの登録処理
```

ブラウザ等で `/` にアクセスします。  
以下のようなログが出力されると思います。

```
local.INFO: Midlleware1
local.INFO: Midlleware2
local.INFO: Midlleware3
local.INFO: main
```

Middlewareグループに含まれるMiddlewareが実行されていることを確認できました。

### デフォルトのMiddlewareグループ
Laravelはデフォルトで `web` と `api` というMiddlewareグループを持っています。  
これらのグループはそれぞれ `routes/web.php` と `routes/api.php` に自動的に適用されます。  
`web` と `api` に含まれるMiddlewareの一覧は[公式ページ](https://laravel.com/docs/11.x/middleware#laravels-default-middleware-groups)に載っております。

## Middlewareのエイリアス
Middlewareにエイリアスを割り当てることができます。  

`bootstrap/app.php` でMiddleware1のエイリアス `m1` を登録します。

`app.php` :
```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // 以下を追加
        $middleware->alias([
            'm1' => Middleware1::class
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

`routes/web.php` で以下のように `/` へのアクセスの定義にMiddlewareの登録処理を追加します。  
Middleware1のクラス名の代わりにエイリアスを指定します。

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
})->middleware('m1'); // Middlewareの登録処理
```

ブラウザ等で `/` にアクセスします。  
以下のようなログが出力されると思います。

```
local.INFO: Midlleware1
local.INFO: main
```

エイリアスに割り当てられているMiddlewareが実行されていることを確認できました。

Laravelがデフォルトで持っているMiddlewareにはもともとエイリアスが割り当てられています。  
[公式ページ](https://laravel.com/docs/11.x/middleware#middleware-aliases)にエイリアスの一覧が載っています。

## Middlewareのパラメータ
Middlewareはパラメータを受け取ることができます。  
Middlewareの `handle` 関数の引数 `next` の後ろに引数を追加することで、パラメータを受け取れます。

Middleware1の `handle` 関数の引数に `arg1` と `arg2` を追加します。

`Middleware1.php` :
```php
class Middleware1
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, string $arg1, string $arg2): Response
    {
        Log::info('Midlleware1');
        Log::info("{$arg1}, {$arg2}");

        return $next($request);
    }
}
```

`routes/web.php` で以下のように `/` へのアクセスの定義にMiddlewareの登録処理を追加します。  
`<Middlewareクラス名>:<引数1の値>,<引数2の値>` のように指定します。

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
})->middleware(Middleware1::class . ':arg1_value,arg2_value'); // Middlewareの登録処理
```

ブラウザ等で `/` にアクセスします。  
以下のようなログが出力されると思います。

```
local.INFO: Midlleware1
local.INFO: arg1_value, arg2_value
local.INFO: main
```

Middlewareがパラメータを受け取れたことを確認できました。