---
layout: my-post
title: "Stripeの支払い成功後にWebhooksで購入後の処理を実行する"
date: 2025-12-03 00:00:00 +0000
categories: web-application-framework laravel
page_name: how-to-run-code-after-successful-stripe-payment
lang: ja
image: /assets/images/web-application-framework/laravel/how-to-run-code-after-successful-stripe-payment/image1.png
---

Stripeの支払い成功後、Webhookで購入後の処理を実行する方法をまとめました。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 参考
- [Laravel Cashier (Stripe)](https://laravel.com/docs/12.x/billing)
- [Stripe CLI](https://docs.stripe.com/stripe-cli#install)
- [Stripe CLI: How to trigger events with nested metadata](https://stackoverflow.com/questions/70465253/stripe-cli-how-to-trigger-events-with-nested-metadata)

## 前提
- Laravelプロジェクトを作成済み  
LaravelプロジェクトをNGINXで動かす方法については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。  
Reactを使用する場合は[こちら](/web-application-framework/laravel/set-up-laravel-with-react-project)をご覧ください。
- Laravel Cashier (Stripe) をインストール済み  
インストール方法については[こちら](/web-application-framework/laravel/install-laravel-cashier)をご覧ください。

## 環境
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2ディストリビューション)
- Docker Engine 28.4.0
- Amazon Linux 2023(DockerコンテナのOS)
- PHP 8.2.29 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 12.38.1
- laravel/cashier 16.0.5

## ワークフロー
1. [チェックアウト処理の作成](#1-チェックアウト処理の作成)
2. [Stripeイベントリスナーの作成](#2-stripeイベントリスナーの作成)
3. [ローカル環境で確認](#3-ローカル環境で確認)

- [(おまけ) リスナーでmetadataを取得する](#おまけ-リスナーでmetadataを取得する)

## 1. チェックアウト処理の作成
[公式ドキュメント](https://laravel.com/docs/12.x/billing#quickstart)を参考に、次のようにチェックアウト処理を作成します。

```php
use Illuminate\Http\Request;
 
Route::get('/checkout', function (Request $request) {
    $stripePriceId = 'price_deluxe_album';
 
    $quantity = 1;
 
    return $request->user()->checkout([$stripePriceId => $quantity], [
        'success_url' => route('checkout-success'),
        'cancel_url' => route('checkout-cancel'),
    ]);
})->name('checkout');
 
Route::view('/checkout/success', 'checkout.success')->name('checkout-success');
Route::view('/checkout/cancel', 'checkout.cancel')->name('checkout-cancel');
```

Stripeで作成したprice IDを `$stripePriceId` に指定してください。  
また、`checkout.success` と `checkout.cancel` のビューも用意しておく必要があります。

`/checkout` にアクセスするとStripeのチェックアウト画面が表示されます。
支払いが成功すると `/checkout/success` にリダイレクトされます。

## 2. Stripeイベントリスナーの作成
アプリケーションがStripeのイベントを受け取った後に何かしらの処理を実行させるためのイベントリスナーを作成します。  
次のArtisanコマンドを実行してください。

```bash
php artisan make:listener StripeEventListener --event=WebhookReceived
```

生成されたリスナーを以下のように編集します。

```php
<?php

namespace App\Listeners;

use Illuminate\Support\Facades\Log;
use Laravel\Cashier\Events\WebhookReceived;

class StripeEventListener
{
    /**
     * Create the event listener.
     */
    public function __construct()
    {
        //
    }

    /**
     * Handle the event.
     */
    public function handle(WebhookReceived $event): void
    {
        if ($event->payload['type'] === 'payment_intent.succeeded') {
            Log::info('The payment succeeded');
        }
    }
}

```

`payment_intent.succeeded` は支払いが成功したときに発火するイベントです。  
Stripeのイベント一覧は[こちら](https://docs.stripe.com/api/events/types)で確認できます。

次に、`AppServiceProvider.php` の `boot()` メソッドでリスナーを登録します。

```php
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Event::listen(WebhookReceived::class, StripeEventListener::class); // 追加
    }
```

## 3. ローカル環境で確認
Stripe CLIを使用してローカル環境で動作を確認します。  
Stripe CLIのインストール方法は[こちら](https://docs.stripe.com/stripe-cli#install)を参照してください。

```bash
> stripe login
> stripe listen --forward-to localhost:80/stripe/webhook
```

アプリケーションのポートとwebhookのパスを指定してください。  
Laravel Cashierはデフォルトで `/stripe/webhook` を使っています。

その後 `/checkout` にアクセスすると、アプリケーションのログに `The payment succeeded` と表示されるはずです。

## (おまけ) リスナーでmetadataを取得する
リスナーで自分が指定した値を取得したい場合、チェックアウト時に `metadata` を追加することで好きな値を送ることができます。

```php
Route::get('/checkout', function (Request $request) {
    $stripePriceId = 'price_deluxe_album';
 
    $quantity = 1;
 
    return $request->user()->checkout([$stripePriceId => $quantity], [
        'success_url' => route('checkout-success'),
        'cancel_url' => route('checkout-cancel'),
        // metadataを追加
        'payment_intent_data' => [
            'metadata' => ['test_key' => 'test_value'],
        ],
    ]);
})->name('checkout');
```

リスナーで `metadata` を取得します。

```php
    /**
     * Handle the event.
     */
    public function handle(WebhookReceived $event): void
    {
        if ($event->payload['type'] === 'payment_intent.succeeded') {
            Log::info('Test data: ' . $event->payload['data']['object']['metadata']['test_key']); // metadataを表示
        }
    }
```

Stripe CLIで `metadata` 付きのイベントをトリガーしたい場合は次のように指定します。

```bash
Stripe trigger payment_intent.succeeded --add payment_intent:metadata['test_key']=test_value
```