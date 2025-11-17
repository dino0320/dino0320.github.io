---
layout: my-post
title: "よく使うDockerコマンドについて"
date: 2024-03-30 00:00:00 +0000
categories: platform docker
page_name: about-docker-commands
lang: ja
image: /assets/images/platform/docker/about-docker-commands/image1.png
---

よく使うDockerコマンドについてまとめます。  
個人的によく使うコマンドとオプションだけ記載しています。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 環境
- Ubuntu 22.04.3 LTS (WSLで起動している)
- Docker Engine 26.0.0

## Dockerコマンド一覧
- [docker ps](#docker-ps)
- [docker rm](#docker-rm)
- [docker images](#docker-images)
- [docker rmi](#docker-rmi)
- [docker build](#docker-build)
- [docker run](#docker-run)
- [docker stop](#docker-stop)
- [docker compose up](#docker-compose-up)
- [docker compose exec](#docker-compose-exec)
- [docker compose down](#docker-compose-down)

## docker ps
`docker ps` コマンドは作成したDockerコンテナ一覧を表示するコマンドです。

使い方:  
```
docker ps [OPTIONS]
```

使用例:  
```
docker ps -a
```

オプション:  

|オプション|説明|
|----|----|
|a|すべてのDockerコンテナを表示します。このオプションを付けない場合、起動中のコンテナのみ表示します。|

## docker rm
`docker rm` コマンドは指定したDockerコンテナを削除するコマンドです。

使い方:  
```
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

使用例:  
```
docker rm <コンテナID1> <コンテナID2>
```

引数:  

|引数|説明|
|----|----|
|CONTAINER|コンテナIDを指定します。複数指定できます。コンテナIDは `docker ps` コマンドで確認できます。|

オプション:  

|オプション|説明|
|----|----|
|f|Dockerコンテナを強制的に削除します。起動中のコンテナも削除できます。|

## docker images
`docker images` コマンドはビルド、ダウンロードしたDockerイメージ一覧を表示するコマンドです。

使い方:  
```
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

使用例:  
```
docker images
```

## docker rmi
`docker rmi` コマンドは指定したDockerイメージを削除するコマンドです。

使い方:  
```
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

使用例1:  
```
docker rmi <イメージID1> <イメージID2>
```

使用例2:  
```
docker rmi php:8.2
```

引数:  

|引数|説明|
|----|----|
|IMAGE|イメージIDを指定します。複数指定できます。イメージIDは `docker images` コマンドで確認できます。同じイメージIDの複数のタグがある場合、イメージIDの代わりに `<リポジトリ名>:<タグ名>` を指定することでタグを削除できます。使用例2では `php:8.2` を指定しています。|

## docker build
`docker build` コマンドは `Dockerfile` からDockerイメージを構築するコマンドです。

使い方:  
```
docker build [OPTIONS] PATH | URL
```

使用例:  
```
docker build -t test:1.0 .
```

引数:

|引数|説明|
|----|----|
|PATH|`Dockerfile` のあるディレクトリのパスを指定します。 使用例では `.` を指定しています。|
|URL|リモートの `Dockerfile` のURLをしてします。|

オプション: 

|オプション|説明|
|----|----|
|t|Dockerイメージにタグを割り当てます。形式は `名前:タグ` です。使用例では `test:1.0` を指定しています。|

## docker run
`docker run` コマンドはDockerイメージからDockerコンテナを起動するコマンドです。  

使い方:  
```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

使用例:  
```
docker run -it --rm --name php-cli -w /app -v `pwd`:/app php:8.2-cli bash
```

引数:  

|引数|説明|
|----|----|
|IMAGE|Dockerイメージを指定します。使用例では `php:8.2-cli` を指定しています。|
|COMMAND|Dockerコンテナ起動後に実行するコマンドを指定できます。使用例では `bash` コマンドを指定し、コンテナに接続するようにしています。|

オプション:  

|オプション|説明|
|----|----|
|i|標準入力を開き続けます。Dockerコンテナへの入力ができるようになります。|
|t|疑似TTYを割り当てます。TTYはユーザーの入出力デバイスで、Dockerコンテナに接続したときに入力を受け付けたり出力を表示したりできるようになります。また、TTYのプロセスが動くことで、コンテナ接続中はコンテナが終了することを防ぎます。|
|rm|Dockerコンテナが終了するときにコンテナを自動的に削除します。|
|name|Dockerコンテナに名前を付けます。使用例では `php-cli` を指定しています。|
|w|Dockerコンテナ内のワーキングディレクトリです。使用例では `/app` ディレクトリを指定しています。|
|v|Dockerコンテナにホストのディレクトリをマウントします。使用例では `pwd` コマンドでホストの現在のディレクトリを出力し、コンテナ内の `/app` ディレクトリにマウントしています。|
|p|Dockerコンテナのポートにホストのポートを転送します。例: `-p 8080:80` (`8080` はホストのポート、`80` はコンテナのポート)|

## docker stop
`docker stop` コマンドは起動中のDockerコンテナを停止するコマンドです。

使い方:  
```
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

使用例:  
```
docker stop <コンテナID>
```

引数:  

|引数|説明|
|----|----|
|CONTAINER|コンテナIDを指定します。複数指定できます。コンテナIDは `docker ps` コマンドで確認できます。|

## docker compose up
`docker compose up` コマンドはdocker-compose.ymlで定義した複数のDockerコンテナを起動するコマンドです。

使い方:  
```
docker compose up [OPTIONS] [SERVICE...]
```

使用例1:  
```
docker compose up -d
```

使用例2:  
```
docker compose up --build
```

オプション:  

|オプション|説明|
|----|----|
|d|Dockerコンテナをバックグラウンドで起動します。|
|build|Dockerコンテナを起動する前にDockerイメージをビルドします。|

## docker compose exec
`docker compose exec` コマンドはdocker-compose.ymlで定義した起動中のDockerコンテナでコマンドを実行するコマンドです。このコマンドはデフォルトで疑似TTYを割り当てます。

使い方:  
```
docker compose exec [OPTIONS] SERVICE COMMAND [ARGS...]
```

使用例:  
```
docker compose exec <サービス名> bash
```

引数:  

|引数|説明|
|----|----|
|COMMAND|Dockerコンテナで実行するコマンドを指定できます。使用例では `bash` コマンドを指定し、コンテナに接続するようにしています。|

## docker compose down
`docker compose down` コマンドはdocker-compose.ymlで定義した起動中の複数のDockerコンテナを停止するコマンドです。

使い方:  
```
docker compose down [OPTIONS] [SERVICES]
```

使用例1:  
```
docker compose down
```

使用例2:  
```
docker compose down -v
```

オプション:  

|オプション|説明|
|----|----|
|v|docker-compose.ymlで定義したDockerボリュームを削除します。|