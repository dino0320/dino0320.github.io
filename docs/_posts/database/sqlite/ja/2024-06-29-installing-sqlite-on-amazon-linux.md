---
layout: my-post
title: "Amazon Linux 2023にSQLiteをインストールする"
date: 2024-06-29 00:00:00 +0000
categories: database sqlite
title_eng: installing-sqlite-on-amazon-linux
---

Amazon Linux 2023のDockerイメージから作成したDockerコンテナにSQLiteをインストールします。  

## 環境
- Windows 10 64ビット
- WSL2 + Ubuntu 22.04.3 LTS
- Docker Engine 26.0.0

## インストールの流れ
1. [SQLiteをインストールする。](#1-sqliteをインストールする)
2. [インストールを確認する。](#2-インストールを確認する)

## 1. SQLiteをインストールする
Amazon Linux 2023の `Dockerfile` を作成し、SQLiteインストールの行を追加します。  
バージョンに `3.40.0` を指定してインストールしています。

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

# SQLiteインストール
RUN yum -y install sqlite-3.40.0
```

## 2. インストールを確認する
SQLiteがインストールされているか確認します。

`Dockerfile` と同じ場所で以下を実行します。

```bash
$ docker build . -t installed_sqlite
$ docker run -it --rm --name sqlite_verification installed_sqlite bash
```

上記のコマンドで、`installed_sqlite` という名前のDockerイメージをビルドし、`sqlite_verification` という名前のDockerコンテナを起動して接続しています。

コンテナ内で以下を実行します。

```
# sqlite3 test.sqlite    # test.sqliteデータベース作成
sqlite> create table `test` (`id`, `name`);    -- testテーブル作成
sqlite> insert into test (id, name) values (1, 'Test');    -- testテーブルにレコード挿入
sqlite> select * from test;    -- testテーブルのレコード確認
1|Test
```

SQLiteの操作ができ、インストールされていることを確認できました。