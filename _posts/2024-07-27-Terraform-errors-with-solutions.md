---
title: Terraform errors with solutions
date: 2024-07-27 15:00:00 -0700
categories: [TERRAFORM, ERRORS with SOLUTIONS]
tags: [terraform, terraform errors with solutions]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/terraform_logo.png)

## Error message: operation error DynamoDB: PutItem, https response error StatusCode: 400

### Error

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$ terraform plan
╷
│ Error: Error acquiring the state lock
│
│ Error message: operation error DynamoDB: PutItem, https response error StatusCode: 400, RequestID: 88QGT8SNLDI8FAJ20TQ0DI5VU3VV4KQNSO5AEMVJF66Q9ASUAAJG, ConditionalCheckFailedException: The conditional request failed
│ Lock Info:
│   ID:        96bebb39-4600-10c1-e80d-030c42410701
│   Path:      ...
│   Operation: OperationTypePlan
│   Who:       ...
│   Version:   1.9.2
│   Created:   2024-07-27 21:19:22.374555726 +0000 UTC
│   Info:
│
│
│ Terraform acquires a state lock to protect the state from being written
│ by multiple users at the same time. Please resolve the issue above and try
│ again. For most commands, you can disable locking with the "-lock=false"
│ flag, but this is not recommended.
╵
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$
```

### Explanation

The problem was that prior to this error message, I cancelled the execute of the `terraform plan` command before it could complete

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$ terraform plan
var.db_password
  The password for the database

  Enter a value:

Interrupt received.
Please wait for Terraform to exit or data loss may occur.
Gracefully shutting down...

cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$
```

### Solution

The solution is simply release the lock manually

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$ terraform force-unlock 96bebb39-4600-10c1-e80d-030c42410701
Do you really want to force-unlock?
  Terraform will remove the lock on the remote state.
  This will allow local Terraform commands to modify this state, even though it
  may still be in use. Only 'yes' will be accepted to confirm.

  Enter a value: yes

Terraform state has been successfully unlocked!

The state has been unlocked, and Terraform commands should now be able to
obtain a new lock on the remote state.
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$
```

## InvalidParameterCombination: RDS does not support creating a DB instance with the following combination

### Error

```bash
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_db_instance.example: Creating...
╷
│ Error: creating RDS DB Instance (database-for-webserver20240727214640354500000001): InvalidParameterCombination: RDS does not support creating a DB instance with the following combination: DBInstanceClass=db.t2.micro, Engine=mysql, EngineVersion=8.0.35, LicenseModel=general-public-license. For supported combinations of instance class and database engine version, see the documentation.
│ 	status code: 400, request id: 0b36640a-cfd1-48ec-a5f7-353dad6c2d00
│
│   with aws_db_instance.example,
│   on main.tf line 5, in resource "aws_db_instance" "example":
│    5: resource "aws_db_instance" "example" {
│
╵
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$
```

### Explanation

For [Resource: aws_db_instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_instance), the `instance_class` with value `"db.t2.micro"` is **invalid**

``tf
resource "aws_db_instance" "example" {
  instance_class      = "db.t2.micro"
  ...
``

### Solution

Use at least `db.t3.micro` as `instance_class` when defining the resource

``tf
resource "aws_db_instance" "example" {
  instance_class      = "db.t3.micro"
  ...
``


## Error: creating RDS DB Instance InvalidParameterValue: Invalid master user name

### Error

```bash
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_db_instance.example: Creating...
╷
│ Error: creating RDS DB Instance (database-for-webserver20240727214912645200000001): InvalidParameterValue: Invalid master user name
│ 	status code: 400, request id: 6180494a-ef52-4691-adc0-7986b7107db8
│
│   with aws_db_instance.example,
│   on main.tf line 5, in resource "aws_db_instance" "example":
│    5: resource "aws_db_instance" "example" {
│
╵
```

### Explanation

I was setting the value of `username` (and also `password`) to be an specific string as opposed to be assigned by variables.


```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_db_instance" "example" {
  username = "var.db_username"
  password = "var.db_password"
  ...
}
```

### Solution

Removed the **quotes** from the username and password

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_db_instance" "example" {
  username = var.db_username
  password = var.db_password
  ...
}
```

## No outputs found

### Error

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$ terraform output
╷
│ Warning: No outputs found
│
│ The state file either has no outputs defined, or all the defined outputs are empty. Please define an output in your configuration with the `output` keyword and run `terraform refresh` for it to become available. If
│ you are using interpolation, please verify the interpolated value is not empty. You can use the `terraform console` command to assist.
╵
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$
```

### Explanation

I created the **output variable** after having run `terraform apply`

### Solution

Run `terraform refresh` as suggested by the Warning message

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$ terraform refresh
data.terraform_remote_state.db: Reading...
data.terraform_remote_state.db: Read complete after 0s
aws_security_group.instance: Refreshing state... [id=sg-081a9b0f536d8d462]
aws_instance.example: Refreshing state... [id=i-004bec7f9e311d32f]

Outputs:

webserver_dns_name = "ec2-XX-XX-XX-XXX.compute-1.amazonaws.com"
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$

cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$ terraform output
webserver_dns_name = "ec2-XX-XX-XX-XXX.compute-1.amazonaws.com"
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$
```

## References

- [Remove Terraform s3 backend state lock](https://ducpp.medium.com/remove-terraform-s3-backend-state-lock-e801c5ffa190)
- [Discussions related to Terraform or Terraform Modules](https://archive.sweetops.com/terraform/2023/07/)
