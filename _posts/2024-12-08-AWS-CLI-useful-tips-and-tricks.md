---
title: AWS CLI useful tips & tricks
date: 2024-12-08 15:30:00 -0700
categories: [AWS, AWS CLI]
tags: [aws, aws cli]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/aws.png)

## Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

## See the account you are working on

```bash
ubuntu@ip-172-31-25-204:~$ aws sts get-caller-identity --query Account --output text
06XXXXXXXXXX
ubuntu@ip-172-31-25-204:~$
```

## Resources

- [Installing or updating to the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [Quick way to get AWS Account number from the AWS CLI tools?](https://stackoverflow.com/questions/33791069/quick-way-to-get-aws-account-number-from-the-aws-cli-tools)
