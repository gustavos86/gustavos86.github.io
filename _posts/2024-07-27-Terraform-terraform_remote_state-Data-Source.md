---
title: The terraform_remote_state Data Source
date: 2024-07-27 09:30:00 -0700
categories: [TERRAFORM]
tags: [terraform]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/terraform_logo.png)

## Introduction

In this post we will be creating 2 isolated resources in AWS, a webserver (and EC2 instance) and a data-store (and RDS instance)

## Create the directory structure

We first create a new directory where we are going to have the structure for this proof of concept

Create the new directory and move into it

```bash
cloud_user@553b1e446c1c:~$ mkdir terraform_file_isolation
cloud_user@553b1e446c1c:~$ cd terraform_file_isolation/
cloud_user@553b1e446c1c:~/terraform_file_isolation$
```

After creating three more folder, the layout should look like this

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation$ mkdir terraform-backend
cloud_user@553b1e446c1c:~/terraform_file_isolation$ mkdir webserver
cloud_user@553b1e446c1c:~/terraform_file_isolation$ mkdir data-store

cloud_user@553b1e446c1c:~/terraform_file_isolation$ tree
.
├── data-store
├── terraform-backend
└── webserver

3 directories, 0 files
cloud_user@553b1e446c1c:~/terraform_file_isolation$
```

## Create the terraform-backend

We will create an S3 bucket and a DynamoDB table.

**NOTE:** This directory will have a local backend.

The directory structure should be as follows once the files are created

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation$ tree
.
├── data-store
├── terraform-backend
│   ├── main.tf
│   └── terraform.tfstate
└── webserver

3 directories, 2 files
cloud_user@553b1e446c1c:~/terraform_file_isolation$
```

Create a file `main.tf` in the `terraform-backend` directory

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation$ cd terraform-backend/
cloud_user@553b1e446c1c:~/terraform_file_isolation/terraform-backend$ vim main.tf
```

-`main.tf`

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "terraform_state" {
  bucket = "terraform-backend-20240727095931"

  # Prevent accidental deletion of this S3 bucket
  lifecycle {
    prevent_destroy = true
  }
}

# Enable versioning to see the full revision history of state files
resource "aws_s3_bucket_versioning" "enabled" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Enable server-side encryption by default
resource "aws_s3_bucket_server_side_encryption_configuration" "default" {
  bucket = aws_s3_bucket.terraform_state.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Explicitly block all public access to the S3 bucket
resource "aws_s3_bucket_public_access_block" "public_access" {
  bucket                  = aws_s3_bucket.terraform_state.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

Now we run the terraform `init`, `plan` (optional but recommended for verifications) and `apply` commands

<details markdown=1>
<summary markdown="span">terraform init</summary>

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/terraform-backend$ terraform init
Initializing the backend...
Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v5.60.0...
- Installed hashicorp/aws v5.60.0 (signed by HashiCorp)
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
cloud_user@553b1e446c1c:~/terraform_file_isolation/terraform-backend$
```
</details><br />

