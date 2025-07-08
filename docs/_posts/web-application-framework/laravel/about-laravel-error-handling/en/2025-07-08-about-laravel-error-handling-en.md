---
layout: my-post
title: "About Error Handling in Laravel"
date: 2025-07-08 00:00:00 +0000
categories: web-application-framework laravel
page_name: about-laravel-error-handling-en
lang: en
---

I investigated error handling in Laravel and tried out how to log and handle responses when exceptions occur.

## Reference
- [Error Handling - Laravel 11.x](https://laravel.com/docs/11.x/errors)

## Environment
- Laravel 11

## Table of Contents
- [Preparation](#preparation)
- [Logging Exceptions with report](#logging-exceptions-with-report)
- [Customizing Responses with render](#customizing-responses-with-render)

## Preparation
Before testing error handling, create a custom exception class that extends Laravel’s base `Exception` class.

Run the following Artisan command to generate the `TestException` class:

```
php artisan make:exception TestException
```

This will create the file: `app/Exceptions/TestException.php`.

## Logging Exceptions with report
By default, exceptions are logged according to the project’s logging configuration.  
This behavior can be customized in `bootstrap/app.php` or in individual `Exception` subclasses.

### Defining in app.php
To define logging behavior in `bootstrap/app.php`, use the `report` method inside the closure passed to `withExceptions`.

Edit `bootstrap/app.php` as follows:

```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        ...
    )
    ->withMiddleware(function (Middleware $middleware) {
        ...
    })
    ->withExceptions(function (Exceptions $exceptions) {
        // Add the following
        $exceptions->report(function (TestException $e) {
            // Log 'test'
            Log::error('test');
        });
    })->create();
```

This configuration logs the message 'test' whenever a `TestException` is thrown.  
You can specify the exception type using the closure’s type hint.

To disable the default logging behavior, either use `stop()` or return `false` from the closure:

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->report(function (TestException $e) {
        // Log 'test'
        Log::error('test');
    })->stop(); // Add stop()
})
 ```

 ```php
 ->withExceptions(function (Exceptions $exceptions) {
    $exceptions->report(function (TestException $e) {
        // Log 'test'
        Log::error('test');
        return false; // Add 'return false'
    });
})
```

### Defining in the Exception Class
You can also define logging behavior within the exception class itself by overriding the `report` method.

In `app/Exceptions/TestException.php`, add:

```php
...
class TestException extends Exception
{
    // Add the following
    public function report(): void
    {
        // Log 'test'
        Log::error('test');
    }
}
```

This logs 'test' whenever a `TestException` is thrown.

If you want to apply custom logic based on certain conditions, you can return `true` or `false` from the `report` method.  
Returning `true` uses the custom logic; returning `false` falls back to the default handler.

```php
public function report(Request $request): bool
{
    $user = $request->user();
    if ($user === null) {
        // Use default handling if user is not authenticated
        return false;
    }

    if ($user->id % 2 === 0) {
        // Log if user ID is even
        Log::error('test');
        return true;
    }

    // Use default handling if user ID is odd
    return false;
}
```

## Customizing Responses with render
By default, Laravel inspects the HTTP request’s `Accept` header and automatically returns either an HTML or JSON response when an exception occurs.  
You can customize this behavior in `bootstrap/app.php` or in individual exception classes.

### Defining in app.php
Edit `bootstrap/app.php` and use the `render` method inside the closure passed to `withExceptions`.

`app.php` :
```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        ...
    )
    ->withMiddleware(function (Middleware $middleware) {
        ...
    })
    ->withExceptions(function (Exceptions $exceptions) {
        // Add the following
        $exceptions->render(function (TestException $e) {
            return response()->json(['error_message' => $e->getMessage()]);
        });
    })->create();
```

This returns a JSON response with the error message when a `TestException` is thrown.

Make sure the `render` method returns an instance of `Illuminate\Http\Response` class or a subclass of it, typically created using the `response` helper.  
If the closure returns nothing, Laravel will use its default response handling.

### Defining in the Exception Class
You can override the `render` method in a custom exception class to define your own response logic.

In `app/Exceptions/TestException.php`, add:

```php
...
class TestException extends Exception
{
    // Add the following
    public function render(): JsonResponse
    {
        return response()->json(['error_message' => $this->message]);
    }
}
```

This returns a JSON response containing the exception message whenever the exception is thrown.

You can also use conditional logic in `render` to switch between custom and default responses by returning `false`:

```php
public function report(Request $request): bool
{
    $user = $request->user();
    if ($user === null) {
        // Use default response
        return false;
    }

    if ($user->id % 2 === 0) {
        // Custom response
        return response()->json(['error_message' => $this->message]);
    }

    // Use default response for odd IDs
    return false;
}
```