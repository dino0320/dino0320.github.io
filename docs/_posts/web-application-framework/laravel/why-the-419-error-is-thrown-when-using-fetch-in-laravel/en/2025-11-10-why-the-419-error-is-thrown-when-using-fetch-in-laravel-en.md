---
layout: my-post
title: "Why the 419 Error Is Thrown When Using Fetch in Laravel?"
date: 2025-11-10 00:00:00 +0000
categories: web-application-framework laravel
page_name: why-the-419-error-is-thrown-when-using-fetch-in-laravel-en
lang: en
image: /assets/images/web-application-framework/laravel/why-the-419-error-is-thrown-when-using-fetch-in-laravel-en/image1.png
---

I tried to use the `fetch` method in TypeScript code within the Laravel project.  
However, a 419 error was thrown.  
This article explains the reason and its solution.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## References
- [CSRF Protection](https://laravel.com/docs/12.x/csrf)
- [Cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery)

## Prerequisite
- You have a Laravel project.  
To learn how to run a Laravel project with NGINX, please refer to [this guide](/web-application-framework/laravel/running-laravel-project-on-nginx-en).  
If you are using React, refer to [this article](/web-application-framework/laravel/set-up-laravel-with-react-project-en).

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
- React 19.2.0

## Table of Contents
- [The Reason](#the-reason)
- [The Solution](#the-solution)

## The Reason
I tried to send an HTTP POST request to a Laravel server using the `fetch` method in TypeScript as follows:

```ts
fetch('/test', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ name: 'test' })
})
.then(data => {
  console.log(data)
})
```

However, a 419 error was returned.

This happens because Laravel protects your application from **[CSRF](#what-is-csrf)** attacks.  
When we use the `fetch` method, CSRF protection logic isn't automatically applied to the request.  
As a result, Laravel can't verify whether the request is valid and therefore throws a 419 error.

### What is CSRF?
CSRF (Cross-site request forgery) is an attack in which unauthorized commands are submitted from a user that the web application trusts, as described on [this page](https://en.wikipedia.org/wiki/Cross-site_request_forgery).  
For example, when a user authenticated by your application submits a form created by attackers on their own webpage, the user could unknowingly send a request that the attackers intend, such as changing their email address or password.

To prevent this kind of attack, the application must verify whether the request originates from a trusted source, for example by checking a CSRF token.

## The Solution
To solve this problem, it's better to use the `axios` library instead of the `fetch` method, as shown below:

```ts
const response = await axios.post('test', {
  name: 'test',
});
console.log(response.data)
```

As [the official document](https://laravel.com/docs/12.x/csrf) explains,  
Laravel automatically generates a **CSRF token** for each active user session managed by the application.  
This token is stored in the user's session and is verified by the `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` middleware, which ensures that the token in the request matches the one stored in the session.  
This middleware is included in the `web` middleware group by default.

The `axios` library automatically includes the token stored in the `XSRF-TOKEN` cookie in the `X-XSRF-TOKEN` header.  
Therefore, as long as you use `axios`, you don't need to manually set the token in the request headers.

However, the `fetch` method doesn't do this automatically.  
If you use `fetch` to send a request to a Laravel application, you need to manually include the CSRF token in the headers as follows:

```ts
const token = decodeURIComponent(getCookie('XSRF-TOKEN'));
fetch('/test', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-XSRF-TOKEN': token
  },
  body: JSON.stringify({ name: 'test' })
})
.then(data => {
  console.log(data)
})

function getCookie(key: string) {
  const cookies = `; ${document.cookie}`
  const splitedCookies = cookies.split(`; ${key}=`)
  if (splitedCookies.length === 2) { // The first element should be an empty string
    const cookie = splitedCookies.pop() ?? ''
    // The cookie string might contain other cookies at the end, so split it and take the first value
    return cookie.split(';').shift() ?? ''
  }

  return ''
}
```