<details markdown=1>
<summary markdown="span">terraform plan</summary>

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/terraform-backend$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_dynamodb_table.terraform_locks will be created
  + resource "aws_dynamodb_table" "terraform_locks" {
      + arn              = (known after apply)
      + billing_mode     = "PAY_PER_REQUEST"
      + hash_key         = "LockID"
      + id               = (known after apply)
      + name             = "terraform-locks"
      + read_capacity    = (known after apply)
      + stream_arn       = (known after apply)
      + stream_label     = (known after apply)
      + stream_view_type = (known after apply)
      + tags_all         = (known after apply)
      + write_capacity   = (known after apply)

      + attribute {
          + name = "LockID"
          + type = "S"
        }

      + point_in_time_recovery (known after apply)

      + server_side_encryption (known after apply)

      + ttl (known after apply)
    }

  # aws_s3_bucket.terraform_state will be created
  + resource "aws_s3_bucket" "terraform_state" {
      + acceleration_status         = (known after apply)
      + acl                         = (known after apply)
      + arn                         = (known after apply)
      + bucket                      = "terraform-backend-20240727095931"
      + bucket_domain_name          = (known after apply)
      + bucket_prefix               = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + object_lock_enabled         = (known after apply)
      + policy                      = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + tags_all                    = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)

      + cors_rule (known after apply)

      + grant (known after apply)

      + lifecycle_rule (known after apply)

      + logging (known after apply)

      + object_lock_configuration (known after apply)

      + replication_configuration (known after apply)

      + server_side_encryption_configuration (known after apply)

      + versioning (known after apply)

      + website (known after apply)
    }

  # aws_s3_bucket_public_access_block.public_access will be created
  + resource "aws_s3_bucket_public_access_block" "public_access" {
      + block_public_acls       = true
      + block_public_policy     = true
      + bucket                  = (known after apply)
      + id                      = (known after apply)
      + ignore_public_acls      = true
      + restrict_public_buckets = true
    }

  # aws_s3_bucket_server_side_encryption_configuration.default will be created
  + resource "aws_s3_bucket_server_side_encryption_configuration" "default" {
      + bucket = (known after apply)
      + id     = (known after apply)

      + rule {
          + apply_server_side_encryption_by_default {
              + sse_algorithm     = "AES256"
                # (1 unchanged attribute hidden)
            }
        }
    }

  # aws_s3_bucket_versioning.enabled will be created
  + resource "aws_s3_bucket_versioning" "enabled" {
      + bucket = (known after apply)
      + id     = (known after apply)

      + versioning_configuration {
          + mfa_delete = (known after apply)
          + status     = "Enabled"
        }
    }

Plan: 5 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
cloud_user@553b1e446c1c:~/terraform_file_isolation/terraform-backend$
```
</details><br />

<details markdown=1>
<summary markdown="span">terraform apply</summary>

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/terraform-backend$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_dynamodb_table.terraform_locks will be created
  + resource "aws_dynamodb_table" "terraform_locks" {
      + arn              = (known after apply)
      + billing_mode     = "PAY_PER_REQUEST"
      + hash_key         = "LockID"
      + id               = (known after apply)
      + name             = "terraform-locks"
      + read_capacity    = (known after apply)
      + stream_arn       = (known after apply)
      + stream_label     = (known after apply)
      + stream_view_type = (known after apply)
      + tags_all         = (known after apply)
      + write_capacity   = (known after apply)

      + attribute {
          + name = "LockID"
          + type = "S"
        }

      + point_in_time_recovery (known after apply)

      + server_side_encryption (known after apply)

      + ttl (known after apply)
    }

  # aws_s3_bucket.terraform_state will be created
  + resource "aws_s3_bucket" "terraform_state" {
      + acceleration_status         = (known after apply)
      + acl                         = (known after apply)
      + arn                         = (known after apply)
      + bucket                      = "terraform-backend-20240727095931"
      + bucket_domain_name          = (known after apply)
      + bucket_prefix               = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + object_lock_enabled         = (known after apply)
      + policy                      = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + tags_all                    = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)

      + cors_rule (known after apply)

      + grant (known after apply)

      + lifecycle_rule (known after apply)

      + logging (known after apply)

      + object_lock_configuration (known after apply)

      + replication_configuration (known after apply)

      + server_side_encryption_configuration (known after apply)

      + versioning (known after apply)

      + website (known after apply)
    }

  # aws_s3_bucket_public_access_block.public_access will be created
  + resource "aws_s3_bucket_public_access_block" "public_access" {
      + block_public_acls       = true
      + block_public_policy     = true
      + bucket                  = (known after apply)
      + id                      = (known after apply)
      + ignore_public_acls      = true
      + restrict_public_buckets = true
    }

  # aws_s3_bucket_server_side_encryption_configuration.default will be created
  + resource "aws_s3_bucket_server_side_encryption_configuration" "default" {
      + bucket = (known after apply)
      + id     = (known after apply)

      + rule {
          + apply_server_side_encryption_by_default {
              + sse_algorithm     = "AES256"
                # (1 unchanged attribute hidden)
            }
        }
    }

  # aws_s3_bucket_versioning.enabled will be created
  + resource "aws_s3_bucket_versioning" "enabled" {
      + bucket = (known after apply)
      + id     = (known after apply)

      + versioning_configuration {
          + mfa_delete = (known after apply)
          + status     = "Enabled"
        }
    }

Plan: 5 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_dynamodb_table.terraform_locks: Creating...
aws_s3_bucket.terraform_state: Creating...
aws_s3_bucket.terraform_state: Creation complete after 0s [id=terraform-backend-20240727095931]
aws_s3_bucket_versioning.enabled: Creating...
aws_s3_bucket_public_access_block.public_access: Creating...
aws_s3_bucket_server_side_encryption_configuration.default: Creating...
aws_s3_bucket_server_side_encryption_configuration.default: Creation complete after 0s [id=terraform-backend-20240727095931]
aws_s3_bucket_public_access_block.public_access: Creation complete after 0s [id=terraform-backend-20240727095931]
aws_s3_bucket_versioning.enabled: Creation complete after 2s [id=terraform-backend-20240727095931]
aws_dynamodb_table.terraform_locks: Still creating... [10s elapsed]
aws_dynamodb_table.terraform_locks: Creation complete after 13s [id=terraform-locks]

Apply complete! Resources: 5 added, 0 changed, 0 destroyed.
cloud_user@553b1e446c1c:~/terraform_file_isolation/terraform-backend$
```
</details><br />

