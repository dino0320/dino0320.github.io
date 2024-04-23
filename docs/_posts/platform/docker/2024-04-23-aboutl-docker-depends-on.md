---
layout: my-post
title: "Dockerのdepends onについて"
date: 2024-04-23 00:00:00 +0000
categories: platform docker
title_eng: about-docker-depends-on
---

Dockerの `depends on` について調査しました。

## 環境
- Docker Engine 26.0.0
- Docker Compose version v2.25.0

## depends onとは
Dockerの `depends on` はサービス間の起動・停止に関する依存関係を定義します。  
`depends on` は `docker-compose.yml` で定義できます。

例えばWebサーバー用のDockerコンテナが起動時にDB用のコンテナに対して処理を行う必要がある場合、DB用のコンテナはWebサーバー用のコンテナより早く起動している必要があります。  
そのような場合、`depends on` を定義することでコンテナの起動順を指定できます。

`depends on` にはShort syntaxとLong syntaxの2種類の書き方があります。

### Short syntax
Short syntaxは、シンプルに依存するサービス名だけを記述します。  

例えば以下のように定義します。例は[公式ページ](https://docs.docker.com/compose/compose-file/05-services/#depends_on)からの引用です。
```yml
services:
  web:
    build: .
    depends_on: # ここにdepends onを定義
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```
上記の例では、サービス `web` はサービス `db` 、`redis` の起動後に起動し、サービス `db` 、`redis` の停止前に停止します。

Short syntaxの場合、サービスは依存するサービスの起動後に起動しますが、準備が整うまで待ってくれるわけではありません。  
上記の例では、サービス `db` はサービス `web` の前に起動しますが、Postgresの準備が整ってから起動するわけではありません。つまりサービス `web` が起動時にサービス `db` のPostgresに対して処理を行う場合、Postgresが起動していなければ失敗する可能性があります。

### Long syntax
Long syntaxは、以下の3つの項目を追記することで、Short syntaxよりいろいろな設定を定義できます。

|項目|説明|
|----|----|
|restart|依存するサービスが更新されたらこのサービスを再起動するかどうかです。trueにすると再起動します。デフォルトはfalseです。例えば `docker compose restart` で依存するサービスを再起動した場合、このサービスも再起動します。|
|condition|依存するサービスの状態です。依存するサービスが指定した状態になったらこのサービスを起動します。依存するサービスの具体的な状態については後ほど記載します。|
|required|依存するサービスが起動する必要があるかどうかです。falseにすると、依存するサービスが起動に失敗しても警告を出すだけで、このサービスは起動します。デフォルトはtrueで、依存するサービスが起動に失敗したらこのサービスも起動しません。|

項目 `condition` について、指定できる依存するサービスの状態は以下の3つです。

|依存するサービスの状態|説明|
|----|----|
|service_started|依存するサービスの起動後にこのサービスを起動します。[Short syntax](#short-syntax)と同じ動きになります。|
|service_healthy|依存するサービスのヘルスチェックが成功したらこのサービスを起動します。Dockerのヘルスチェックについては[こちら](/platform/docker/about-docker-healthcheck)をご覧ください。|
|service_completed_successfully|依存するサービスが正常に終了したらこのサービスを起動します。|

例えば以下のように定義します。例は[公式ページ](https://docs.docker.com/compose/compose-file/05-services/#depends_on)からの引用です。
```yml
services:
  web:
    build: .
    depends_on: # ここにdepends onを定義
      db:
        condition: service_healthy
        restart: true
      redis:
        condition: service_started
  redis:
    image: redis
  db:
    image: postgres
```
上記の例では、  
サービス `web` はサービス `db` のヘルスチェックが成功したら起動します。また、サービス `web` はサービス `db` が更新されたら再起動します。  
サービス `web` はサービス `redis` の起動後に起動します。  
サービス `web` はサービス `db` 、`redis` の停止前に停止します。