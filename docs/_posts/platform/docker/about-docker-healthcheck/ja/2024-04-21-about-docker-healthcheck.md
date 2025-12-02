---
layout: my-post
title: "Dockerのhealthcheckについて"
date: 2024-04-21 00:00:00 +0000
categories: platform docker
page_name: about-docker-healthcheck
lang: ja
image: /assets/images/platform/docker/about-docker-healthcheck/image1.png
---

Dockerのhealthcheckについて調査しました。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 環境
- Docker Engine 26.0.0
- Docker Compose version v2.25.0

## healthcheckとは
DockerのhealthcheckはDockerコンテナがhealthy(正常)かどうかをチェックするために定義します。  
具体的には、指定したコマンドを一定間隔で実行し、成功したらDockerコンテナはhealthyと判断されます。一定回数連続で失敗すると、unhealthyになります。(Dockerコンテナの初期状態はstarting)

healthcheckは `Dockerfile` または `docker-compose.yml` で定義できます。  
`docker-compose.yml` で定義したhealthcheckは `Dockerfile` で定義したものを上書きします。

`docker-compose.yml` で定義できるhealthcheckの項目は以下の通りです。  

|項目|説明|
|----|----|
|test|コンテナの状態をチェック(ヘルスチェック)するために実行されるコマンドです。コマンドは終了ステータスが0の場合成功、1の場合失敗とみなされます。|
|interval|ヘルスチェックを実行する間隔です。|
|timeout|一回のヘルスチェックのタイムアウト時間です。ヘルスチェックの実行時間がこの値を超えると失敗になります。|
|retries|ヘルスチェックのリトライ回数です。ヘルスチェックが連続でこの回数分失敗すると、コンテナはunhealthyとみなされます。|
|start_period|コンテナのブートストラップのための初期化時間です。この時間中はヘルスチェックが何回失敗してもunhealthyになりません。この時間中にヘルスチェックが成功すると、初期化時間は終了します。|
|start_interval|start_periodで定義された初期化時間の間、ヘルスチェックを実行する間隔です。|

以下はhealthcheckの設定例です。
```yml
healthcheck:
  test: curl -f https://localhost || exit 1
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
  start_interval: 5s
```
上記の例を説明すると、  
`curl -f https://localhost || exit 1` がヘルスチェックのコマンドです。  
40秒の初期化時間の間、5秒間隔でヘルスチェックが実行されます。  
初期化時間の終了後、1分30秒の間隔でヘルスチェックが実行されます。  
1回のヘルスチェックは10秒でタイムアウトします。  
ヘルスチェックが連続で3回失敗すると、コンテナはunhealthyとみなされます。