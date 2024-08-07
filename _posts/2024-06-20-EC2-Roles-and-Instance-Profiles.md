---
title: How to attach IAM Instance Profile to EC2 instance
date: 2024-06-20 20:42:00 -0700
categories: [AWS, IAM, EC2]
tags: [aws, iam, ec2]     # TAG names should always be lowercase
---

# Introduction

**IMPORTANT**
*The outputs shown here were taken from a temporary AWS account which has now been deleted*

**Note**
*The words "associate" and "attach" are used interchangeably*

In this guide we are going to:

- Create a new IAM Policy which has read access to a specific S3 bucket
- Create an IAM Role and associate the IAM Policy to it
- Create an IAM Instance Profile and associate the IAM Role to it
- Associate the IAM Isntance Profile to an EC2 instance

Amazon EC2 instances should be able to securely access other AWS services. Credentials need to be rotated regularly to minimize the adverse impact of a security breach.

- **AWS IAM roles** provide the ability to automatically grant instances temporary credentials without the need for manual management.
- **IAM instance profiles** provide the mechanism to attach IAM roles to EC2 instances.

- **Trust Policy**: Establishes trust, answering who can assume the role.

- **Permissions Policy**: Defines access levels, detailing what actions a principal can or cannot do on AWS resources.

# Configuration via AWS CLI

## Pre-requisites

Trust Policy file: `trust_policy_ec2.json`

<details markdown=1><summary markdown="span">trust_policy_ec2.json</summary>

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {"Service": "ec2.amazonaws.com"},
            "Action": "sts:AssumeRole"
        }
    ]
}
```

</details><br />

IAM Policy file: `dev_s3_read_access.json`

<details markdown=1><summary markdown="span">dev_s3_read_access.json</summary>

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowUserToSeeBucketListInTheConsole",
            "Action": ["s3:ListAllMyBuckets", "s3:GetBucketLocation"],
            "Effect": "Allow",
            "Resource": ["arn:aws:s3:::*"]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:Get*",
                "s3:List*"
            ],
            "Resource": [
                "arn:aws:s3:::<S3_BUCKET_NAME>/*",
                "arn:aws:s3:::<S3_BUCKET_NAME>"
            ]
        }
    ]
}
```

</details><br />

From the Bastion:

```bash
aws configure
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-152 ~]$ aws configure
AWS Access Key ID [None]: 
AWS Secret Access Key [None]: 
Default region name [None]: us-east-1
Default output format [None]: json
[cloud_user@ip-10-0-1-152 ~]$ 
```
</details><br />

![]({{ site.baseurl }}/images/services/iam.png)

## 1. Create IAM Policy

```bash
aws iam create-policy \
  --policy-name DevS3ReadAccess \
  --policy-document file://dev_s3_read_access.json
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-152 ~]$ aws iam create-policy --policy-name DevS3ReadAccess --policy-document file://dev_s3_read_access.json
{
    "Policy": {
        "PolicyName": "DevS3ReadAccess", 
        "PermissionsBoundaryUsageCount": 0, 
        "CreateDate": "2024-06-21T04:38:53Z", 
        "AttachmentCount": 0, 
        "IsAttachable": true, 
        "PolicyId": "ANPA4Z6EZZAXTN35TEGNL", 
        "DefaultVersionId": "v1", 
        "Path": "/", 
        "Arn": "arn:aws:iam::880344156207:policy/DevS3ReadAccess", 
        "UpdateDate": "2024-06-21T04:38:53Z"
    }
}
[cloud_user@ip-10-0-1-152 ~]$
```

</details><br />

IAM Policy created as seen in the AWS Management Console

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/03-IAM-Policy-created.png)

## 2. Create IAM Role

The IAM Role is created with the Trust Policy associated which gives to the EC2 service permissions to assume the role.

```bash
aws iam create-role \
  --role-name DEV_ROLE \
  --assume-role-policy-document file://trust_policy_ec2.json
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-152 ~]$ aws iam create-role --role-name DEV_ROLE --assume-role-policy-document file://trust_policy_ec2.json
{
    "Role": {
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17", 
            "Statement": [
                {
                    "Action": "sts:AssumeRole", 
                    "Effect": "Allow", 
                    "Principal": {
                        "Service": "ec2.amazonaws.com"
                    }
                }
            ]
        }, 
        "RoleId": "AROA4Z6EZZAX3ROCRLHHR", 
        "CreateDate": "2024-06-21T04:28:10Z", 
        "RoleName": "DEV_ROLE", 
        "Path": "/", 
        "Arn": "arn:aws:iam::880344156207:role/DEV_ROLE"
    }
}
[cloud_user@ip-10-0-1-152 ~]$ 
```

</details><br />

IAM Role created as seen in the AWS Management Console

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/02-IAM-Role-created.png)

## 3. Associate the IAM Policy to the IAM Role

