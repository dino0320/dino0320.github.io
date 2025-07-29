---
layout: my-post
title: "LaravelでValkeyを使用する"
date: 2025-07-29 00:00:00 +0000
categories: web-application-framework laravel
page_name: use-valkey-with-laravel
lang: ja
---

この記事では、LaravelプロジェクトでValkeyを使用する方法について説明します。

## 参考
- [Client Libraries](https://valkey.io/clients/)

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0
- Amazon Linux 2023(DockerコンテナのOS)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 11

## 前提
- Laravelプロジェクトを動かすDockerコンテナがある。(Docker Compose使用)  
LaravelプロジェクトをNGINXで動かす方法については[こちら](/web-application-framework/laravel/running-laravel-project-on-nginx)をご覧ください。

## 設定の流れ
1. [ValkeyのDockerコンテナを作成](#1-valkeyのdockerコンテナを作成)
2. [Dockerの設定を更新](#2-dockerの設定を更新)
3. [Laravelの設定を更新](#3-laravelの設定を更新)
4. [動作確認](#4-動作確認)

## 1. ValkeyのDockerコンテナを作成
`docker-compose.yml` ファイルにValkeyのサービス設定を追加します。

```yml
services:

...

  # 以下を追加
  valkey:    # サービス名
    image: valkey/valkey:8.1
```

この例では、AWS ElastiCacheでサポートされているバージョンに合わせて `8.1` を指定しています。

## 2. Dockerの設定を更新
ValkeyをPhpRedisクライアントで操作できるようにするため、Dockerイメージのビルド時にPhpRedisをインストールします。

PHP起動時にPhpRedisを読み込むように、`Dockerfile` と同じディレクトリに `50-valkey.ini` という設定ファイルを作成します。

`50-valkey.ini` :

```ini
extension = redis.so
```

`Dockerfile` を以下のように更新してPhpRedisをインストールします。

`Dockerfile` :

```dockerfile
FROM amazonlinux:2023

...

# PhpRedisインストール
RUN yum -y install php-pear-1:1.10.13 php8.2-devel
RUN pecl install redis-6.1.0
COPY 50-valkey.ini /etc/php.d/50-valkey.ini    # PHP設定ファイルをコピー
```

対応しているPhpRedisのバージョンについては[こちらのページ](https://valkey.io/clients/)を参照してください。

## 3. Laravelの設定を更新
`.env` ファイルを以下のように変更してValkeyを使用するようにします。

`.env` :

```
REDIS_CLIENT=phpredis
REDIS_HOST=valkey
REDIS_PASSWORD=null
REDIS_PORT=6379
```

## 4. 動作確認
`docker-compose.yml` があるディレクトリで、以下のコマンドを実行してコンテナを起動します。

```bash
$ docker compose up -d --build
```

`routes/web.php` などのファイルに、Redisにキーを設定するコードを追加します。

`web.php` :

```php
use Illuminate\Support\Facades\Redis;

Route::get('/', function () {
    Redis::set('name', 'Test');
});
```

ブラウザで `/` にアクセスします。

次に、Valkeyコンテナに入ります。

```bash
$ docker compose exec valkey bash
```

Valkeyに接続します。

```
# redis-cli
```

Valkeyに保存されているすべてのキーを確認します。

```
> keys *
1) "<REDIS_PREFIX>_name"
```

`REDIS_PREFIX` は `.env` ファイルで設定されています。

保存された値を確認します。

```
get "<REDIS_PREFIX>_name"
"Test"
```

`Test` という値が正しくValkeyに保存されていることが確認できました。