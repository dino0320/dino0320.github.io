---
layout: my-post
title: "Installing SQLite on Amazon Linux"
date: 2025-05-29 00:00:00 +0000
categories: database sqlite
page_name: installing-sqlite-on-amazon-linux-en
lang: en
---

Install SQLite in a Docker Container Based on Amazon Linux 2023.

## Environment
- Windows 10 64-bit  
- WSL2 + Ubuntu 22.04.3 LTS  
- Docker Engine 26.0.0  

## Installation Flow
1. [Install SQLite](#1-install-sqlite)  
2. [Verify the Installation](#2-verify-the-installation)  

## 1. Install SQLite  
Create a `Dockerfile` for Amazon Linux 2023 and add a line to install SQLite.  
We’ll install version `3.40.0`.

`Dockerfile`:
```dockerfile
FROM amazonlinux:2023

# Install SQLite
RUN yum -y install sqlite-3.40.0
```

## 2. Verify the Installation  
Check whether SQLite is installed.

Run the following commands in the same directory as the `Dockerfile`:

```bash
$ docker build . -t installed_sqlite
$ docker run -it --rm --name sqlite_verification installed_sqlite bash
```

These commands build a Docker image named `installed_sqlite` and launch a container named `sqlite_verification`.

Inside the container, run:

```
# sqlite3 test.sqlite    # Create test.sqlite database
sqlite> create table `test` (`id`, `name`);    -- Create a table named test
sqlite> insert into test (id, name) values (1, 'Test');    -- Insert a record
sqlite> select * from test;    -- Check the records in the test table
1|Test
```

If you can perform the operations above, SQLite has been installed successfully.