---
title: IAM Policies
date: 2024-06-23 12:38:00 -0700
categories: [AWS, IAM]
tags: [aws, iam]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/iam.png)

There are 3 types of IAM Policies:

- **AWS managed policy** is a standalone policy that is created and administered by AWS.
- **Customer managed policy** is a standalone policy that you administer in your own AWS account.
- **Inline policy** is a policy that's embedded in an IAM identity (a user, group, or role).

1. Creating the IAM Policy

Policy file in JSON format `dynamoDB_readOnly_allResources.json`

```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "VisualEditor0",
			"Effect": "Allow",
			"Action": [
				"dynamodb:DescribeImport",
				"dynamodb:DescribeContributorInsights",
				"dynamodb:ListTagsOfResource",
				"dynamodb:DescribeReservedCapacityOfferings",
				"dynamodb:PartiQLSelect",
				"dynamodb:DescribeTable",
				"dynamodb:GetItem",
				"dynamodb:DescribeContinuousBackups",
				"dynamodb:DescribeExport",
				"dynamodb:GetResourcePolicy",
				"dynamodb:DescribeKinesisStreamingDestination",
				"dynamodb:DescribeLimits",
				"dynamodb:BatchGetItem",
				"dynamodb:ConditionCheckItem",
				"dynamodb:Scan",
				"dynamodb:Query",
				"dynamodb:DescribeStream",
				"dynamodb:DescribeTimeToLive",
				"dynamodb:ListStreams",
				"dynamodb:DescribeGlobalTableSettings",
				"dynamodb:GetShardIterator",
				"dynamodb:DescribeGlobalTable",
				"dynamodb:DescribeReservedCapacity",
				"dynamodb:DescribeBackup",
				"dynamodb:DescribeEndpoints",
				"dynamodb:GetRecords",
				"dynamodb:DescribeTableReplicaAutoScaling"
			],
			"Resource": "*"
		}
	]
}
```

Run `aws create-policy` in the AWS CLI

```
aws iam create-policy \
  --policy-name MyCustomerManagedPolicy \
  --policy-document file://dynamoDB_readOnly_allResources.json
```

<details>

<summary>Output</summary>

```json
88665a1c97a7:.aws hectorsg$ aws iam create-policy \
>   --policy-name MyCustomerManagedPolicy \
>   --policy-document file://dynamoDB_readOnly_allResources.json \
>   --profile cloudUser
{
    "Policy": {
        "PolicyName": "MyCustomerManagedPolicy",
        "PolicyId": "ANPAYFCWEVMDS4O3CID6W",
        "Arn": "arn:aws:iam::560673893127:policy/MyCustomerManagedPolicy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2024-06-23T19:50:54+00:00",
        "UpdateDate": "2024-06-23T19:50:54+00:00"
    }
}
```

</details>