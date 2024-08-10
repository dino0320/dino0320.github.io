---
layout: my-post
title: "LaravelでRedisを使うようにSessionの設定をする"
date: 2024-05-12 00:00:00 +0000
categories: web-application-framework laravel
title_eng: setting-session-using-redis-on-laravel
---

LaravelでSessionをRedisに保存するように設定を行います。  
Dockerを使用して環境を構築しています。

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
1. [RedisのDockerコンテナを作成する。](#1-redisのdockerコンテナを作成する)
2. [LaravelプロジェクトのDockerコンテナの設定を変更する。](#2-laravelプロジェクトのdockerコンテナの設定を変更する)
3. [Laravelの設定を変更する。](#3-laravelの設定を変更する)
4. [設定の確認をする。](#4-設定の確認をする)

## 1. RedisのDockerコンテナを作成する
RedisのDockerコンテナを作成します。  
`docker-compose.yml` にRedisコンテナの設定を追加します。

`docker-compose.yml` :
```yml
services:

～略～

  # Redisコンテナの設定を追加
  redis:    # サービス名
    image: redis:7.0
```
今回は今後AWSにデプロイすることを考えて、ElastiCacheがサポートしているRedisのバージョンからバージョンを選び、`7.0` を指定します。

## 2. LaravelプロジェクトのDockerコンテナの設定を変更する
LaravelプロジェクトのDockerコンテナの設定を変更します。  

今回はRedisの操作にPhpRedisを使用するので、Dockerイメージのビルド時にPhpRedisをインストールする処理を追加します。

まず、PHPの起動時にPhpRedisをロードするために、PHPの追加の設定ファイル `50-redis.ini` を作成します。  
`Dockerfile` と同じディレクトリに作成しています。

`50-redis.ini` : 
```ini
extension = redis.so
```

次に、`Dockerfile` にPhpRedisをインストールする処理を追加します。  
PhpRedisのバージョンはこの時点で最新のものを指定しています。

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

～省略～

# PhpRedisインストール
RUN yum -y install php-pear-1:1.10.13 php8.2-devel
RUN pecl install redis-6.0.2
COPY 50-redis.ini /etc/php.d/50-redis.ini    # PHPの追加の設定ファイルをコピー
```

## 3. Laravelの設定を変更する
SessionをRedisに保存するようにLaravelの設定を変更します。  
`.env` のSessionに関する環境変数を以下のように変更します。

`.env` :
```
SESSION_DRIVER=redis    # ドライバーをredisに変更
SESSION_LIFETIME=120
SESSION_ENCRYPT=false
SESSION_PATH=/
SESSION_DOMAIN=null
SESSION_CONNECTION=default    # config/database.php を見て使用するRedisサーバーを指定
```

## 4. 設定の確認をする
SessionがRedisに保存されるか確認します。  

Dockerコンテナを起動します。  
`docker-compose.yml` のあるディレクトリに移動し、以下を実行します。
```bash
$ docker compose up -d --build
```

`routes/web.php` などに、`/` にアクセスしたらSessionに値をセットする処理を追加します。

`web.php` : 
```php
Route::get('/', function (Request $request) {
    $request->session()->put('test_key', 'test_value');
});
```

ブラウザなどで `/` にアクセスします。

RedisのDockerコンテナに接続します。  
`<サービス名>` はRedisコンテナのサービス名です。
```bash
$ docker compose exec <サービス名> bash
```

コンテナ内でRedisに接続します。
```
# redis-cli
```

Redisに保存されているキー一覧を表示します。
```
> keys *
1) "laravel_database_6NDAsrurvjXJA7ovtIWVgqFpqAMHYlqy9JIP1DeW"
```

Sessionのキーの値を確認します。
```
> get laravel_database_6NDAsrurvjXJA7ovtIWVgqFpqAMHYlqy9JIP1DeW
"s:212:\"a:4:{s:6:\"_token\";s:40:\"W5bHdoKGrTO3r9xdrnqU4ZeK5muvsDyajjdpve1J\";s:8:\"test_key\";s:10:\"test_value\";s:9:\"_previous\";a:1:{s:3:\"url\";s:21:\"http://localhost:8080\";}s:6:\"_flash\";a:2:{s:3:\"old\";a:0:{}s:3:\"new\";a:0:{}}}\";"
```

結果に `s:8:\"test_key\";s:10:\"test_value\";` とあるように、Sessionの値をRedisに保存することができました。