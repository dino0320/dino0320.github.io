---
layout: my-post
title: "Configure the Default User in WSL"
date: 2025-10-07 00:00:00 +0000
categories: platform windows
page_name: configure-wsl-default-user-en
lang: en
---

This article explains how to configure the default user for WSL.

## Environment
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2 distribution)

Connect to your WSL distribution and edit the `/etc/wsl.conf` file as follows:

```conf
[user]
default=<your desired username>
```

Then, restart WSL. The default user should now be updated.

```bash
wsl --shutdown
wsl --distribution Ubuntu   # You should now be connected as the updated default user.
```