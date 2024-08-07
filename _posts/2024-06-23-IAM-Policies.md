---
title: Create IAM Policies using AWS CLI
date: 2024-06-23 12:38:00 -0700
categories: [AWS, IAM]
tags: [aws, iam]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/iam.png)

There are 3 types of IAM Policies:

- **Customer managed policy** is a standalone policy that you administer in your own AWS account.
- **AWS managed policy** is a standalone policy that is created and administered by AWS.
- **Inline policy** is a policy that's embedded in an IAM identity (a user, group, or role).

# Steps

1. Create an IAM Policy (this is a "Customer managed policy")

Policy file in JSON format `dynamoDB_readOnly_allResources.json`

<details markdown=1><summary markdown="span">Output</summary>

```json
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

</details><br />

Run `aws create-policy` in the AWS CLI to create a **Customer managed policy**

```bash
aws iam create-policy \
  --policy-name MyCustomerManagedPolicy \
  --policy-document file://dynamoDB_readOnly_allResources.json
```

<!--
How to use <details><summary> and properly render markdown code insite it
https://github.com/gettalong/kramdown/issues/155#issuecomment-1024896918
-->

<details markdown=1><summary markdown="span">Output</summary>

```bash
$ aws iam create-policy \
>   --policy-name MyCustomerManagedPolicy \
>   --policy-document file://dynamoDB_readOnly_allResources.json \
>   --profile cloudUser
{
    "Policy": {
        "PolicyName": "MyCustomerManagedPolicy",
        "PolicyId": "ANPAUAMBKUK62TMBIRE7M",
        "Arn": "arn:aws:iam::275686007485:policy/MyCustomerManagedPolicy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2024-06-23T20:59:41+00:00",
        "UpdateDate": "2024-06-23T20:59:41+00:00"
    }
}
$
```

</details><br />

2. Attach the IAM Policy to the user

```bash
aws iam attach-user-policy \
  --user-name charlie \
  --policy-arn "arn:aws:iam::275686007485:policy/MyCustomerManagedPolicy"
```

<details markdown=1><summary markdown="span">Output</summary>

```bash
$ aws iam attach-user-policy \
>   --user-name charlie \
>   --policy-arn "arn:aws:iam::275686007485:policy/MyCustomerManagedPolicy" \
>   --profile cloudUser
$
```

</details><br />

3. Find an AWS managed policy

```bash
aws iam list-policie
```

<details markdown=1><summary markdown="span">Output</summary>

```bash
$ aws iam list-policies --profile cloudUser
{
    "Policies": [
        {
...skipping...
            "PolicyName": "AWSLambda_FullAccess",
            "PolicyId": "ANPAZKAPJZG4OXQPYWZ5D",
            "Arn": "arn:aws:iam::aws:policy/AWSLambda_FullAccess",
            "Path": "/",
            "DefaultVersionId": "v1",
            "AttachmentCount": 0,
            "PermissionsBoundaryUsageCount": 0,
            "IsAttachable": true,
            "CreateDate": "2020-11-17T21:14:08+00:00",
            "UpdateDate": "2020-11-17T21:14:08+00:00"
        },
...skipping...
```

</details><br />

4. Attach the AWS managed policy to a user

```bash
aws iam attach-user-policy \
  --user-name sally \
  --policy-arn "arn:aws:iam::aws:policy/AWSLambda_FullAccess"
```

<details markdown=1><summary markdown="span">Output</summary>

```bash
$ aws iam attach-user-policy \
>   --user-name sally \
>   --policy-arn "arn:aws:iam::aws:policy/AWSLambda_FullAccess" \
>   --profile cloudUser
```

</details><br />

5. Create and attach an inline policy to a User

Policy file in JSON format `s3_readOnly_allResources.json`

<details markdown=1><summary markdown="span">Output</summary>

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "VisualEditor0",
			"Effect": "Allow",
			"Action": [
				"s3:GetObjectVersionTagging",
				"s3:GetStorageLensConfigurationTagging",
				"s3:GetObjectAcl",
				"s3:GetBucketObjectLockConfiguration",
				"s3:GetIntelligentTieringConfiguration",
				"s3:GetStorageLensGroup",
				"s3:GetAccessGrantsInstanceForPrefix",
				"s3:GetObjectVersionAcl",
				"s3:GetBucketPolicyStatus",
				"s3:GetAccessGrantsLocation",
				"s3:GetObjectRetention",
				"s3:GetBucketWebsite",
				"s3:GetJobTagging",
				"s3:GetMultiRegionAccessPoint",
				"s3:GetObjectAttributes",
				"s3:GetAccessGrantsInstanceResourcePolicy",
				"s3:GetObjectLegalHold",
				"s3:GetBucketNotification",
				"s3:DescribeMultiRegionAccessPointOperation",
				"s3:GetReplicationConfiguration",
				"s3:GetObject",
				"s3:DescribeJob",
				"s3:GetAnalyticsConfiguration",
				"s3:GetObjectVersionForReplication",
				"s3:GetAccessPointForObjectLambda",
				"s3:GetStorageLensDashboard",
				"s3:GetLifecycleConfiguration",
				"s3:GetAccessPoint",
				"s3:GetInventoryConfiguration",
				"s3:GetBucketTagging",
				"s3:GetAccessPointPolicyForObjectLambda",
				"s3:GetBucketLogging",
				"s3:GetAccessGrant",
				"s3:GetAccelerateConfiguration",
				"s3:GetObjectVersionAttributes",
				"s3:GetBucketPolicy",
				"s3:GetEncryptionConfiguration",
				"s3:GetObjectVersionTorrent",
				"s3:GetBucketRequestPayment",
				"s3:GetAccessPointPolicyStatus",
				"s3:GetAccessGrantsInstance",
				"s3:GetObjectTagging",
				"s3:GetMetricsConfiguration",
				"s3:GetBucketOwnershipControls",
				"s3:GetBucketPublicAccessBlock",
				"s3:GetMultiRegionAccessPointPolicyStatus",
				"s3:GetMultiRegionAccessPointPolicy",
				"s3:GetAccessPointPolicyStatusForObjectLambda",
				"s3:GetDataAccess",
				"s3:GetBucketVersioning",
				"s3:GetBucketAcl",
				"s3:GetAccessPointConfigurationForObjectLambda",
				"s3:GetObjectTorrent",
				"s3:GetMultiRegionAccessPointRoutes",
				"s3:GetStorageLensConfiguration",
				"s3:GetAccountPublicAccessBlock",
				"s3:GetBucketCORS",
				"s3:GetBucketLocation",
				"s3:GetAccessPointPolicy",
				"s3:GetObjectVersion"
			],
			"Resource": "*"
		}
	]
}
```

</details><br />

```bash
aws iam put-user-policy \
  --user-name ian \
  --policy-name MyInlinePolicy \
  --policy-document "$(cat s3_readOnly_allResources.json)"
```

<details markdown=1><summary markdown="span">Output</summary>

```bash
$ aws iam put-user-policy \
>   --user-name ian \
>   --policy-name MyInlinePolicy \
>   --policy-document "$(cat s3_readOnly_allResources.json)" \
>   --profile cloudUser
$
```

</details><br />
