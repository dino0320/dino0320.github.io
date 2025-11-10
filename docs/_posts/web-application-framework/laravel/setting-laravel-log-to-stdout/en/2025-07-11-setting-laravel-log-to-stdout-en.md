---
layout: my-post
title: "Configure Laravel to Output Logs to Standard Output"
date: 2025-07-11 00:00:00 +0000
categories: web-application-framework laravel
page_name: setting-laravel-log-to-stdout-en
lang: en
---

This article is how to configure Laravel to output logs to standard output.

To make it possible to view logs from a Laravel application running inside a Docker container as Docker logs, we will configure Laravel to output its logs to standard output.  
In this example, we will configure a Laravel application running in a container based on the Amazon Linux 2023 image.

## References
- [Logging - Laravel 11.x](https://laravel.com/docs/11.x/logging)
- [Docker運用のためのLaravelログ出力 - Qiita](https://qiita.com/batch9703/items/e277b2a2a4caa967ed99)

## Prerequisites
- You have a Docker container running a Laravel project.  
To learn how to run a Laravel project with NGINX, please refer to [this guide](/web-application-framework/laravel/running-laravel-project-on-nginx-en).

## Environment
- Windows 10 64-bit
- WSL2 + Ubuntu 22.04.3 LTS
- Docker Engine 26.0.0
- Amazon Linux 2023(OS of the Docker container)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 11

## Configuration Steps
1. [Change Laravel Log Settings](#1-change-laravel-log-settings)
2. [Change PHP-FPM Log Output Destination](#2-change-php-fpm-log-output-destination)
3. [Check Laravel Logs](#3-check-laravel-logs)

## 1. Change Laravel Log Settings
By default, Laravel logs are written to files.  
We will change the configuration to also output logs to standard output in addition to files.

In short, change the log-related environment variables in `.env` as follows:

`.env` :
```
LOG_CHANNEL=stack
LOG_STACK="single,stderr"    #  Add stderr
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug
```

Here is an explanation of the above.

As specified in `.env`, Laravel uses the `stack` log channel by default.  
Open `config/logging.php` to see how stack is defined.

`logging.php` :
```php
'stack' => [
    'driver' => 'stack',
    'channels' => explode(',', env('LOG_STACK', 'single')),
    'ignore_exceptions' => false,
],
```

The `stack` driver allows you to combine multiple log channels.  
The channels used are specified in the `LOG_STACK` environment variable.

By default, `LOG_STACK` is set to `single` in `.env`:

`.env` :
```
LOG_CHANNEL=stack
LOG_STACK=single
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug
```

The `single` channel is used to write logs to a file and is defined in `config/logging.php`.  
The `stderr` channel, which outputs to standard error, should also be defined there.

`logging.php` :
```php
'single' => [
    'driver' => 'single',
    'path' => storage_path('logs/laravel.log'),
    'level' => env('LOG_LEVEL', 'debug'),
    'replace_placeholders' => true,
],
```
```php
'stderr' => [
    'driver' => 'monolog',
    'level' => env('LOG_LEVEL', 'debug'),
    'handler' => StreamHandler::class,
    'formatter' => env('LOG_STDERR_FORMATTER'),
    'with' => [
        'stream' => 'php://stderr',
    ],
    'processors' => [PsrLogMessageProcessor::class],
],
```

To keep file logging and also output to standard error, leave `single` and add `stderr` to LOG_STACK.  
Since multiple channels can be specified with commas, the final result looks like this:

`.env` :
```
LOG_CHANNEL=stack
LOG_STACK="single,stderr"    # Add stderr
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug
```

## 2. Change PHP-FPM Log Output Destination
Even after updating Laravel's log settings, the logs may not appear in Docker logs.  
This is because PHP-FPM logs are not sent to standard output by default, so we need to change their destination.

To change the error log output of PHP-FPM, edit the global `error_log` setting in `php-fpm.conf`.  
In Amazon Linux 2023, `php-fpm.conf` is located in the `/etc` directory.

Change the value of `error_log` to point to the standard error stream:

`php-fpm.conf` :
```conf
; Error log file
; If it's set to "syslog", log is sent to syslogd instead of being written
; in a local file.
; Default Value: /var/log/php-fpm.log
error_log = /dev/stderr    ; Changed
```

Also, enable standard output by setting `catch_workers_output` to `yes` in `www.conf`.  
To simplify log formatting, set `decorate_workers_output` to `no`.

Create an additional PHP-FPM config file named `zzz-www.conf` in the `php-fpm.d` directory.
On Amazon Linux 2023, this directory is located under `/etc`.

`zzz-www.conf` :
```conf
[www]
catch_workers_output = yes
decorate_workers_output = no
```

After restarting PHP-FPM, the configuration should be applied and logs should start appearing in standard error output.

### Change Log Output During Docker Image Build
We will apply these PHP-FPM settings while building the Docker image.

#### Method 1
Create an additional PHP-FPM config file named `zzz-www.conf`.  
Place it in the same directory as the `Dockerfile`.

`zzz-www.conf` :
```conf
[www]
catch_workers_output = yes
decorate_workers_output = no
```

Update the `Dockerfile` to copy `zzz-www.conf` and replace `error_log` with `/dev/stderr`.

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

... install php-fpm, etc.

# Add the following
COPY zzz-www.conf /etc/php-fpm.d/zzz-www.conf
# Change error_log
RUN sed -i "s/error_log = .*/error_log = \/dev\/stderr/" /etc/php-fpm.conf
```

Once the container starts, Laravel logs should be output to standard error.

#### Method 2
Create an additional PHP-FPM config file named `zzz-php-fpm.conf`.  
Place it in the same directory as the `Dockerfile`.

`zzz-php-fpm.conf` :
```conf
[global]
error_log = /dev/stderr
```

Create an additional PHP-FPM config file named `zzz-www.config`.  
Place it in the same directory as the `Dockerfile`.

`zzz-www.conf` :
```conf
[www]
catch_workers_output = yes
decorate_workers_output = no
```

Update the `Dockerfile` to copy both files and comment out the default `error_log`:

Unless you comment it out, the default setting will be applied.

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

... install php-fpm, etc.

# Add the following
COPY zzz-php-fpm.conf /etc/php-fpm.d/zzz-php-fpm.conf
COPY zzz-www.conf /etc/php-fpm.d/zzz-www.conf
# Comment error_log out
RUN sed -i "s/error_log/;error_log/" /etc/php-fpm.conf
```

When you start the Docker container, Laravel logs should be output to standard error.

## 3. Check Laravel Logs
Add a log entry somewhere to verify the setup.

For example, log something when accessing `/`:

```php
Route::get('/', function () {
    Log::info('test'); // Added
});
```

Access `/` from your browser.  
When you check the Docker logs, you should see output like this:

```
[2024-05-01 06:52:04] local.INFO: test
```

## (Bonus) Output PHP Logs to Standard Output
You can also configure PHP error logs to be output to standard output.

This can be done by setting `php_admin_value[error_log]` in `www.conf` to `/dev/stderr`.  
Also, set `catch_workers_output = yes` and `decorate_workers_output = no`.

To apply these settings, create a PHP-FPM config file `zzz-www.conf` under the `php-fpm.d` directory (in `/etc` on Amazon Linux 2023):

`zzz-www.conf` :
```conf
[www]
catch_workers_output = yes
decorate_workers_output = no
php_admin_value[error_log] = /dev/stderr
```

After restarting PHP-FPM, the settings should take effect, and PHP logs will be output to standard error.

### Change Log Output During Docker Image Build
To apply this during Docker image build, create `zzz-www.conf` in the same directory as the `Dockerfile`.

`zzz-www.conf` :
```conf
[www]
catch_workers_output = yes
decorate_workers_output = no
php_admin_value[error_log] = /dev/stderr
```

Update the `Dockerfile` to copy this file and comment out the existing `php_admin_value[error_log]` setting:

Unless you comment it out, the default setting will be applied.

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

... install php-fpm, etc.

# Add the following
COPY zzz-www.conf /etc/php-fpm.d/zzz-www.conf
# Comment php_admin_value[error_log] out
RUN sed -i "s/php_admin_value\[error_log\]/;php_admin_value\[error_log\]/" /etc/php-fpm.d/www.conf
```

Once you launch the Docker container, the PHP error logs should be output to standard error.