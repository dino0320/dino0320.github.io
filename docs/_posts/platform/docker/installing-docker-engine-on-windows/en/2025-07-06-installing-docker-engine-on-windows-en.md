---
layout: my-post
title: "Installing Docker Engine on Windows"
date: 2025-07-06 00:00:00 +0000
categories: platform docker
lang: en
---

This guide explains how to install Docker Engine on Windows.  
Although using [Docker Desktop for Windows](https://www.docker.com/ja-jp/products/docker-desktop/) is the standard method for installing Docker on Windows, here we’ll install Docker Engine using WSL (Windows Subsystem for Linux).

## References  
- [WSL を使用して Windows に Linux をインストールする方法](https://learn.microsoft.com/ja-jp/windows/wsl/install)
- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

## Environment
- Windows 10 64-bit

## Installation Steps
1. [Install WSL and Ubuntu](#1-install-wsl-and-ubuntu)
2. [Install Docker Engine](#2-install-docker-engine)
3. [Enable Docker Commands Without sudo](#3-enable-docker-commands-without-sudo)

## 1. Install WSL and Ubuntu
For details on installing WSL and Ubuntu, see [this guide](/platform/windows/installing-wsl-en).

If the WSL version for Ubuntu is 1, you’ll need to upgrade it to version 2.

To check the WSL version of your installed Linux distributions, open Command Prompt or PowerShell and run:

```
> wsl -l -v
  NAME      STATE           VERSION
* Ubuntu    Running         1
```

If the version is 1, change it to 2 with the following command:

```
wsl --set-version Ubuntu 2
```

## 2. Install Docker Engine
Now let’s install Docker Engine on Ubuntu inside WSL.

#### 1. Connect to Ubuntu via WSL
Open Command Prompt or PowerShell and run:

```
> wsl --distribution Ubuntu --user <UNIX-username>
```

#### 2. Add Docker’s official GPG key 
In Ubuntu, run:

```
$ sudo apt-get update
$ sudo apt-get install ca-certificates curl
$ sudo install -m 0755 -d /etc/apt/keyrings
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
$ sudo chmod a+r /etc/apt/keyrings/docker.asc
```

#### 3. Add the Docker repository to APT sources
Still in Ubuntu, run:

```
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt-get update
```

#### 4. Install Docker packages
Run the following:

```
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### 5. Verify the installation
Check that Docker was installed correctly by verifying the version:

```
$ sudo docker version
Client: Docker Engine - Community
 Version:           26.0.0
 API version:       1.45
 Go version:        go1.21.8
 Git commit:        2ae903e
 Built:             Wed Mar 20 15:17:48 2024
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          26.0.0
  API version:      1.45 (minimum version 1.24)
  Go version:       go1.21.8
  Git commit:       8b79278
  Built:            Wed Mar 20 15:17:48 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.28
  GitCommit:        ae07eda36dd25f8a1b98dfbf587313b99c0190bb
 runc:
  Version:          1.1.12
  GitCommit:        v1.1.12-0-g51d5e94
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

## 3. Enable Docker Commands Without sudo
By default, only the root user can run Docker commands.  
To allow your user to run Docker without sudo, follow these steps:

#### 1. Connect to Ubuntu via WSL
Run this from Command Prompt or PowerShell (use a non-root user):

```
> wsl --distribution Ubuntu --user <UNIX-username>
```

#### 2. Add your user to the docker group
In Ubuntu, run:

```
$ sudo usermod -aG docker $USER
```

To confirm your user was added to the docker group:

```
$ cat /etc/group | grep docker
docker:x:999:<UNIX-username>
```

#### 3. Reconnect to Ubuntu
First, exit Ubuntu:

```
$ exit
```

Then reconnect:

```
> wsl --distribution Ubuntu --user <UNIX-username>
```

#### 4. Verify Docker can run without sudo
Run:

```
$ docker version
Client: Docker Engine - Community
 Version:           26.0.0
 API version:       1.45
 Go version:        go1.21.8
 Git commit:        2ae903e
 Built:             Wed Mar 20 15:17:48 2024
 OS/Arch:           linux/amd64
 Context:           default
 
Server: Docker Engine - Community
 Engine:
  Version:          26.0.0
  API version:      1.45 (minimum version 1.24)
  Go version:       go1.21.8
  Git commit:       8b79278
  Built:            Wed Mar 20 15:17:48 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.28
  GitCommit:        ae07eda36dd25f8a1b98dfbf587313b99c0190bb
 runc:
  Version:          1.1.12
  GitCommit:        v1.1.12-0-g51d5e94
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```
If you see the version details without any permission errors, Docker is now usable without sudo.