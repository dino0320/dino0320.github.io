---
layout: my-post
title: "How to Run Code After a Successful Stripe Payment Using Webhooks"
date: 2025-12-03 00:00:00 +0000
categories: web-application-framework laravel
page_name: how-to-run-code-after-successful-stripe-payment-en
lang: en
image: /assets/images/web-application-framework/laravel/how-to-run-code-after-successful-stripe-payment-en/image1.png
---

This article explains how to run code after a successful Stripe payment using Webhooks.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## References
- [Laravel Cashier (Stripe)](https://laravel.com/docs/12.x/billing)
- [Stripe CLI](https://docs.stripe.com/stripe-cli#install)
- [Stripe CLI: How to trigger events with nested metadata](https://stackoverflow.com/questions/70465253/stripe-cli-how-to-trigger-events-with-nested-metadata)

## Prerequisites
- You have a Laravel project.  
To learn how to run a Laravel project with NGINX, please refer to [this guide](/web-application-framework/laravel/running-laravel-project-on-nginx-en).  
If you are using React, refer to [this article](/web-application-framework/laravel/set-up-laravel-with-react-project-en).
- You have already installed Laravel Cashier (Stripe).  
For more information, refer to [this article](/web-application-framework/laravel/install-laravel-cashier-en).

## Environment
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2 distribution)
- Docker Engine 28.4.0
- Amazon Linux 2023(OS of the Docker container)
- PHP 8.2.29 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 12.38.1
- laravel/cashier 16.0.5

## Workflow
1. [Create Test Checkout](#1-create-test-checkout)
2. [Create Stripe Event Listener](#2-create-stripe-event-listener)
3. [Verify Result Locally](#3-verify-result-locally)

- [(Bonus) Retrieve Metadata in Listener](#bonus-retrieve-metadata-in-listener)

## 1. Create Test Checkout
Refer to [the official document](https://laravel.com/docs/12.x/billing#quickstart) and create a test checkout like the following:

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

Specify the price ID you created in Stripe in `$stripePriceId`.  
You also need to prepare the `checkout.success` and `checkout.cancel` views.

When you access `/checkout`, you should see the Stripe Checkout screen.  
After successfully completing the payment, you will be redirected to `/checkout/success`.

## 2. Create Stripe Event Listener
Create an event listener to run code after the application receives Stripe events.  
Run the following Artisan command:

```bash
php artisan make:listener StripeEventListener --event=WebhookReceived
```

Edit the generated listener as follows:

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

`payment_intent.succeeded` is an event that occurs when a payment is successfully completed.  
You can check all Stripe event types [here](https://docs.stripe.com/api/events/types).

Add the listener inside the `boot()` method in `AppServiceProvider.php`:

```php
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Event::listen(WebhookReceived::class, StripeEventListener::class); // Added
    }
```

## 3. Verify Result Locally
Verify the result locally by running the following Stripe CLI commands.  
To install Stripe CLI, refer to [this page](https://docs.stripe.com/stripe-cli#install).

```bash
> stripe login
> stripe listen --forward-to localhost:80/stripe/webhook
```

Specify your application's port and webhook path.  
Laravel Cashier listens on `/stripe/webhook` by default.

Access `/checkout`, and you should see the message `The payment succeeded` in your application logs.

## (Bonus) Retrieve Metadata in Listener
How can you retrieve any custom values in the listener?  
Add metadata during checkout:

```php
Route::get('/checkout', function (Request $request) {
    $stripePriceId = 'price_deluxe_album';
 
    $quantity = 1;
 
    return $request->user()->checkout([$stripePriceId => $quantity], [
        'success_url' => route('checkout-success'),
        'cancel_url' => route('checkout-cancel'),
        // Add the following
        'payment_intent_data' => [
            'metadata' => ['test_key' => 'test_value'],
        ],
    ]);
})->name('checkout');
```

Retrieve the metadata in the listener:

```php
    /**
     * Handle the event.
     */
    public function handle(WebhookReceived $event): void
    {
        if ($event->payload['type'] === 'payment_intent.succeeded') {
            Log::info('Test data: ' . $event->payload['data']['object']['metadata']['test_key']); // display the metadata
        }
    }
```

If you want to trigger an event with metadata using Stripe CLI, you can specify it like this:

```bash
Stripe trigger payment_intent.succeeded --add payment_intent:metadata['test_key']=test_value
```