The resources were created and can be seen in the **AWS Management Console**

![]({{ site.baseurl }}/images/services/s3.png)

![]({{ site.baseurl }}/images/2024/07-27-Terraform-terraform_remote_state-Data-Source/01-S3-bucket-created.png)

![]({{ site.baseurl }}/images/services/dynamodb.png)

![]({{ site.baseurl }}/images/2024/07-27-Terraform-terraform_remote_state-Data-Source/02-DynamoDB-table-created.png)

## Create the data-store

The data-store will be an **RDS** instance. We will move to the `data-store` directory and create the following files

The directory structure should look like as following as this point:

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation$ tree
.
├── data-store
│   ├── backend.tf
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── terraform-backend
│   ├── main.tf
│   └── terraform.tfstate
└── webserver

3 directories, 6 files
cloud_user@553b1e446c1c:~/terraform_file_isolation$
```

- `backend.tf`

To use the **S3 bucket** & **Dynamo DB table** as remote backend for the state file and take the lock. These were created in the `terraform-backend` directory in the previous sections.

```tf
terraform {
  backend "s3" {
    # Replace this with your bucket name!
    bucket         = "terraform-backend-20240727095931"
    key            = "data-store/terraform.tfstate"
    region         = "us-east-1"

    # Replace this with your DynamoDB table name!
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

- `main.tf`

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_db_instance" "example" {
  identifier_prefix   = "database-for-webserver"
  engine              = "mysql"
  allocated_storage   = 10
  instance_class      = "db.t3.micro"
  skip_final_snapshot = true
  db_name             = "example_database"

  # We use variables and create them using
  # the environment variables approach to keep these as a secret
  username = var.db_username
  password = var.db_password
}
```

- `variables.tf`

```tf
variable "db_username" {
  description = "The username for the database"
  type        = string
  sensitive   = true
}

variable "db_password" {
  description = "The password for the database"
  type        = string
  sensitive   = true
}
```

To keep the **db_username** & **db_password** as secrets, these are passed as variables. We set them as environment variables for this purpose. Just come up with some username and password and set them with:

```bash
$ export TF_VAR_db_username="(YOUR_DB_USERNAME)"
$ export TF_VAR_db_password="(YOUR_DB_PASSWORD)"
```

- `outputs.tf`

```
output "address" {
  value       = aws_db_instance.example.address
  description = "Connect to the database at this endpoint"
}

output "port" {
  value       = aws_db_instance.example.port
  description = "The port the database is listening on"
}
```

And now we run the terraform `init`, `plan` and the `apply` command

<details markdown=1>
<summary markdown="span">terraform init</summary>

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$ terraform init
Initializing the backend...

Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.
Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v5.60.0...
- Installed hashicorp/aws v5.60.0 (signed by HashiCorp)
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$
```
</details><br />

<details markdown=1>
<summary markdown="span">terraform plan</summary>

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$ export TF_VAR_db_username=admin
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$ export TF_VAR_db_password=superMegaStrongPassword
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$

cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_db_instance.example will be created
  + resource "aws_db_instance" "example" {
      + address                               = (known after apply)
      + allocated_storage                     = 10
      + apply_immediately                     = false
      + arn                                   = (known after apply)
      + auto_minor_version_upgrade            = true
      + availability_zone                     = (known after apply)
      + backup_retention_period               = (known after apply)
      + backup_target                         = (known after apply)
      + backup_window                         = (known after apply)
      + ca_cert_identifier                    = (known after apply)
      + character_set_name                    = (known after apply)
      + copy_tags_to_snapshot                 = false
      + db_name                               = "example_database"
      + db_subnet_group_name                  = (known after apply)
      + dedicated_log_volume                  = false
      + delete_automated_backups              = true
      + domain_fqdn                           = (known after apply)
      + endpoint                              = (known after apply)
      + engine                                = "mysql"
      + engine_lifecycle_support              = (known after apply)
      + engine_version                        = (known after apply)
      + engine_version_actual                 = (known after apply)
      + hosted_zone_id                        = (known after apply)
      + id                                    = (known after apply)
      + identifier                            = (known after apply)
      + identifier_prefix                     = "database-for-webserver"
      + instance_class                        = "db.t3.micro"
      + iops                                  = (known after apply)
      + kms_key_id                            = (known after apply)
      + latest_restorable_time                = (known after apply)
      + license_model                         = (known after apply)
      + listener_endpoint                     = (known after apply)
      + maintenance_window                    = (known after apply)
      + master_user_secret                    = (known after apply)
      + master_user_secret_kms_key_id         = (known after apply)
      + monitoring_interval                   = 0
      + monitoring_role_arn                   = (known after apply)
      + multi_az                              = (known after apply)
      + nchar_character_set_name              = (known after apply)
      + network_type                          = (known after apply)
      + option_group_name                     = (known after apply)
      + parameter_group_name                  = (known after apply)
      + password                              = (sensitive value)
      + performance_insights_enabled          = false
      + performance_insights_kms_key_id       = (known after apply)
      + performance_insights_retention_period = (known after apply)
      + port                                  = (known after apply)
      + publicly_accessible                   = false
      + replica_mode                          = (known after apply)
      + replicas                              = (known after apply)
      + resource_id                           = (known after apply)
      + skip_final_snapshot                   = true
      + snapshot_identifier                   = (known after apply)
      + status                                = (known after apply)
      + storage_throughput                    = (known after apply)
      + storage_type                          = (known after apply)
      + tags_all                              = (known after apply)
      + timezone                              = (known after apply)
      + username                              = (sensitive value)
      + vpc_security_group_ids                = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + address = (known after apply)
  + port    = (known after apply)

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$
```
</details><br />

<details markdown=1>
<summary markdown="span">terraform apply</summary>

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_db_instance.example will be created
  + resource "aws_db_instance" "example" {
      + address                               = (known after apply)
      + allocated_storage                     = 10
      + apply_immediately                     = false
      + arn                                   = (known after apply)
      + auto_minor_version_upgrade            = true
      + availability_zone                     = (known after apply)
      + backup_retention_period               = (known after apply)
      + backup_target                         = (known after apply)
      + backup_window                         = (known after apply)
      + ca_cert_identifier                    = (known after apply)
      + character_set_name                    = (known after apply)
      + copy_tags_to_snapshot                 = false
      + db_name                               = "example_database"
      + db_subnet_group_name                  = (known after apply)
      + dedicated_log_volume                  = false
      + delete_automated_backups              = true
      + domain_fqdn                           = (known after apply)
      + endpoint                              = (known after apply)
      + engine                                = "mysql"
      + engine_lifecycle_support              = (known after apply)
      + engine_version                        = (known after apply)
      + engine_version_actual                 = (known after apply)
      + hosted_zone_id                        = (known after apply)
      + id                                    = (known after apply)
      + identifier                            = (known after apply)
      + identifier_prefix                     = "database-for-webserver"
      + instance_class                        = "db.t3.micro"
      + iops                                  = (known after apply)
      + kms_key_id                            = (known after apply)
      + latest_restorable_time                = (known after apply)
      + license_model                         = (known after apply)
      + listener_endpoint                     = (known after apply)
      + maintenance_window                    = (known after apply)
      + master_user_secret                    = (known after apply)
      + master_user_secret_kms_key_id         = (known after apply)
      + monitoring_interval                   = 0
      + monitoring_role_arn                   = (known after apply)
      + multi_az                              = (known after apply)
      + nchar_character_set_name              = (known after apply)
      + network_type                          = (known after apply)
      + option_group_name                     = (known after apply)
      + parameter_group_name                  = (known after apply)
      + password                              = (sensitive value)
      + performance_insights_enabled          = false
      + performance_insights_kms_key_id       = (known after apply)
      + performance_insights_retention_period = (known after apply)
      + port                                  = (known after apply)
      + publicly_accessible                   = false
      + replica_mode                          = (known after apply)
      + replicas                              = (known after apply)
      + resource_id                           = (known after apply)
      + skip_final_snapshot                   = true
      + snapshot_identifier                   = (known after apply)
      + status                                = (known after apply)
      + storage_throughput                    = (known after apply)
      + storage_type                          = (known after apply)
      + tags_all                              = (known after apply)
      + timezone                              = (known after apply)
      + username                              = (sensitive value)
      + vpc_security_group_ids                = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + address = (known after apply)
  + port    = (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_db_instance.example: Creating...
aws_db_instance.example: Still creating... [10s elapsed]
aws_db_instance.example: Still creating... [20s elapsed]
aws_db_instance.example: Still creating... [30s elapsed]
aws_db_instance.example: Still creating... [40s elapsed]
aws_db_instance.example: Still creating... [50s elapsed]
aws_db_instance.example: Still creating... [1m0s elapsed]
aws_db_instance.example: Still creating... [1m10s elapsed]
aws_db_instance.example: Still creating... [1m20s elapsed]
aws_db_instance.example: Still creating... [1m30s elapsed]
aws_db_instance.example: Still creating... [1m40s elapsed]
aws_db_instance.example: Still creating... [1m50s elapsed]
aws_db_instance.example: Still creating... [2m0s elapsed]
aws_db_instance.example: Still creating... [2m10s elapsed]
aws_db_instance.example: Still creating... [2m20s elapsed]
aws_db_instance.example: Still creating... [2m30s elapsed]
aws_db_instance.example: Still creating... [2m40s elapsed]
aws_db_instance.example: Still creating... [2m50s elapsed]
aws_db_instance.example: Still creating... [3m0s elapsed]
aws_db_instance.example: Still creating... [3m10s elapsed]
aws_db_instance.example: Still creating... [3m20s elapsed]
aws_db_instance.example: Still creating... [3m30s elapsed]
aws_db_instance.example: Still creating... [3m40s elapsed]
aws_db_instance.example: Still creating... [3m50s elapsed]
aws_db_instance.example: Still creating... [4m0s elapsed]
aws_db_instance.example: Still creating... [4m10s elapsed]
aws_db_instance.example: Still creating... [4m20s elapsed]
aws_db_instance.example: Still creating... [4m30s elapsed]
aws_db_instance.example: Still creating... [4m40s elapsed]
aws_db_instance.example: Creation complete after 4m45s [id=db-LIX6LC52KTB3IWDOAJG2NFEJ6Y]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

address = "database-for-webserver20240728000100847100000001.c7mu4ogq890e.us-east-1.rds.amazonaws.com"
port = 3306
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$
```
</details><br />

In the **AWS Management Console** we should see the RDS instance created

![]({{ site.baseurl }}/images/services/rds.png)

![]({{ site.baseurl }}/images/2024/07-27-Terraform-terraform_remote_state-Data-Source/03-RDS-instance-created.png)

## Create the webserver

Create the following files in the `webserver` directory. The directory structure should look like as following:

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation$ tree
.
├── data-store
│   ├── backend.tf
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── terraform-backend
│   ├── main.tf
│   └── terraform.tfstate
└── webserver
    ├── backend.tf
    ├── main.tf
    ├── outputs.tf
    ├── user-data.sh
    └── variables.tf

3 directories, 11 files
cloud_user@553b1e446c1c:~/terraform_file_isolation$
```

- `backend.tf`

```tf
terraform {
  backend "s3" {
    # Replace this with your bucket name!
    bucket         = "terraform-backend-20240727095931"
    key            = "webserver/terraform.tfstate"
    region         = "us-east-1"

    # Replace this with your DynamoDB table name!
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

### terraform_remote_state

The `terraform_remote_state` **Data Source** will allow us to pull information from the `data-store/terraform.tfstate` so we can use it to fill `db_address` & `db_port` in the EC2 instance.

In this case, will add `terraform_remote_state` in the `main.tf` file

- `main.tf`

```tf
data "terraform_remote_state" "db" {
  backend = "s3"

  config = {
    bucket = "terraform-backend-20240727095931"
    key    = "data-store/terraform.tfstate"
    region = "us-east-1"
  }
}
```

```tf
resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.instance.id]

  # Render the User Data script as a template
  user_data = templatefile("user-data.sh", {
    server_port = var.server_port
    db_address  = data.terraform_remote_state.db.outputs.address
    db_port     = data.terraform_remote_state.db.outputs.port
  })

  user_data_replace_on_change = true

  tags = {
    Name = "ubuntu1"
  }
}

resource "aws_security_group" "instance" {
  name = "terraform-example-instance"

  ingress {
    from_port   = var.server_port
    to_port     = var.server_port
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

- `variables.tf`

```tf
variable "server_port" {
  description = "The port the server will use for HTTP requests"
  type        = number
  default     = 8080
}
```

- `user-data.sh`

```bash
#!/bin/bash

cat > index.html <<EOF
<h1>Hello, World</h1>
<p>DB address: ${db_address}</p>
<p>DB port: ${db_port}</p>
EOF

nohup busybox httpd -f -p ${server_port} &
```

- `outputs.tf`

```bash
output "webserver_dns_name" {
  value       = aws_instance.example.public_dns
  description = "The domain name of the webserver"
}
```

<details markdown=1>
<summary markdown="span">terraform init</summary>

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$ terraform init
Initializing the backend...

Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.
Initializing provider plugins...
- terraform.io/builtin/terraform is built in to Terraform
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v5.60.0...
- Installed hashicorp/aws v5.60.0 (signed by HashiCorp)
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$
```
</details><br />


<details markdown=1>
<summary markdown="span">terraform plan</summary>

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$ terraform plan
data.terraform_remote_state.db: Reading...
data.terraform_remote_state.db: Read complete after 0s

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.example will be created
  + resource "aws_instance" "example" {
      + ami                                  = "ami-04a81a99f5ec58529"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = (known after apply)
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = (known after apply)
      + ebs_optimized                        = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + host_resource_group_arn              = (known after apply)
      + iam_instance_profile                 = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_lifecycle                   = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = (known after apply)
      + monitoring                           = (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + spot_instance_request_id             = (known after apply)
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Name" = "ubuntu1"
        }
      + tags_all                             = {
          + "Name" = "ubuntu1"
        }
      + tenancy                              = (known after apply)
      + user_data                            = "6c2c47f6520828af103b52145f540d381e66eb68"
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = true
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification (known after apply)

      + cpu_options (known after apply)

      + ebs_block_device (known after apply)

      + enclave_options (known after apply)

      + ephemeral_block_device (known after apply)

      + instance_market_options (known after apply)

      + maintenance_options (known after apply)

      + metadata_options (known after apply)

      + network_interface (known after apply)

      + private_dns_name_options (known after apply)

      + root_block_device (known after apply)
    }

  # aws_security_group.instance will be created
  + resource "aws_security_group" "instance" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = (known after apply)
      + id                     = (known after apply)
      + ingress                = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + from_port        = 8080
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 8080
                # (1 unchanged attribute hidden)
            },
        ]
      + name                   = "terraform-example-instance"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags_all               = (known after apply)
      + vpc_id                 = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$
```
</details><br />


<details markdown=1>
<summary markdown="span">terraform apply</summary>

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$ terraform apply
data.terraform_remote_state.db: Reading...
data.terraform_remote_state.db: Read complete after 0s

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.example will be created
  + resource "aws_instance" "example" {
      + ami                                  = "ami-04a81a99f5ec58529"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = (known after apply)
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = (known after apply)
      + ebs_optimized                        = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + host_resource_group_arn              = (known after apply)
      + iam_instance_profile                 = (known after apply)
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_lifecycle                   = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t2.micro"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = (known after apply)
      + monitoring                           = (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + source_dest_check                    = true
      + spot_instance_request_id             = (known after apply)
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Name" = "ubuntu1"
        }
      + tags_all                             = {
          + "Name" = "ubuntu1"
        }
      + tenancy                              = (known after apply)
      + user_data                            = "6c2c47f6520828af103b52145f540d381e66eb68"
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = true
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification (known after apply)

      + cpu_options (known after apply)

      + ebs_block_device (known after apply)

      + enclave_options (known after apply)

      + ephemeral_block_device (known after apply)

      + instance_market_options (known after apply)

      + maintenance_options (known after apply)

      + metadata_options (known after apply)

      + network_interface (known after apply)

      + private_dns_name_options (known after apply)

      + root_block_device (known after apply)
    }

  # aws_security_group.instance will be created
  + resource "aws_security_group" "instance" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = (known after apply)
      + id                     = (known after apply)
      + ingress                = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + from_port        = 8080
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 8080
                # (1 unchanged attribute hidden)
            },
        ]
      + name                   = "terraform-example-instance"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags_all               = (known after apply)
      + vpc_id                 = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_security_group.instance: Creating...
aws_security_group.instance: Creation complete after 3s [id=sg-081a9b0f536d8d462]
aws_instance.example: Creating...
aws_instance.example: Still creating... [10s elapsed]
aws_instance.example: Still creating... [20s elapsed]
aws_instance.example: Still creating... [30s elapsed]
aws_instance.example: Creation complete after 32s [id=i-004bec7f9e311d32f]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$
```
</details><br />

We see the EC2 instance has been created

![]({{ site.baseurl }}/images/services/ec2.png)

![]({{ site.baseurl }}/images/2024/07-27-Terraform-terraform_remote_state-Data-Source/04-EC2-instance-created.png)

Verification

```bash
$ curl http://ec2-54-166-238-6.compute-1.amazonaws.com:8080
<h1>Hello, World</h1>
<p>DB address: database-for-webserver20240728010153141100000001.c7qw6qcukxnz.us-east-1.rds.amazonaws.com</p>
<p>DB port: 3306</p>
$
```

Visiting http://ec2-54-166-238-6.compute-1.amazonaws.com:8080/ on a web browser

![]({{ site.baseurl }}/images/2024/07-27-Terraform-terraform_remote_state-Data-Source/05-http-verification.png)

## Remove the resources

Run `terraform destroy` on all environments to remove all the infrastructure created with Terraform

```bash
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$ terraform destroy
cloud_user@553b1e446c1c:~/terraform_file_isolation/webserver$ cd ../data-store/
cloud_user@553b1e446c1c:~/terraform_file_isolation/data-store$ terraform destroy
cloud_user@553b1e446c1c:~/terraform_file_isolation/terraform-backend$ cd ../terraform-backend
cloud_user@553b1e446c1c:~/terraform_file_isolation/terraform-backend$ terraform destroy
```

## References

- [Terraform: Up & Running By Yevgeniy Brikman](https://www.terraformupandrunning.com/)
- [The terraform_remote_state Data Source](https://developer.hashicorp.com/terraform/language/state/remote-state-data)
- [templatefile Function](https://developer.hashicorp.com/terraform/language/functions/templatefile)
