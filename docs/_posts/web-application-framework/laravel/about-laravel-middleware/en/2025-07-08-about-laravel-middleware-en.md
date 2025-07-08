---
layout: my-post
title: "Laravel Middleware"
date: 2025-07-08 00:00:00 +0000
categories: web-application-framework laravel
page_name: about-laravel-middleware-en
lang: en
---

I have researched Laravel Middleware.

## Reference
- [Middleware - Laravel 11.x](https://laravel.com/docs/11.x/middleware)

## Environment
- Laravel 11

## What is Middleware?
Laravel Middleware is a mechanism for inspecting and filtering HTTP requests handled by a Laravel application.

For example, Laravel includes middleware that checks if a user is authenticated.  
This middleware runs before the application processes the HTTP request and verifies whether the user is logged in.  
If authenticated, the request proceeds; otherwise, the user is redirected to the login page.

Laravel comes with several default middleware, but you can also create your own.

## Table of Contents
- [Creating and Testing Middleware](#creating-and-testing-middleware)
- [Executing Middleware After Main Processing](#executing-middleware-after-main-processing)
- [Registering Middleware](#registering-middleware)
- [Middleware Group](#middleware-group)
- [Middleware Aliases](#middleware-aliases)
- [Middleware Parameters](#middleware-parameters)

## Creating and Testing Middleware
Let's create and test middleware.

Create three middleware:

```
php artisan make:middleware Middleware1
php artisan make:middleware Middleware2
php artisan make:middleware Middleware3
```

These middleware are created in the `app/Http/Middleware` directory. Add logging to each middleware.

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
        Log::info('Midlleware1'); // Added

        return $next($request);
    }
}
```

By calling the `$next` callback, the HTTP request is passed to the next handler (other middleware, etc.).

Register the middleware in `bootstrap/app.php`.  
Register them in the order: Middleware1 → Middleware2 → Middleware3. The `append` function adds middleware to the end of the list.

`app.php` :
```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Add the following
        $middleware->append(Middleware1::class);
        $middleware->append(Middleware2::class);
        $middleware->append(Middleware3::class);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

Modify the logic to output logs when `/` is accessed.  
Add the following to `routes/web.php`:

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
});
```

Access `/` in a browser.  
The following logs should appear:

```
local.INFO: Midlleware1
local.INFO: Midlleware2
local.INFO: Midlleware3
local.INFO: main
```

This confirms that middleware processes run before the main request and in the order they were added.

## Executing Middleware After Main Processing
If you want middleware to run after the application processes the request, move the `$next` callback call to the beginning.

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
        $response = $next($request); // Move to the beginning

        // Middleware processing
        Log::info('Midlleware1');

        return $response;
    }
}
```

Register the middleware in `bootstrap/app.php` as before to verify that it runs after the main process.

`app.php` :
```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Add following
        $middleware->append(Middleware1::class);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

Modify the logic to output logs when `/` is accessed.  
Add the following to `routes/web.php`:

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
});
```

Access `/` in a browser.  
The following logs should appear:

```
local.INFO: main
local.INFO: Midlleware1
```

This confirms that Middleware1 runs after the main request.

## Registering Middleware
There are two ways to register middleware:
- Global Middleware: Registers middleware that runs for all HTTP requests.
- Route-Specific Middleware: Registers middleware for specific routes.

### Global Middleware
Registering middleware in `bootstrap/app.php` makes it global as in the [above example](#creating-and-testing-middleware).

### Route-Specific Middleware
Register middleware for specific routes in `routes/web.php`:

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
})->middleware(Middleware2::class); // Register middleware
```

the `middleware` method can register middleware.

Register Middleware1 as a global middleware.

Access `/` in a browser.  
The following logs should appear: 

```
local.INFO: Midlleware1
local.INFO: Midlleware2
local.INFO: main
```

You can see that not only Middleware1 runs, but also Middleware2 does.  

Define another route as the following.  
Middleware2 will not run when you access this route in a browser.

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

To apply multiple middleware to a route, pass an array to the `middleware` function.

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
})->middleware([Middleware1::class, Middleware2::class]); // Register middleware
```

## Middleware Group
You can group multiple middleware together.  
Register a middleware group in `bootstrap/app.php`:

```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Add the following
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

Apply the middleware group to a route in `routes/web.php`: 

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
})->middleware('middleware-group1'); // Register middleware group
```

Access `/` in a browser.  
The following logs should appear:

```
local.INFO: Midlleware1
local.INFO: Midlleware2
local.INFO: Midlleware3
local.INFO: main
```

Now you can see that all middlewares in the middleware group are executed.

### Default Middleware Groups
Laravel provides two default middleware groups: `web` and `api`.  
These groups are automatically applied to the routes defined in `routes/web.php` and `routes/api.php`, respectively.  
You can find the list of middleware included in the `web` and `api` groups on the [official documentation](https://laravel.com/docs/11.x/middleware#laravels-default-middleware-groups).

## Middleware Aliases
You can assign aliases to middleware.

In `bootstrap/app.php`, register an alias `m1` for Middleware1.

```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Add the following
        $middleware->alias([
            'm1' => Middleware1::class
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

Then in `routes/web.php`, apply the middleware to the `/` route using the alias instead of the class name.

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
})->middleware('m1'); // Register middleware
```

When you access `/` in the browser, the following logs should appear:

```
local.INFO: Midlleware1
local.INFO: main
```

This confirms that the middleware registered with the alias was executed properly.

Laravel’s default middleware already comes with predefined aliases.  
You can find the list on the [official documentation](https://laravel.com/docs/11.x/middleware#middleware-aliases).

## Middleware Parameters
Middleware can accept parameters.  
To pass parameters to a middleware, you add them after the `$next` argument in the `handle` method.

Add `arg1` and `arg2` to the `handle` method of Middleware1.

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

Then apply the middleware with parameters to the `/` route in `routes/web.php`.  
Use the format `<MiddlewareClass>:<param1>,<param2>`.

`web.php` :
```php
Route::get('/', function () {
    Log::info('main');
})->middleware(Middleware1::class . ':arg1_value,arg2_value'); // Register middleware
```

When you access `/` in the browser, the following logs should appear:

```
local.INFO: Midlleware1
local.INFO: arg1_value, arg2_value
local.INFO: main
```

This confirms that the middleware correctly received the parameters.