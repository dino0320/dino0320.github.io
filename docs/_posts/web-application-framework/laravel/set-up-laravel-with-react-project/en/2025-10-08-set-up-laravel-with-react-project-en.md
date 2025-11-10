---
layout: my-post
title: "Set Up Laravel with a React Project"
date: 2025-10-08 00:00:00 +0000
categories: web-application-framework laravel
page_name: set-up-laravel-with-react-project-en
lang: en
---

This article explains how to set up a Laravel project with React using Inertia.js.

## References
- [Using React or Vue](https://laravel.com/docs/12.x/frontend#using-react-or-vue)
- [Server-side setup](https://inertiajs.com/server-side-setup)
- [Client-side setup](https://inertiajs.com/client-side-setup)
- [Pages](https://inertiajs.com/pages)
- [VPS環境でlaravel+reactで環境作成時のエラー備忘録「 ERR_CONNECTION_REFUSED」](https://qiita.com/aimkbiz/items/8e30dccd8d5906e83ef5)

## What is inertia?
Inertia allows us to use front-end frameworks like React or Vue with Laravel.  
For more information, see [this section of the Laravel documentation](https://laravel.com/docs/12.x/frontend#using-react-or-vue).

> Inertia bridges the gap between your Laravel application and your modern React or Vue frontend, allowing you to build full-fledged, modern frontends using React or Vue while leveraging Laravel routes and controllers for routing, data hydration, and authentication — all within a single code repository.

## Environment
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2 distribution)
- Docker Engine 28.4.0
- Amazon Linux 2023(OS of the Docker container)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 12
- Node.js 20.14.0
- Vite 7.1.9

## Prerequisites
- You have a Docker container running a Laravel project on WSL (using Docker Compose).  
For running a Laravel project with NGINX, refer to [this article](/web-application-framework/laravel/running-laravel-project-on-nginx-en).
- Vite is already installed.  
For instructions, see [this article](/web-application-framework/laravel/installing-vite-with-laravel-on-wsl-and-docker-en).

## Setting Up Steps
1. [Install Composer Package](#1-install-composer-package)
2. [Create Root Template](#2-create-root-template)
3. [Set Up Middleware](#3-set-up-middleware)
4. [Install NPM Packages](#4-install-npm-packages)
5. [Edit vite.config.js](#5-edit-viteconfigjs)
6. [Create Main JavaScript File](#6-create-main-javascript-file)
7. [Check Inertia Page](#7-check-inertia-page)

## 1. Install Composer Package
Install the following Composer package:  
(If you don't have Composer installed, refer to [this article](/programming/php/installing-composer-on-linux-en).)

```bash
composer require inertiajs/inertia-laravel
```

## 2. Create Root Template
Create a root template named `app.blade.php` in the `resources/views` directory, as described in [this article](https://inertiajs.com/server-side-setup).

> setup the root template that will be loaded on the first page visit to your application. This template should include your site's CSS and JavaScript assets, along with the @inertia and @inertiaHead directives.

`app.blade.php`:

```php
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1"> 
    @viteReactRefresh
    @vite('resources/js/app.jsx')
    @inertiaHead
  </head>
  <body>
    @inertia
  </body>
</html>
```

Make sure to include `@viteReactRefresh` before `@vite`, as noted in the official documentation:

> For React applications, it's recommended to include the @viteReactRefresh directive before the @vite directive to enable Fast Refresh in development.

This template loads `resources/js/app.jsx` via `@vite`, which will serve as the JavaScript entry point to boot your Inertia app (created later).

## 3. Set Up Middleware
Generate the `HandleInertiaRequests` middleware using the Artisan command:

```bash
php artisan inertia:middleware
```

Then append it to the `web` middleware group in your `bootstrap/app.php` file:

`app.php`:

```php
<?php

use App\Http\Middleware\HandleInertiaRequests;
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware): void {
        $middleware->web(append: [
            HandleInertiaRequests::class, // Add this line
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions): void {
        //
    })->create();

```

## 4. Install NPM Packages
Install the following NPM packages:

```bash
npm install @inertiajs/react @vitejs/plugin-react react react-dom
```

## 5. Edit vite.config.js
Edit the `vite.config.js` file as follows:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import tailwindcss from '@tailwindcss/vite';
import react from '@vitejs/plugin-react'; // Add

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.jsx'], // Specify resources/js/app.jsx
            refresh: true,
        }),
        tailwindcss(),
        react(), // Add
    ],
    server: {
        host: true,
        hmr: {
            host: 'localhost',
        },
    },
});

```

The `react()` method enables support for `@viteReactRefresh` in your Blade template.

Notes:  
`server.host`: specifies the IP address that the server should listen on.  
By default, it only listens to `localhost`, but setting it to `0.0.0.0` or `true` will allow it to listen to all addresses.  
This makes it possible to access the server inside the container from the Docker host.

`server.hmr.hostspecifies` the IP address for HMR (Hot Module Replacement) communication.  
Since the address is within the Docker container, we set it to `localhost`. HMR allows CSS and JavaScript files to be updated dynamically without reloading the browser.

## 6. Create Main JavaScript File
Create a main JavaScript file to boot your Inertia app named `app.jsx` inside the `resources/js` directory:

`app.jsx`:

```jsx
import { createInertiaApp } from '@inertiajs/react'
import { createRoot } from 'react-dom/client'

createInertiaApp({
  resolve: name => {
    const pages = import.meta.glob('./Pages/**/*.jsx', { eager: true })
    return pages[`./Pages/${name}.jsx`]
  },
  setup({ el, App, props }) {
    createRoot(el).render(<App {...props} />)
  },
})

```

## 7. Check Inertia Page
Create an Inertia page to verify that everything is working correctly.

Create `Welcome.jsx` in the `resources/js/Pages` directory.

`Welcome.jsx`:

```jsx
import { Head } from '@inertiajs/react'

export default function Welcome() {
  return (
    <div>
      <Head title="Welcome" />
      <h1>Welcome</h1>
      <p>Hello, welcome to your first Inertia app!</p>
    </div>
  )
}

```

Then, add the following route to `routes/web.php` so that visiting `/` displays the page:

`web.php`:

```php
<?php

use Illuminate\Support\Facades\Route;
use Inertia\Inertia;

Route::get('/', function () {
    return Inertia::render('welcome');
});

```

### Start the Development Server
Start the development server so CSS and JavaScript files are updated dynamically.

In the root directory of your Laravel project, run the following command:

```bash
npm run dev
```

Access `/` in your browser.  
You should see a page like this.

![Inertia test page](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Inertia test page")

### Build for Production
To build optimized CSS and JavaScript files, run:

```bash
npm run build
```

Access `/` in your browser.  
The result should remain the same.