---
title: EC2 Roles and Instance Profiles
date: 2024-06-20 20:42:00 -0700
categories: [AWS, IAM, EC2]
tags: [aws, iam, ec2]     # TAG names should always be lowercase
---

# Introduction

**IMPORTANT**
*The outputs shown here where taken from a temporary AWS account which has now been deleted.*

Amazon EC2 instances should be able to securely access other AWS services. Credentials need to be rotated regularly to minimize the adverse impact of a security breach.

- **AWS IAM roles** provide the ability to automatically grant instances temporary credentials without the need for manual management.
- **IAM instance profiles** provide the mechanism to attach IAM roles to EC2 instances.

- **Trust Policy**: Establishes trust, answering who can assume the role.

- **Permissions Policy**: Defines access levels, detailing what actions a principal can or cannot do on AWS resources.

# Configuration via AWS CLI

## Pre-requisites

Trust Policy file: `trust_policy_ec2.json`

```
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

IAM Policy file: `dev_s3_read_access.json`

```
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

From the Bastion

```
[cloud_user@ip-10-0-1-152 ~]$ aws configure
AWS Access Key ID [None]: 
AWS Secret Access Key [None]: 
Default region name [None]: us-east-1
Default output format [None]: json
[cloud_user@ip-10-0-1-152 ~]$ 
```
## 1. Create IAM Policy

```
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

## 2. Create IAM Role

The IAM Role is created with the Trust Policy attached which gives to the EC2 service permissions to assume the role.

```
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

## 3. Associate the IAM Policy to the IAM Role

```
[cloud_user@ip-10-0-1-152 ~]$ aws iam attach-role-policy --role-name DEV_role --policy-arn "arn:aws:iam::880344156207:policy/DevS3ReadAccess"
[cloud_user@ip-10-0-1-152 ~]$
```

To verify, list the attached IAM Policies for this IAM Role

```
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

## 4. Create an IAM Instance Profile

```
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

## 5. Attach the IAM Role to the IAM Instance Profile

```
[cloud_user@ip-10-0-1-152 ~]$ aws iam add-role-to-instance-profile --instance-profile-name DEV_PROFILE --role-name DEV_ROLE
[cloud_user@ip-10-0-1-152 ~]$
```

To verfiy:

```
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

## 6. Associate the IAM Instance Profile with the EC2 Instance

```
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

To verify:

<details open>
  <summary>aws ec2 describe-instances --instance-ids $INSTANCE_ID</summary>

