---
layout: my-post
title: "How to Install AWS CLI v2 on Ubuntu"
date: 2025-07-19 00:00:00 +0000
categories: aws cli
page_name: install-aws-cli-on-linux-en
lang: en
image: /assets/images/aws/cli/install-aws-cli-on-linux-en/image1.png
---

This article explains how to install the AWS CLI on Ubuntu.  

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## Reference
- [Installing or updating to the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

## Environment
- Ubuntu 22.04.3 LTS

To install the AWS CLI, the `unzip` utility is required.  
If it's not already installed in your environment, run the following command:

```bash
$ sudo apt install unzip
```

Then, install the AWS CLI with the following commands:

```bash
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ unzip awscliv2.zip
$ sudo ./aws/install
```

To verify the AWS CLI installation, run:

```bash
aws --version
aws-cli/2.27.55 Python/3.13.4 Linux/6.6.87.2-microsoft-standard-WSL2 exe/x86_64.ubuntu.22
```

If if the AWS CLI version is displayed, the installation was successful.