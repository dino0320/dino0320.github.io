---
layout: my-post
title: "Laravelのログを標準出力に設定する"
date: 2024-05-01 00:00:00 +0000
categories: web-application-framework laravel
title_eng: setting-laravel-log-to-stdout
---

Laravelのログを標準出力に出力するように設定します。  

Dockerコンテナ上のLaravelアプリケーションのログをDockerのログとして確認できるようにするため、Laravelのログを標準出力に出力するようにします。  
今回はAmazon Linux 2023のイメージのコンテナ上に作成したLaravelアプリケーションの設定を行います。

## 参考ページ
- [Logging](https://laravel.com/docs/11.x/logging)
- [Docker運用のためのLaravelログ出力](https://qiita.com/batch9703/items/e277b2a2a4caa967ed99)

## 前提
- Laravelプロジェクトを動かすDockerコンテナがある。  
LaravelプロジェクトをNGINXで動かす方法については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。

## 環境
- Windows 10 64ビット
- WSL2 + Ubuntu 22.04.3 LTS
- Docker Engine 26.0.0
- Amazon Linux 2023(DockerコンテナのOS)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 11

## 設定の流れ
1. [Laravelのログの設定を変更する。](#laravelのログの設定を変更する)
2. [php-fpmのログ出力先を変更する。](#php-fpmのログ出力先を変更する)
3. [Laravelのログを確認する。](#laravelのログを確認する)

## Laravelのログの設定を変更する
Laravelはデフォルトではファイルにログを出力するようになっています。  
ファイルに加えて、標準出力にもログを出力するように設定を変更します。

結論から言うと、`.env` のログ関連の環境変数を以下のように変更します。

`.env` :
```
LOG_CHANNEL=stack
LOG_STACK="single,stderr"    # stderr を追加
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug
```

以下は上記の説明になります。

`.env` で指定されているように、デフォルトでは `stack` というログチャンネルを使用しています。  
ログの設定が記載されている `config/logging.php` を開くと、`stack` の定義を確認できます。

`logging.php` :
```php
'stack' => [
    'driver' => 'stack',
    'channels' => explode(',', env('LOG_STACK', 'single')),
    'ignore_exceptions' => false,
],
```

`stack` はドライバーに `stack` を使用しており、`channels` に指定した複数のログチャンネルをまとめることができます。  
`channels` には `LOG_STACK` に指定したログチャンネルが指定されます。

`LOG_STACK` は `.env` に定義されているため確認すると、デフォルトで `single` が指定されていると思います。  

`.env` :
```
LOG_CHANNEL=stack
LOG_STACK=single
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug
```

`single` はファイルにログを出力するためのログチャンネルで、`config/logging.php` に定義されているはずです。  
また、`config/logging.php` には標準エラー出力にログを出力するためのログチャンネル `stderr` も定義されていると思います。  

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

ファイルへのログ出力を残しつつ標準出力への出力を追加したいため、`LOG_STACK` の `single` をそのままにして `stderr` を追加します。  
`LOG_STACK` は `,` で区切ることで複数のチャンネルを指定できるため、以下の結論になりました。

`.env` :
```
LOG_CHANNEL=stack
LOG_STACK="single,stderr"    # stderr を追加
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug
```

## php-fpmのログ出力先を変更する
Laravelのログの設定を変更してもDockerのログに出力されていないかもしれません。  
これはphp-fpmのログが標準出力に出力されていないからのようなので、php-fpmのログの出力先を変更します。

php-fpmのエラーログの出力先は `php-fpm.conf` のグローバル設定項目 `error_log` を変更することで変更できます。  
Amazon Linux 2023の場合、`php-fpm.conf` は `/etc` ディレクトリにありました。

`php-fpm.conf` の `error_log` の値を標準エラー出力のリンクに変更します。  

`php-fpm.conf` :
```conf
; Error log file
; If it's set to "syslog", log is sent to syslogd instead of being written
; in a local file.
; Default Value: /var/log/php-fpm.log
error_log = /dev/stderr    ; 変更
```

また、標準出力を可能にするため、`www.conf` の `catch_workers_output` を `yes` にします。  
さらにログの表示をシンプルにするため、`decorate_workers_output` を `no` にします。

`php-fpm.d` ディレクトリにphp-fpmの追加のconfigファイル `zzz-www.config` を作成します。  
Amazon Linux 2023の場合、`php-fpm.d` ディレクトリは `/etc` ディレクトリにありました。

`zzz-www.conf` :
```conf
[www]
catch_workers_output = yes
decorate_workers_output = no
```

php-fpmを再起動すると設定が反映され、標準エラー出力にログが出力されるようになるはずです。

### Dockerイメージビルド時にログ出力先を変更する
Laravelアプリケーションを動かすDockerイメージをビルドするときにphp-fpmの設定を変更してみます。

#### 方法1
php-fpmの追加のconfigファイル `zzz-www.config` を作成します。  
`Dockerfile` と同じディレクトリに作成しています。

`zzz-www.conf` :
```conf
[www]
catch_workers_output = yes
decorate_workers_output = no
```

`Dockerfile` に `zzz-www.conf` をコピーする処理と、`error_log` の出力先を標準エラー出力のリンクに置換する処理を追加します。

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

～php-fpmのインストール処理～

# 以下を追加
COPY zzz-www.conf /etc/php-fpm.d/zzz-www.conf    # 追加のconfigファイルをコピー
RUN sed -i "s/error_log = .*/error_log = \/dev\/stderr/" /etc/php-fpm.conf    # error_log を変更
```

Dockerコンテナを起動すると、Laravelのログが標準エラー出力に出力されるはずです。

#### 方法2
php-fpmの追加のconfigファイル `zzz-php-fpm.conf` を作成します。  
`Dockerfile` と同じディレクトリに作成しています。

`zzz-php-fpm.conf` :
```conf
[global]
error_log = /dev/stderr
```

php-fpmの追加のconfigファイル `zzz-www.config` を作成します。  
`Dockerfile` と同じディレクトリに作成しています。

`zzz-www.conf` :
```conf
[www]
catch_workers_output = yes
decorate_workers_output = no
```

`Dockerfile` を変更します。  
`zzz-php-fpm.conf` と `zzz-www.conf` をコピーする処理と、`error_log` をコメントアウトする処理を追加します。(コメントアウトしないとデフォルトの設定が反映されてしまったので)

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

～php-fpmのインストール処理～

# 以下を追加
COPY zzz-php-fpm.conf /etc/php-fpm.d/zzz-php-fpm.conf    # zzz-php-fpm.conf をコピー
COPY zzz-www.conf /etc/php-fpm.d/zzz-www.conf    # zzz-www.conf をコピー
RUN sed -i "s/error_log/;error_log/" /etc/php-fpm.conf    # error_log をコメントアウト
```

Dockerコンテナを起動すると、Laravelのログが標準エラー出力に出力されると思います。

## Laravelのログを確認する
適当な場所にログの処理を追加して設定を確認します。  

例えば `/` へのアクセス時にログを出力するようにします。
```php
Route::get('/', function () {
    Log::info('test'); // 追加
});
```

ブラウザ等で `/` にアクセスします。  
Dockerのログを確認すると、以下のようにログが出力されていると思います。
```
[2024-05-01 06:52:04] local.INFO: test
```

## (おまけ) PHPのログを標準出力に設定する
PHPのログも標準出力に出力してみます。

PHPのエラーログの出力先は `www.conf` の `php_admin_value[error_log]` を変更することで変更できるため、値が標準エラー出力のリンクになるようにします。  

また、標準出力を可能にするため、`www.conf` の `catch_workers_output` を `yes` にします。  
さらにログの表示をシンプルにするため、`decorate_workers_output` を `no` にします。

上記の設定をするため、`php-fpm.d` ディレクトリにphp-fpmの追加のconfigファイル `zzz-www.config` を作成します。  
Amazon Linux 2023の場合、`php-fpm.d` ディレクトリは `/etc` ディレクトリにありました。

`zzz-www.conf` :
```conf
[www]
catch_workers_output = yes
decorate_workers_output = no
php_admin_value[error_log] = /dev/stderr
```

php-fpmを再起動すると設定が反映され、標準エラー出力にログが出力されるようになるはずです。

### Dockerイメージビルド時にログ出力先を変更する
Laravelアプリケーションを動かすDockerイメージをビルドするときにphp-fpmの設定を変更してみます。

php-fpmの追加のconfigファイル `zzz-www.config` を作成します。  
`Dockerfile` と同じディレクトリに作成しています。

`zzz-www.conf` :
```conf
[www]
catch_workers_output = yes
decorate_workers_output = no
php_admin_value[error_log] = /dev/stderr
```

`Dockerfile` を変更します。  
`zzz-www.conf` をコピーする処理と、`php_admin_value[error_log]` をコメントアウトする処理を追加します。(コメントアウトしないとデフォルトの設定が反映されてしまったので)

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

～php-fpmのインストール処理～

# 以下を追加
COPY zzz-www.conf /etc/php-fpm.d/zzz-www.conf    # zzz-www.conf をコピー
RUN sed -i "s/php_admin_value\[error_log\]/;php_admin_value\[error_log\]/" /etc/php-fpm.d/www.conf    # php_admin_value[error_log] をコメントアウト
```

Dockerコンテナを起動すると、PHPのログが標準エラー出力に出力されると思います。