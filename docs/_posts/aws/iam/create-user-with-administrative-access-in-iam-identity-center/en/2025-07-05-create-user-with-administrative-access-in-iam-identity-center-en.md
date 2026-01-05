---
layout: my-post
title: "How to Create an Administrator User in AWS IAM Identity Center"
date: 2025-07-05 00:00:00 +0000
categories: aws iam
page_name: create-user-with-administrative-access-in-iam-identity-center-en
lang: en
image: /assets/images/aws/iam/create-user-with-administrative-access-in-iam-identity-center-en/image10.png
---

This is an article how to create a user with administrative access in IAM Identity Center using AWS Management Console.  
You shouldn't use your AWS account root user for every tasks.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image10.png "Thumbnail")

## Reference
- [Set up to use Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/get-set-up-for-amazon-ecs.html)

## Prerequisite
- Have an AWS account root user

## Workflow
1. [Create New User](#1-create-new-user)
2. [Create New Permission Set](#2-create-new-permission-set)
3. [Assign Permission Set to User](#3-assign-permission-set-to-user)
4. [Access AWS Management Console](#4-access-aws-management-console)

## 1. Create New User
Sign in AWS Management Console using your AWS account root user email.

![Sign in form](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Sign in form")

![Sign in form using root user](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "Sign in form using root user")

Go to IAM Identity Center and enable it.

![How to access IAM Identity Center](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "How to access IAM Identity Center")

Click "Users" in IAM Identity Center.

![Users in IAM Identity Center](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image4.png "Users in IAM Identity Center")

Click "Add User" and fill in Primary Information. Other fields are optional.  
I selected "Send an email to this user with password setup instructions." this time.

![User Primary Information](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image5.png "User Primary Information")

Click "Next" then you can add the user to groups optionally.  
Click "Add user" to complete the process.

The user will receive an email invitation.  
Click "Accept Invitation".  
Set a password and turn on multi-factor authentication(MFA).  
I chose "Authenticator apps" and used Google Authenticator.  
After setup, you can access AWS access portal.  
Use your AWS access portal URL in the email to access it next time onwards.

## 2. Create New Permission Set
To grant administrative access, create a permission set.  
Return to IAM Identity Center as your AWS account root user and click "Permission sets".

![Permission sets in IAM Identity Center](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "Permission sets in IAM Identity Center")

Click "Create permission set".  
Select "Predefined permission set" and "AdministratorAccess".

![Select permission set type](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image7.png "Select permission set type")

Other fields are optional. Click "Create" to finish.

## 3. Assign Permission Set to User
Click "AWS accounts" in IAM Identity Center.

![AWS accounts in IAM Identity Center](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image6.png "AWS accounts in IAM Identity Center")

Select your AWS account root user and click "Assign users or groups".  
Click the "Users" tab and select the user that you created, then click "Next".  
Select the AdministratorAccess permission set, then click "Next".  
Click "Submit". The user is assigned to your AWS account root user with administrative access.

## 4. Access AWS Management Console
Verify to access AWS Management Console using the user with administrative access.

Access AWS access portal using the link in the email which you received when you generated the user.

Click "AdministratorAccess", then you can access AWS Management Console.

![AWS access portal](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image9.png "AWS access portal")
