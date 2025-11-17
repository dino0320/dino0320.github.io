---
layout: my-post
title: "Installing Mysql Client on Amazon Linux"
date: 2025-05-29 00:00:00 +0000
categories: database mysql
page_name: installing-mysql-client-on-amazon-linux-en
lang: en
image: /assets/images/database/mysql/installing-mysql-client-on-amazon-linux-en/image1.png
---

Install the MySQL client in a Docker Container Based on Amazon Linux 2023.  
We'll install the MySQL client in a Docker container created from the Amazon Linux 2023 Docker image.  
This article is based on [this page](https://dev.classmethod.jp/articles/install-mysql-client-to-amazon-linux-2023/).

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## What is the MySQL Client?  
It's a tool that allows you to connect to and interact with a MySQL server.

## Environment  
- Ubuntu 22.04.3 LTS (running on WSL)  
- Docker Engine 26.0.0  

## Installation Flow  
1. [Install the MySQL Client](#1-install-the-mysql-client)  
2. [Verify the Installation](#2-verify-the-installation)  

## 1. Install the MySQL Client  
Create a `Dockerfile` for Amazon Linux 2023 and add commands to install the MySQL client.

`Dockerfile`:
```dockerfile
FROM amazonlinux:2023

# Install MySQL client
RUN rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
RUN yum -y install https://repo.mysql.com/mysql80-community-release-el9-1.noarch.rpm
RUN yum -y install mysql-community-client-8.0.34
```

We're adding a YUM repository intended for EL9-based systems.  
According to [the reference site](https://dev.classmethod.jp/articles/install-mysql-client-to-amazon-linux-2023/), it's not possible to use Fedora packages (even though Amazon Linux 2023 includes components from Fedora 34, 35, and 36).

Note: The GPG key may have an expiration date. If you're reading this at a later time, the key might not work anymore.  
You can find the latest GPG keys and YUM repositories at [repo.mysql.com](https://repo.mysql.com).

In this case, we're targeting future deployment to AWS and plan to use Amazon Aurora, which is compatible with MySQL 8.0.34.  
Therefore, we install version 8.0.34 of the MySQL client to match the server version.  
(It's unclear whether matching versions is strictly necessary, but it's generally recommended.)

## 2. Verify the Installation  
We'll test the installed MySQL client.

Create a `docker-compose.yml` file in the same directory as the above `Dockerfile`.

`docker-compose.yml`:
```yaml
services:
  mysql_client:
    build: .
    tty: true    # Prevent the Docker container from exiting immediately
  mysql_server:
    image: mysql:8.0.34
    environment:
      MYSQL_ROOT_PASSWORD: root
```

In the same directory, run the following commands to start the containers and access the MySQL client container (based on Amazon Linux 2023):

```bash
$ docker compose up -d
$ docker compose exec mysql_client bash
```

Inside the container, run:
```
# mysql -u root -p -h mysql_server
```

When prompted for a password, enter `root`, which is the value set in `MYSQL_ROOT_PASSWORD` in the `docker-compose.yml`.

If the prompt changes to `mysql>`, the connection was successful.  
This confirms that the MySQL client was installed correctly.