```
[cloud_user@ip-10-0-1-152 ~]$ aws ec2 describe-instances --instance-ids i-070823c5d33601d4f
{
    "Reservations": [
        {
            "Instances": [
                {
                    "Monitoring": {
                        "State": "disabled"
                    }, 
                    "PublicDnsName": "ec2-54-145-81-219.compute-1.amazonaws.com", 
                    "State": {
                        "Code": 16, 
                        "Name": "running"
                    }, 
                    "EbsOptimized": false, 
                    "LaunchTime": "2024-06-21T02:23:54.000Z", 
                    "PublicIpAddress": "54.145.81.219", 
                    "PrivateIpAddress": "10.0.1.46", 
                    "ProductCodes": [], 
                    "VpcId": "vpc-0c98e19f2c023fcdf", 
                    "CpuOptions": {
                        "CoreCount": 1, 
                        "ThreadsPerCore": 2
                    }, 
                    "StateTransitionReason": "", 
                    "InstanceId": "i-070823c5d33601d4f", 
                    "EnaSupport": true, 
                    "ImageId": "ami-046fc64ce51e6ccab", 
                    "PrivateDnsName": "ip-10-0-1-46.ec2.internal", 
                    "SecurityGroups": [
                        {
                            "GroupName": "cfst-3035-52ca76f5ada32d15aa36187e15dd0e64-SecurityGroupHTTPAndSSH-6HoPH3j3tOvN", 
                            "GroupId": "sg-07f9529db4ceaa38a"
                        }
                    ], 
                    "ClientToken": "2dbeece3-f285-1e37-c2ca-1a18ecdff753", 
                    "SubnetId": "subnet-077f734ee83abc902", 
                    "InstanceType": "t3.micro", 
                    "CapacityReservationSpecification": {
                        "CapacityReservationPreference": "open"
                    }, 
                    "NetworkInterfaces": [
                        {
                            "Status": "in-use", 
                            "MacAddress": "0e:5a:9f:9e:ab:97", 
                            "SourceDestCheck": true, 
                            "VpcId": "vpc-0c98e19f2c023fcdf", 
                            "Description": "", 
                            "NetworkInterfaceId": "eni-023b7e1a74e0dd32e", 
                            "PrivateIpAddresses": [
                                {
                                    "PrivateDnsName": "ip-10-0-1-46.ec2.internal", 
                                    "PrivateIpAddress": "10.0.1.46", 
                                    "Primary": true, 
                                    "Association": {
                                        "PublicIp": "54.145.81.219", 
                                        "PublicDnsName": "ec2-54-145-81-219.compute-1.amazonaws.com", 
                                        "IpOwnerId": "amazon"
                                    }
                                }
                            ], 
                            "PrivateDnsName": "ip-10-0-1-46.ec2.internal", 
                            "InterfaceType": "interface", 
                            "Attachment": {
                                "Status": "attached", 
                                "DeviceIndex": 0, 
                                "DeleteOnTermination": true, 
                                "AttachmentId": "eni-attach-056ac16923e808825", 
                                "AttachTime": "2024-06-21T02:23:54.000Z"
                            }, 
                            "Groups": [
                                {
                                    "GroupName": "cfst-3035-52ca76f5ada32d15aa36187e15dd0e64-SecurityGroupHTTPAndSSH-6HoPH3j3tOvN", 
                                    "GroupId": "sg-07f9529db4ceaa38a"
                                }
                            ], 
                            "Ipv6Addresses": [], 
                            "OwnerId": "880344156207", 
                            "PrivateIpAddress": "10.0.1.46", 
                            "SubnetId": "subnet-077f734ee83abc902", 
                            "Association": {
                                "PublicIp": "54.145.81.219", 
                                "PublicDnsName": "ec2-54-145-81-219.compute-1.amazonaws.com", 
                                "IpOwnerId": "amazon"
                            }
                        }
                    ], 
                    "SourceDestCheck": true, 
                    "Placement": {
                        "Tenancy": "default", 
                        "GroupName": "", 
                        "AvailabilityZone": "us-east-1a"
                    }, 
                    "Hypervisor": "xen", 
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/xvda", 
                            "Ebs": {
                                "Status": "attached", 
                                "DeleteOnTermination": true, 
                                "VolumeId": "vol-0757c962419a8240a", 
                                "AttachTime": "2024-06-21T02:23:55.000Z"
                            }
                        }
                    ], 
                    "Architecture": "x86_64", 
                    "RootDeviceType": "ebs", 
                    "IamInstanceProfile": {
                        "Id": "AIPA4Z6EZZAX3ANMOXI64", 
                        "Arn": "arn:aws:iam::880344156207:instance-profile/DEV_PROFILE"
                    }, 
                    "RootDeviceName": "/dev/xvda", 
                    "VirtualizationType": "hvm", 
                    "Tags": [
                        {
                            "Value": "EC2InstanceWebServer", 
                            "Key": "aws:cloudformation:logical-id"
                        }, 
                        {
                            "Value": "cfst-3035-52ca76f5ada32d15aa36187e15dd0e64", 
                            "Key": "aws:cloudformation:stack-name"
                        }, 
                        {
                            "Value": "arn:aws:cloudformation:us-east-1:880344156207:stack/cfst-3035-52ca76f5ada32d15aa36187e15dd0e64/40664ce0-2f75-11ef-bea5-0e5f2c0b2977", 
                            "Key": "Application"
                        }, 
                        {
                            "Value": "21378432", 
                            "Key": "UserId"
                        }, 
                        {
                            "Value": "Web Server", 
                            "Key": "Name"
                        }, 
                        {
                            "Value": "arn:aws:cloudformation:us-east-1:880344156207:stack/cfst-3035-52ca76f5ada32d15aa36187e15dd0e64/40664ce0-2f75-11ef-bea5-0e5f2c0b2977", 
                            "Key": "aws:cloudformation:stack-id"
                        }
                    ], 
                    "HibernationOptions": {
                        "Configured": false
                    }, 
                    "MetadataOptions": {
                        "State": "applied", 
                        "HttpEndpoint": "enabled", 
                        "HttpTokens": "optional", 
                        "HttpPutResponseHopLimit": 1
                    }, 
                    "AmiLaunchIndex": 0
                }
            ], 
            "ReservationId": "r-0fa72987155594236", 
            "RequesterId": "043234062703", 
            "Groups": [], 
            "OwnerId": "880344156207"
        }
    ]
}
[cloud_user@ip-10-0-1-152 ~]$ 
```
</details>

## 7. Verification

From the EC2 instance

```
[cloud_user@ip-10-0-1-46 ~]$ aws sts get-caller-identity
{
    "Account": "880344156207", 
    "UserId": "AROA4Z6EZZAX3ROCRLHHR:i-070823c5d33601d4f", 
    "Arn": "arn:aws:sts::880344156207:assumed-role/DEV_ROLE/i-070823c5d33601d4f"
}
[cloud_user@ip-10-0-1-46 ~]$ 
```

![]({{ site.baseurl }}/images/2024/2024-06-EC2-Rooles-and-Instance-Profiles/01-IAM-Role-attached-to-EC2-Instance.png)

```
[cloud_user@ip-10-0-1-46 ~]$ aws s3 ls
2024-06-21 02:23:38 cfst-3035-52ca76f5ada32d15aa36-s3bucketengineering-824fw0br0cfy
2024-06-21 02:23:39 cfst-3035-52ca76f5ada32d15aa36-s3bucketlookupfiles-zexuabydhnz4
2024-06-21 02:23:38 cfst-3035-52ca76f5ada32d15aa36187-cloudtrailbucket-ezsk1oxlo9ye
2024-06-21 02:23:38 cfst-3035-52ca76f5ada32d15aa36187e1-s3bucketsecret-zurkfb5u53jk
2024-06-21 02:23:38 cfst-3035-52ca76f5ada32d15aa36187e15d-s3bucketprod-4oyyncl2dgfw
2024-06-21 02:23:38 cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
[cloud_user@ip-10-0-1-46 ~]$ 
```

```
[cloud_user@ip-10-0-1-46 ~]$ aws s3 ls s3://cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
2024-06-21 02:25:04         41 file1-cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
2024-06-21 02:25:04         41 file2-cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
2024-06-21 02:25:04         41 file3-cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
2024-06-21 02:25:04         41 file4-cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
2024-06-21 02:25:04         41 file5-cfst-3035-52ca76f5ada32d15aa36187e15dd-s3bucketdev-yedz0kexet91
[cloud_user@ip-10-0-1-46 ~]$ 
```

# Configuration using the AWS Management Console

## 1. Create IAM Policy


## 2. Associate the IAM Policy with an IAM Role


## 3. Attach the IAM Role to the EC2 instance


## 4. Validation

