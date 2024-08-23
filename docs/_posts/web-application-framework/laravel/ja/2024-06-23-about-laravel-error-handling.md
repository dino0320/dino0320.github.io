---
layout: my-post
title: "LaravelのError Handlingについて"
date: 2024-06-23 00:00:00 +0000
categories: web-application-framework laravel
page_name: about-laravel-error-handling
lang: ja
---

LaravelのError Handlingについて調査しました。  
例外発生時のログ記録処理やレスポンス処理などを試しました。

## 参考ページ
- [Error Handling - Laravel 11.x](https://laravel.com/docs/11.x/errors)

## 環境
- Laravel 11

## もくじ
- [事前準備](#事前準備)
- [例外発生時のログ記録処理(report)](#例外発生時のログ記録処理report)
- [例外発生時のレスポンス処理(render)](#例外発生時のレスポンス処理render)

## 事前準備
Error Handlingについて試す前にテスト用の `Exception` クラスの子クラスを作成します。  

以下のコマンドを実行して `TestException` クラスを作成します。

```
php artisan make:exception TestException
```

`app/Exceptions/TestException.php` が作成されます。

## 例外発生時のログ記録処理(report)
デフォルトでは、発生した例外はプロジェクトのログ設定に基づいて記録されます。  
この処理は `bootstrap/app.php` や `Exception` クラスの子クラスで好きなように定義できます。

### app.phpで定義する場合
`bootstrap/app.php` でログ記録処理を定義する場合、以下のように定義できます。

`bootstrap/app.php` を開き、`withExceptions` 関数の引数のクロージャ内で `report` 関数を呼び出します。  
ログ記録処理は `report` 関数の引数のクロージャ内に定義します。

`app.php` :
```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        ～略～
    )
    ->withMiddleware(function (Middleware $middleware) {
        ～略～
    })
    ->withExceptions(function (Exceptions $exceptions) {
        // 以下を追加
        $exceptions->report(function (TestException $e) {
            // 'test'とログに記録する
            Log::error('test');
        });
    })->create();
```

上記の例では、例外 `TestException` が発生したら `test` とログに記録する処理を定義しています。  
`report` 関数のクロージャの引数の型を指定することで対象の例外を決めることができます。  

上記のように定義してもデフォルトのログ記録処理は行われます。  
デフォルトの挙動を止めたい場合は `stop` 関数を使用するか、クロージャ内で `false` を返すようにします。  

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->report(function (TestException $e) {
        // 'test'とログに記録する
        Log::error('test');
    })->stop(); // stop関数を追加
})
 ```
 ```php
 ->withExceptions(function (Exceptions $exceptions) {
    $exceptions->report(function (TestException $e) {
        // 'test'とログに記録する
        Log::error('test');
        return false; // falseを返すように変更
    });
})
```

### Exceptionクラスの子クラスで定義する場合
任意の `Exception` クラスの子クラスでログ記録処理を定義する場合、以下のように定義できます。

作成した `app/Exceptions/TestException.php` を開き、`report` 関数を定義します。  

`TestException.php` :
```php
～略～
class TestException extends Exception
{
    // 以下を追加
    public function report(): void
    {
        // 'test'とログに記録する
        Log::error('test');
    }
}
```

上記の例では、例外 `TestException` が発生したら `test` とログに記録する処理を定義しています。  

特定の条件が満たされたときのみ実行したい処理がある場合、`true` や `false` を返すことでデフォルト処理とカスタム処理を使い分けることができます。  
`true` のときはカスタム処理、`false` のときはデフォルト処理になります。

```php
public function report(Request $request): bool
{
    $user = $request->user();
    if ($user === null) {
        // 認証済みユーザーでなければデフォルト処理
        return false;
    }

    if ($user->id % 2 === 0) {
        // ユーザーIDが偶数のとき'test'とログに記録する(カスタム処理)
        Log::error('test');
        return true;
    }

    // ユーザーIDが奇数のときデフォルト処理
    return false;
}
```

## 例外発生時のレスポンス処理(render)
デフォルトでは、発生した例外はHTTPリクエストの `Accept` ヘッダーを見て自動でHTMLレスポンスまたはJSONレスポンスに変換されます。  
この処理は `bootstrap/app.php` や `Exception` クラスの子クラスで好きなように定義できます。

### app.phpで定義する場合
`bootstrap/app.php` でレスポンス処理を定義する場合、以下のように定義できます。

`bootstrap/app.php` を開き、`withExceptions` 関数の引数のクロージャ内で `render` 関数を呼び出します。  
レスポンス処理は `render` 関数の引数のクロージャ内に定義します。

`app.php` :
```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        ～略～
    )
    ->withMiddleware(function (Middleware $middleware) {
        ～略～
    })
    ->withExceptions(function (Exceptions $exceptions) {
        // 以下を追加
        $exceptions->render(function (TestException $e) {
            return response()->json(['error_message' => $e->getMessage()]);
        });
    })->create();
```

上記の例では、例外 `TestException` が発生したらエラーメッセージを含めたJSON形式のレスポンスを返す処理を定義しています。  
`render` 関数のクロージャの引数の型を指定することで対象の例外を決めることができます。  

`render` 関数のクロージャの戻り値は、`response` ヘルパーを使って `Illuminate\Http\Response` クラスまたはその子クラスにします。  
クロージャが何も返さない場合、デフォルトの挙動になります。

### Exceptionクラスの子クラスで定義する場合
任意の `Exception` クラスの子クラスでレスポンス処理を定義する場合、以下のように定義できます。

作成した `app/Exceptions/TestException.php` を開き、`render` 関数を定義します。  

`TestException.php` :
```php
～略～
class TestException extends Exception
{
    // 以下を追加
    public function render(): JsonResponse
    {
        return response()->json(['error_message' => $this->message]);
    }
}
```

上記の例では、例外 `TestException` が発生したらエラーメッセージを含めたJSON形式のレスポンスを返す処理を定義しています。  

特定の条件が満たされたときのみ返したいレスポンスがある場合、`false` を返すことでデフォルト処理とカスタム処理を使い分けることができます。  
`false` を返すとデフォルト処理になります。

```php
public function report(Request $request): bool
{
    $user = $request->user();
    if ($user === null) {
        // 認証済みユーザーでなければデフォルト処理
        return false;
    }

    if ($user->id % 2 === 0) {
        // ユーザーIDが偶数のときエラーメッセージを含めたJSON形式のレスポンスを返す(カスタム処理)
        return response()->json(['error_message' => $this->message]);
    }

    // ユーザーIDが奇数のときデフォルト処理
    return false;
}
```