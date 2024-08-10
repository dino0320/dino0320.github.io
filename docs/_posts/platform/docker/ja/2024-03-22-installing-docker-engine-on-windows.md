---
layout: my-post
title: "WindowsでDocker Engineをインストールする"
date: 2024-03-22 00:00:00 +0000
categories: platform docker
lang: ja
---

WindowsでDocker Engineをインストールします。  
WindowsではDockerを導入する際[Docker Desktop for Windows](https://www.docker.com/ja-jp/products/docker-desktop/)を使用することが普通ですが、今回はWSLを使ってDocker Engineをインストールしてみます。

## 環境
- Windows 10 64ビット

## インストールの流れ
1. [WSLとUbuntuをインストールする。](#1-wslとubuntuをインストールする)
2. [Docker Engineをインストールする。](#2-docker-engineをインストールする)
3. [sudoなしでDockerコマンドを使えるようにする。](#3-sudoなしでdockerコマンドを使えるようにする)

以下のページを参考にしています。  
- [WSL を使用して Windows に Linux をインストールする方法](https://learn.microsoft.com/ja-jp/windows/wsl/install)
- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

## 1. WSLとUbuntuをインストールする
WSLとUbuntuのインストールについては[こちら](/platform/windows/installing-wsl)をご覧ください。  
Ubuntuを起動するWSLのバージョンが1の場合、2に変更します。  
`wsl -l` でインストールしたLinuxディストリビューションの一覧が見れます。`v` オプションを付けると、ディストリビューションを起動するWSLのバージョンを確認できます。    
コマンドプロンプトかPowerShellを開き、以下を実行します。
```
> wsl -l -v
  NAME      STATE           VERSION
* Ubuntu    Running         1
```
WSLのバージョンが1の場合、2に変更します。以下を実行します。
```
wsl --set-version Ubuntu 2
```

## 2. Docker Engineをインストールする
Docker Engineをインストールします。 
#### 1. WSLでUbuntuに接続する。  
コマンドプロンプトかPowerShellを開き、以下を実行します。
```
> wsl --distribution Ubuntu --user <UNIXユーザー名>
```
#### 2. Dockerの公式GPGキーを追加する。  
Ubuntuで以下を実行します。
```
$ sudo apt-get update
$ sudo apt-get install ca-certificates curl
$ sudo install -m 0755 -d /etc/apt/keyrings
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
$ sudo chmod a+r /etc/apt/keyrings/docker.asc
```
#### 3. aptソースにDockerリポジトリを追加する。  
Ubuntuで以下を実行します。
```
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt-get update
```
#### 4. Dockerパッケージをインストールする。
Ubuntuで以下を実行します。
```
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
#### 5. インストールの確認をする。  
インストールできたかを確認するために、Dockerのバージョンを確認します。  
Ubuntuで以下を実行します。
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
インストールの確認ができました。  

## 3. sudoなしでDockerコマンドを使えるようにする
現状ルートユーザーしかDockerコマンドを使えないので、他のユーザーもsudoなしで使えるようにします。  
#### 1. WSLでUbuntuに接続する。  
コマンドプロンプトかPowerShellを開き、以下を実行します。  
`<UNIXユーザー名>` はroot以外のユーザーにします。このユーザーがsudoなしでDockerコマンドを使えるようになります。
```
> wsl --distribution Ubuntu --user <UNIXユーザー名>
```
#### 2. dockerグループにユーザーを追加する。  
Ubuntuで以下を実行します。
```
$ sudo usermod -aG docker $USER
```
Ubuntuで以下を実行することで、dockerグループに追加されたか確認できます。
```
$ cat /etc/group | grep docker
docker:x:999:<UNIXユーザー名>
```
#### 3. Ubuntuに再接続する。  
Ubuntuで以下を実行してUbuntuからログアウトします。
```
$ exit
```
コマンドプロンプトかPowerShellを開き、以下を実行してUbuntuに接続します。
```
> wsl --distribution Ubuntu --user <UNIXユーザー名>
```
#### 4. Dockerコマンドが使えるかを確認する。  
Dockerコマンドが使えるかを確認するために、Dockerのバージョンを確認します。  
Ubuntuで以下を実行します。
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
Dockerコマンドが使えることを確認できました。