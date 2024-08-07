---
title: Setup user profile in AWS CLI
date: 2024-06-22 20:32:00 -0700
categories: [AWS, CLI]
tags: [aws, cli]     # TAG names should always be lowercase
---

**IMPORTANT**
*The outputs shown here were taken from a temporary AWS account which has now been deleted*

# Install AWS CLI

Refer to official AWS documentation [Install or update to the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

# Obtain Access Key and Secret Access Key

1. Go to IAM service in the AWS Management Console

![]({{ site.baseurl }}/images/services/iam.png)

2. Select a specific user and go to the **Security credentials** tab

![]({{ site.baseurl }}/images/2024/06-22-Using-AWS-CLI/01-IAM-User-Security-credentials.png)

3. Create the Access Keys

a)

![]({{ site.baseurl }}/images/2024/06-22-Using-AWS-CLI/02-Create-access-keys.png)

b)

![]({{ site.baseurl }}/images/2024/06-22-Using-AWS-CLI/03-Create-access-keys.png)

c)

![]({{ site.baseurl }}/images/2024/06-22-Using-AWS-CLI/04-Create-access-keys.png)

4. Set the credentials in the AWS CLI

```bash
$ aws configure --profile cloudUser
AWS Access Key ID [None]: AKIAUOQPBICVZ3MTVB2F
AWS Secret Access Key [None]: <OMMITED_FOR_SECURITY_REASONS>
Default region name [None]:
Default output format [None]:
$
```

5. Verification

```bash
$ aws sts get-caller-identity --profile cloudUser
{
    "UserId": "AIDAUOQPBICVW2YZ64DKF",
    "Account": "306047959211",
    "Arn": "arn:aws:iam::306047959211:user/cloud_user"
}
$
```