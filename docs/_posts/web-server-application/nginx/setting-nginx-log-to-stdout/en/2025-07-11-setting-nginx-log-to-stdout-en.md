---
layout: my-post
title: "Configure NGINX to Output Logs to Standard Output"
date: 2025-07-11 00:00:00 +0000
categories: web-server-application nginx
page_name: setting-nginx-log-to-stdout-en
lang: en
---

We’ll configure NGINX to output access logs to standard output and error logs to standard error.

In this setup, we’ll configure NGINX running inside a Docker container.  
Instead of using the official NGINX Docker image, we’re using an Amazon Linux 2023 image with NGINX installed.

## Reference
- [コンテナ内のプロセスのログ出力先を標準出力/標準エラー出力に設定する方法 - Qiita](https://qiita.com/sshota0809/items/a86cd3379f88fb5cd1b8)

## Prerequisites
- A Docker container with NGINX installed.  
For how to run NGINX in a Docker container on WSL, refer to [this guide](/web-application-framework/laravel/running-laravel-project-on-nginx-en).

## Environment
- Windows 10 64-bit
- WSL2 + Ubuntu 22.04.3 LTS
- Docker Engine 26.0.0
- Amazon Linux 2023(Docker container OS)
- NGINX 1.24.0

## Configuration Steps
1. [Check the log file locations](#1-check-the-log-file-locations)
2. [Create symbolic links to standard output](#2-create-symbolic-links-to-standard-output)
3. [Verify the configuration](#3-verify-the-configuration)

## 1. Check the log file locations
First, check where NGINX is outputting access and error logs.

Open the `nginx.conf` configuration file and check the `access_log` and `error_log` directives.  
On Amazon Linux 2023, `nginx.conf` is located in the `/etc/nginx` directory by default.

Here’s what the `access_log` and `error_log` settings look like:

```
access_log  /var/log/nginx/access.log  main;
error_log /var/log/nginx/error.log notice;
```

The access log is output to `/var/log/nginx/access.log`, and the error log to `/var/log/nginx/error.log`.  
The `main` after `access_log` refers to the log format defined in the `log_format` directive within `nginx.conf`.  
The `notice` after `error_log` specifies the minimum log level.   Logs at `notice` level and above will be output.

With that, we’ve confirmed the locations of the log files.

## 2. Create symbolic links to standard output
To output logs to standard output, we’ll create symbolic links from the log files to standard output and error output.

We’ll link the access log to standard output and the error log to standard error.  
Modify the Dockerfile used to build the Docker image that installs NGINX as follows:

`Dockerfile` :
```dockerfile
FROM amazonlinux:2023

... Install NGINX steps

# Create symbolic links with the log file names
RUN ln -s /dev/stdout /var/log/nginx/access.log
RUN ln -s /dev/stderr /var/log/nginx/error.log
```

Now the log files are symbolic links to standard output and error output.

## 3. Verify the configuration
Let’s confirm that logs are being output to standard output.

Start the Docker container with NGINX installed, and access the NGINX server from a browser or similar tool.  
Then check the logs from the Docker container by running the following:

```bash
$ docker logs <container_id>
```

If you see the NGINX access logs output like below, the configuration was successful:

```
192.168.224.1 - - [20/Apr/2024:05:50:47 +0000] "GET / HTTP/1.1" 200 34063 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36" "-"
192.168.224.1 - - [20/Apr/2024:05:50:47 +0000] "GET / HTTP/1.1" 200 34063 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36" "-"
```