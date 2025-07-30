---
layout: my-post
title: "Configure an SSO Session and a Profile with AWS CLI"
date: 2025-07-15 00:00:00 +0000
categories: aws cli
page_name: configure-sso-session-and-profile-with-aws-cli-en
lang: en
---

This article explains how to configure an SSO session and a profile with the AWS CLI.

## Reference
- [Configuring IAM Identity Center authentication with the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)

## Prerequisite
- An AWS account
- A user in IAM Identity Center  
For more information, follow [this instruction](/aws/iam/create-user-with-administrative-access-in-iam-identity-center-en).
- AWS CLI already installed  
For more information, follow [this instruction](/aws/cli/install-aws-cli-on-windows-en)

## Environment
- Windows 10 64-bit

## Configuration Steps
1. [Check SSO Start URL and SSO Region](#1-check-sso-start-url-and-sso-region)
2. [Configure an SSO Session and a Profile](#2-configure-an-sso-session-and-a-profile)
3. [SSO Sign In and Sign Out](#3-sso-sign-in-and-sign-out)

## 1. Check SSO Start URL and SSO Region
Navigate to the AWS access portal.  
For more information, check [this article](/aws/iam/create-user-with-administrative-access-in-iam-identity-center-en).

Click "Access Keys"

![AWS access portal](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "AWS access portal")

Select "Windows".  
You can find your SSO start URL and SSO Region in the "AWS IAM Identity Center credentials (Recommended)" section.

![Credentials Screen](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "Credentials Screen")

## 2. Configure an SSO Session and a Profile
Open a command prompt and run the following command.

```
>aws configure sso
```

Fill in the following fields to configure an SSO session.

```
SSO session name (Recommended): my-sso  # Put any name
SSO start URL [None]: <Your SSO start URL>
SSO region [None]: <Your SSO Region>
SSO registration scopes [sso:account:access]: sso:account:access  # Default value is fine
```

Your default browser will open.  
Sign in using your user credentials and allow the AWS CLI access to your data.

Once you sign in, return to the command prompt.  
Then, fill in the following fields to create a profile.

| Field | Description |
|-------|-------------|
| AWS account | Select your AWS account from the displayed list. If there is only one, it will be selected automatically. |
| IAM role | Select the IAM role attached to your user. If there is only one, it will be selected automatically. |
| Default client Region | Enter the default region. This value is used when running AWS CLI commands, unless you override it. |
| CLI default output format | Choose your preferred CLI output format. See options [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html#cli-config-output). |
| Profile name | Enter any name for the profile. If you use `default`, you don't need to specify the `--profile` option when running AWS CLI commands. |

Here's an example:

```
The only AWS account available to you is: <Your AWS account ID>
Using the account ID <Your AWS account ID>
The only role available to you is: AdministratorAccess
Using the role name "AdministratorAccess"
Default client Region [None]: ap-northeast-1
CLI default output format (json if not specified) [None]: json
Profile name [AdministratorAccess-507911341149]: my-dev-profile
```

Run the following command to check whether the SSO session and the profile are working properly:  
(If you named the profile `default`, you can omit the `--profile` option.)

```
aws sts get-caller-identity --profile my-dev-profile
{
    "UserId": "<Your user ID>",
    "Account": "<Your AWS account ID>",
    "Arn": "<Your user Arn>"
}
```

If your user information is displayed, the SSO session and profile were successfully configured.

## 3. SSO Sign In and Sign Out
Next time, you can sign in using SSO with the following command:    
(If you named the profile `default`, you can omit the `--profile` option.)

```
>aws sso login --profile my-dev-profile
```

Your default browser will open.  
Sign in with your user credentials.  
Then return to the command prompt and run this command to verify you're signed in.

```
aws sts get-caller-identity --profile my-dev-profile
{
    "UserId": "<Your user ID>",
    "Account": "<Your AWS account ID>",
    "Arn": "<Your user Arn>"
}
```

If your user information is displayed, you've successfully signed in.

To sign out, run the following command:

```
>aws sso logout
```