```bash
aws iam attach-role-policy \
  --role-name DEV_ROLE \
  --policy-arn "arn:aws:iam::880344156207:policy/DevS3ReadAccess"
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-152 ~]$ aws iam attach-role-policy --role-name DEV_ROLE --policy-arn "arn:aws:iam::880344156207:policy/DevS3ReadAccess"
[cloud_user@ip-10-0-1-152 ~]$
```

</details><br />

To verify, list the attached IAM Policies for this IAM Role

```bash
aws iam list-attached-role-policies \
  --role-name DEV_ROLE
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-152 ~]$ aws iam list-attached-role-policies --role-name DEV_ROLE

{
    "AttachedPolicies": [
        {
            "PolicyName": "DevS3ReadAccess", 
            "PolicyArn": "arn:aws:iam::880344156207:policy/DevS3ReadAccess"
        }
    ]
}
[cloud_user@ip-10-0-1-152 ~]$ 
```
</details><br />

IAM Policy attached to the IAM Role as seen in the AWS Management Console

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/04-IAM-Policy-attached-to-IAM-Role.png)

## 4. Create an IAM Instance Profile

```bash
aws iam create-instance-profile \
  --instance-profile-name DEV_PROFILE
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-152 ~]$ aws iam create-instance-profile --instance-profile-name DEV_PROFILE
{
    "InstanceProfile": {
        "InstanceProfileId": "AIPA4Z6EZZAX3ANMOXI64", 
        "Roles": [], 
        "CreateDate": "2024-06-21T04:43:44Z", 
        "InstanceProfileName": "DEV_PROFILE", 
        "Path": "/", 
        "Arn": "arn:aws:iam::880344156207:instance-profile/DEV_PROFILE"
    }
}
[cloud_user@ip-10-0-1-152 ~]$
```

</details><br />

## 5. Associate the IAM Role to the IAM Instance Profile

```bash
aws iam add-role-to-instance-profile \
  --instance-profile-name DEV_PROFILE \
  --role-name DEV_ROLE
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-152 ~]$ aws iam add-role-to-instance-profile --instance-profile-name DEV_PROFILE --role-name DEV_ROLE
[cloud_user@ip-10-0-1-152 ~]$
```

</details><br />

To verfiy:

```bash
aws iam get-instance-profile \
  --instance-profile-name DEV_PROFILE
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-152 ~]$ aws iam get-instance-profile --instance-profile-name DEV_PROFILE

{
    "InstanceProfile": {
        "InstanceProfileId": "AIPA4Z6EZZAX3ANMOXI64", 
        "Roles": [
            {
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17", 
                    "Statement": [
                        {
                            "Action": "sts:AssumeRole", 
                            "Effect": "Allow", 
                            "Principal": {
                                "Service": "ec2.amazonaws.com"
                            }
                        }
                    ]
                }, 
                "RoleId": "AROA4Z6EZZAX3ROCRLHHR", 
                "CreateDate": "2024-06-21T04:28:10Z", 
                "RoleName": "DEV_ROLE", 
                "Path": "/", 
                "Arn": "arn:aws:iam::880344156207:role/DEV_ROLE"
            }
        ], 
        "CreateDate": "2024-06-21T04:43:44Z", 
        "InstanceProfileName": "DEV_PROFILE", 
        "Path": "/", 
        "Arn": "arn:aws:iam::880344156207:instance-profile/DEV_PROFILE"
    }
}
[cloud_user@ip-10-0-1-152 ~]$ 
```

</details><br />

IAM Instance Profile ARN shown in the IAM Role as seen in the AWS Management Console

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/05-IAM-Instance-Profile-in-IAM-Role.png)

## 6. Associate the IAM Instance Profile with the EC2 Instance

![]({{ site.baseurl }}/images/services/ec2.png)

```bash
aws ec2 associate-iam-instance-profile \
  --instance-id i-070823c5d33601d4f \
  --iam-instance-profile Name="DEV_PROFILE"
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-152 ~]$ aws ec2 associate-iam-instance-profile --instance-id i-070823c5d33601d4f --iam-instance-profile Name="DEV_PROFILE"
{
    "IamInstanceProfileAssociation": {
        "InstanceId": "i-070823c5d33601d4f", 
        "State": "associating", 
        "AssociationId": "iip-assoc-05d6cb7f1f7930315", 
        "IamInstanceProfile": {
            "Id": "AIPA4Z6EZZAX3ANMOXI64", 
            "Arn": "arn:aws:iam::880344156207:instance-profile/DEV_PROFILE"
        }
    }
}
[cloud_user@ip-10-0-1-152 ~]$
```

</details><br />

Verification in AWS CLI:

```bash
aws ec2 describe-instances \
  --instance-ids i-070823c5d33601d4f | egrep IamInstanceProfile -A3 
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-152 ~]$ aws ec2 describe-instances --instance-ids i-070823c5d33601d4f | egrep IamInstanceProfile -A3 
                    "IamInstanceProfile": {
                        "Id": "AIPA4Z6EZZAX3ANMOXI64", 
                        "Arn": "arn:aws:iam::880344156207:instance-profile/DEV_PROFILE"
                    },
[cloud_user@ip-10-0-1-152 ~]$ 
```

</details><br />

IAM Instance Profile associated to EC2 Instance as seen in the AWS Management Console

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/01-IAM-Role-attached-to-EC2-Instance.png)

## 7. Verification

From the EC2 instance:

```bash
aws sts get-caller-identity
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-46 ~]$ aws sts get-caller-identity
{
    "Account": "880344156207", 
    "UserId": "AROA4Z6EZZAX3ROCRLHHR:i-070823c5d33601d4f", 
    "Arn": "arn:aws:sts::880344156207:assumed-role/DEV_ROLE/i-070823c5d33601d4f"
}
[cloud_user@ip-10-0-1-46 ~]$ 
```

</details><br />

```bash
aws s3 ls
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-46 ~]$ aws s3 ls
2024-06-21 02:23:38 cfst-3035-52ca76f5ada32d15aa36-s3bucketengineering-824fw0br0cfy
2024-06-21 02:23:39 cfst-3035-52ca76f5ada32d15aa36-s3bucketlookupfiles-zexuabydhnz4
2024-06-21 02:23:38 cfst-3035-52ca76f5ada32d15aa36187-cloudtrailbucket-ezsk1oxlo9ye
2024-06-21 02:23:38 cfst-3035-52ca76f5ada32d15aa36187e1-s3bucketsecret-zurkfb5u53jk
2024-06-21 02:23:38 cfst-3035-52ca76f5ada32d15aa36187e15d-s3bucketprod-4oyyncl2dgfw
2024-06-21 02:23:38 cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
[cloud_user@ip-10-0-1-46 ~]$ 
```

</details><br />

```bash
aws s3 ls s3://cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-46 ~]$ aws s3 ls s3://cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
2024-06-21 02:25:04         41 file1-cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
2024-06-21 02:25:04         41 file2-cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
2024-06-21 02:25:04         41 file3-cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
2024-06-21 02:25:04         41 file4-cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
2024-06-21 02:25:04         41 file5-cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
[cloud_user@ip-10-0-1-46 ~]$ 
```

</details><br />

# Configuration using the AWS Management Console

![]({{ site.baseurl }}/images/services/iam.png)

## 1. Create IAM Policy

a)

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/06-Create-IAM-Policy-via-GUI.png)

b)

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/07-Create-IAM-Policy-via-GUI_2.png)

c)

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/08-Create-IAM-Policy-via-GUI_3.png)

## 2. Associate the IAM Policy with an IAM Role

a)

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/09-Create-IAM-Role-via-GUI.png)

b)

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/10-Create-IAM-Role-via-GUI.png)

c)

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/11-Create-IAM-Role-via-GUI.png)

d)

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/12-Create-IAM-Role-via-GUI.png)

e)

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/13-Create-IAM-Role-via-GUI.png)

![]({{ site.baseurl }}/images/services/ec2.png)

## 3. Associate the IAM Role to the EC2 instance

a)

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/14-Attach-IAM-Role-to-EC2-instance.png)

b)

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/15-Attach-IAM-Role-to-EC2-instance.png)

Verification:

![]({{ site.baseurl }}/images/2024/06-20-EC2-Roles-and-Instance-Profiles/16-Attach-IAM-Role-to-EC2-instance.png)

## 4. Verification

```bash
aws s3 ls s3://cfst-3035-ca2a0330a2c7f2c29971d25633e-s3bucketprod-uekmlotj3pna
```

<details markdown=1>
  <summary markdown="span">
    Output
  </summary>

```bash
[cloud_user@ip-10-0-1-104 ~]$ aws s3 ls s3://cfst-3035-ca2a0330a2c7f2c29971d25633e-s3bucketprod-uekmlotj3pna
2024-06-22 03:12:12         41 file1-cfst-3035-ca2a0330a2c7f2c29971d25633e-s3bucketprod-uekmlotj3pna
2024-06-22 03:12:12         41 file2-cfst-3035-ca2a0330a2c7f2c29971d25633e-s3bucketprod-uekmlotj3pna
2024-06-22 03:12:12         41 file3-cfst-3035-ca2a0330a2c7f2c29971d25633e-s3bucketprod-uekmlotj3pna
2024-06-22 03:12:12         41 file4-cfst-3035-ca2a0330a2c7f2c29971d25633e-s3bucketprod-uekmlotj3pna
2024-06-22 03:12:13         41 file5-cfst-3035-ca2a0330a2c7f2c29971d25633e-s3bucketprod-uekmlotj3pna
[cloud_user@ip-10-0-1-104 ~]$ 
```
</details><br />