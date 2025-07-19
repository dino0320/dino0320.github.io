---
layout: my-post
title: "Install AWS CLI on Linux"
date: 2025-07-19 00:00:00 +0000
categories: aws cli
page_name: install-aws-cli-on-linux-en
lang: en
---

This article explains how to install the AWS CLI on Ubuntu.  

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