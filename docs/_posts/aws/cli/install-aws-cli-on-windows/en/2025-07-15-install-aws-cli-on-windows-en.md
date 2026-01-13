---
layout: my-post
title: "How to Install AWS CLI v2 on Windows"
date: 2025-07-15 00:00:00 +0000
categories: aws cli
page_name: install-aws-cli-on-windows-en
lang: en
image: /assets/images/aws/cli/install-aws-cli-on-windows-en/image6.png
---

This article explains how to install the AWS CLI on Windows.  
Only 64-bit versions of Windows are supported.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "Thumbnail")

## Reference
- [Installing or updating to the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

## Prerequisite
- An AWS account

## Environment
- Windows 10 64-bit

## Installation Steps
1. [Download Installer](#1-download-installer)
2. [Verify AWS CLI](#2-verify-aws-cli)

## 1. Download Installer
Download the latest version of the installer from [here](https://awscli.amazonaws.com/AWSCLIV2.msi).  
You can check version information [here](https://raw.githubusercontent.com/aws/aws-cli/v2/CHANGELOG.rst).

Open the downloaded `.msi` file.

Click "Next".

![The first screen on the Setup Wizard](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "The first screen on the Setup Wizard")

Read the license agreement, then check the "I accept the terms on the License Agreement" check box.  
Click "Next".

![The license agreement screen on the Setup Wizard](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "The license agreement screen on the Setup Wizard")

Click "Next".

![The custom setup screen on the Setup Wizard](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "The custom setup screen on the Setup Wizard")

Click "Install".

![The confirmation screen on the Setup Wizard](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "The confirmation screen on the Setup Wizard")

Click "Finish".

![The final screen on the Setup Wizard](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "The final screen on the Setup Wizard")

## 2. Verify AWS CLI
Verify that the AWS CLI was successfully installed.

Open the Windows Start Menu and type `cmd`.  
Click the Command Prompt result at the top.

Run the following command to check the AWS CLI version.

```
>aws --version
aws-cli/2.27.50 Python/3.13.4 Windows/10 exe/AMD64
```

If the version was displayed, the AWS CLI was installed successfully.

To start using the AWS CLI, follow [this article](/aws/cli/configure-sso-session-and-profile-with-aws-cli-en).