---
layout: my-post
title: "Understanding WSL User IDs and File Permission Issues with Docker"
date: 2025-07-06 00:00:00 +0000
categories: platform windows
page_name: about-user-id-of-wsl-en
lang: en
image: /assets/images/platform/windows/about-user-id-of-wsl-en/image1.png
---

I looked into the User ID (uid) in WSL.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## Reference
- [Advanced settings configuration in WSL](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)

## Environment
- Windows 10 64-bit
- Ubuntu 22.04.3 LTS (WSL2 distribution)

## WSL User ID
The default user in WSL has both a User ID (uid) and Group ID (gid) of `1000`.  
You can confirm this on Ubuntu like below (assuming your WSL default username is `username`):

```bash
cat /etc/passwd
...
username:x:1000:1000:,,,:/home/username:/bin/bash
```

## Why I wrote this article
While developing a Laravel application using Docker on WSL, I ran into an issue where files created with `php artisan make:...` couldn't be edited in VSCode running on WSL.

The reason is that when files are created inside the Docker container, they are owned by the root user, resulting in file permissions like `-rw-r--r-- 1 root root`.  
However, in VSCode, files are edited as the WSL default user (not root), so write access is denied.

To resolve this, I figured I could connect to the Docker container as the WSL default user.
So, I tried accessing the container as the WSL default user using its UID (`1000`):

```bash
$ docker compose exec --user 1000 web bash
```

Then I created a file and checked its permissions:

```bash
$ php artisan make:command SendEmails

 INFO  Console command [app/Console/Commands/SendEmails.php] created successfully.

$ ls -l app/Console/Commands/SendEmails.php
-rw-r--r-- 1 1000 root ...
```

The file permissions are now `-rw-r--r-- 1 1000 root`.
After exiting the container and checking from WSL (with `username` as the default user):

```bash
$ ls -l app/Console/Commands/SendEmails.php
-rw-r--r-- 1 username root ...
```

This confirms that the file is now writable by the WSL default user.  
Now I can edit it in VSCode. (Though there may be better ways to handle this.)