---
title: Terraform Workspaces
date: 2024-07-25 16:00:00 -0700
categories: [TERRAFORM]
tags: [terraform]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/terraform_logo.png)

## Introduction

Terraform workspaces allow you to use different *Terraform state files* in the same directory. This allows you do have different environments.

**IMPORTANT:** The use of workspaces is discouraged, a better approach is to do **isolation via file layout**.

## Proof of Concept

We will use different workspaces to spin-up an EC2 instance.
We are going to be using a local backend.

As a firs step, create a new directory, move to it and create a `main.tf` file that will spin-up an EC2 instance.

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"
  
  tags = {
    Name = "ubuntu1"
  }
}
```

<details markdown=1>
<summary markdown="span">terraform init</summary>

```bash
cloud_user@553b1e446c1c:~$ mkdir workspaces_terraform_poc
cloud_user@553b1e446c1c:~$ cd workspaces_terraform_poc
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ vim main.tf
icloud_user@553b1e446c1c:~/workspaces_terraform_poc$
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ ll
total 12
drwxrwxr-x  2 cloud_user cloud_user 4096 Jul 25 23:10 ./
drwxr-x--- 16 cloud_user cloud_user 4096 Jul 25 23:10 ../
-rw-rw-r--  1 cloud_user cloud_user  191 Jul 25 23:10 main.tf
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform init
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
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ ll
total 20
drwxrwxr-x  3 cloud_user cloud_user 4096 Jul 25 23:10 ./
drwxr-x--- 16 cloud_user cloud_user 4096 Jul 25 23:10 ../
drwxr-xr-x  3 cloud_user cloud_user 4096 Jul 25 23:10 .terraform/
-rw-r--r--  1 cloud_user cloud_user 1377 Jul 25 23:10 .terraform.lock.hcl
-rw-rw-r--  1 cloud_user cloud_user  191 Jul 25 23:10 main.tf
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$
```
</details><br />

## List the Workspaces

```bash
terraform workspace list
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace list
* default

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$
```
</details><br />

## Run Terraform on the default workspace

<details markdown=1>
<summary markdown="span">terraform apply</summary>

```bash
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform apply

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
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
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

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.example: Creating...
aws_instance.example: Still creating... [10s elapsed]
aws_instance.example: Still creating... [20s elapsed]
aws_instance.example: Still creating... [30s elapsed]
aws_instance.example: Creation complete after 33s [id=i-0819f0b5fe8304e96]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ ll
total 28
drwxrwxr-x  3 cloud_user cloud_user 4096 Jul 25 23:14 ./
drwxr-x--- 16 cloud_user cloud_user 4096 Jul 25 23:10 ../
drwxr-xr-x  3 cloud_user cloud_user 4096 Jul 25 23:10 .terraform/
-rw-r--r--  1 cloud_user cloud_user 1377 Jul 25 23:10 .terraform.lock.hcl
-rw-rw-r--  1 cloud_user cloud_user  191 Jul 25 23:10 main.tf
-rw-rw-r--  1 cloud_user cloud_user 4833 Jul 25 23:14 terraform.tfstate
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$
```
</details><br />

The EC2 instance was created

![]({{ site.baseurl }}/images/2024/07-25-Terraform-workspaces/01-EC2-instance-created.png)

## Creating a new workspace

We will create a new workspace named **workspace_test**

```bash
terraform workspace new
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace new workspace_test
Created and switched to workspace "workspace_test"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace list
  default
* workspace_test

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace show
workspace_test
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ ll
total 32
drwxrwxr-x  4 cloud_user cloud_user 4096 Jul 25 23:18 ./
drwxr-x--- 16 cloud_user cloud_user 4096 Jul 25 23:10 ../
drwxr-xr-x  3 cloud_user cloud_user 4096 Jul 25 23:18 .terraform/
-rw-r--r--  1 cloud_user cloud_user 1377 Jul 25 23:10 .terraform.lock.hcl
-rw-rw-r--  1 cloud_user cloud_user  191 Jul 25 23:10 main.tf
-rw-rw-r--  1 cloud_user cloud_user 4833 Jul 25 23:14 terraform.tfstate
drwxr-xr-x  3 cloud_user cloud_user 4096 Jul 25 23:18 terraform.tfstate.d/
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$
```
</details><br />

## Run Terraform on the workspace named workspace_test

```bash
terraform init
terraform apply
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform apply

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
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
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

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions in workspace "workspace_test"?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.example: Creating...
aws_instance.example: Still creating... [10s elapsed]
aws_instance.example: Still creating... [20s elapsed]
aws_instance.example: Still creating... [30s elapsed]
aws_instance.example: Creation complete after 32s [id=i-0eea388d1345423b5]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$
```
</details><br />

A second EC2 instance was created.

![]({{ site.baseurl }}/images/2024/07-25-Terraform-workspaces/02-Another-EC2-instance-created.png)

Since `terraform apply` was run on a different environment, a second `terraform.tfstate` file was used and Terraform created a new EC2 instance instead of realizing it already have that one created before hand.

The second `terraform.tfstate` file was created in the `terraform.tfstate.d/<WORKSPACE_NAME>/` folder.

```bash
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ ll
total 32
drwxrwxr-x  4 cloud_user cloud_user 4096 Jul 25 23:18 ./
drwxr-x--- 16 cloud_user cloud_user 4096 Jul 25 23:10 ../
drwxr-xr-x  3 cloud_user cloud_user 4096 Jul 25 23:18 .terraform/
-rw-r--r--  1 cloud_user cloud_user 1377 Jul 25 23:10 .terraform.lock.hcl
-rw-rw-r--  1 cloud_user cloud_user  191 Jul 25 23:10 main.tf
-rw-rw-r--  1 cloud_user cloud_user 4833 Jul 25 23:14 terraform.tfstate
drwxr-xr-x  3 cloud_user cloud_user 4096 Jul 25 23:18 terraform.tfstate.d/
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ cd terraform.tfstate.d/
cloud_user@553b1e446c1c:~/workspaces_terraform_poc/terraform.tfstate.d$ ll
total 12
drwxr-xr-x 3 cloud_user cloud_user 4096 Jul 25 23:18 ./
drwxrwxr-x 4 cloud_user cloud_user 4096 Jul 25 23:18 ../
drwxr-xr-x 2 cloud_user cloud_user 4096 Jul 25 23:20 workspace_test/
cloud_user@553b1e446c1c:~/workspaces_terraform_poc/terraform.tfstate.d$ cd workspace_test/
cloud_user@553b1e446c1c:~/workspaces_terraform_poc/terraform.tfstate.d/workspace_test$ ll
total 16
drwxr-xr-x 2 cloud_user cloud_user 4096 Jul 25 23:20 ./
drwxr-xr-x 3 cloud_user cloud_user 4096 Jul 25 23:18 ../
-rw-rw-r-- 1 cloud_user cloud_user 4825 Jul 25 23:20 terraform.tfstate
cloud_user@553b1e446c1c:~/workspaces_terraform_poc/terraform.tfstate.d/workspace_test$
```

## Creating a third workspace

We will run `terraform apply` again on a third workspace.

**IMPORTANT:** Move back to the main directory where you have the HCL file `main.tf`. Otherwise the "workspace" directory structure will be created from the sub-directory where you are located.

```bash
cloud_user@553b1e446c1c:~/workspaces_terraform_poc/terraform.tfstate.d/workspace_test$ cd ..
cloud_user@553b1e446c1c:~/workspaces_terraform_poc/terraform.tfstate.d$ cd ..
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$
```

Now run `terraform workspace new`

```bash
terraform workspace new
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace new third_workspace
Created and switched to workspace "third_workspace"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace list
  default
* third_workspace
  workspace_test

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace show
third_workspace
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$
```
</details><br />

An again `terraform init` & `terraform apply`

```bash
terraform init
terraform apply
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform init
Initializing the backend...
Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file
- Using previously-installed hashicorp/aws v5.60.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform apply

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
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
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

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions in workspace "third_workspace"?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.example: Creating...
aws_instance.example: Still creating... [10s elapsed]
aws_instance.example: Still creating... [20s elapsed]
aws_instance.example: Still creating... [30s elapsed]
aws_instance.example: Creation complete after 32s [id=i-004efc41442cb0e25]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$
```
</details><br />

A third EC2 instance was created:

![]({{ site.baseurl }}/images/2024/07-25-Terraform-workspaces/03-A-third-EC2-instance-created.png)

## The clean-up

We will need to run `terraform destroy` per workspace, then move to the `default` workspace` and finally delete other workspaces created.

```bash
terraform workspace list
terraform destroy
terraform workspace select workspace_test
terraform destroy
terraform workspace select default
terraform destroy
terraform workspace delete third_workspace
terraform workspace delete  workspace_test
terraform list
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace list
  default
* third_workspace
  workspace_test

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform destroy
aws_instance.example: Refreshing state... [id=i-004efc41442cb0e25]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_instance.example will be destroyed
  - resource "aws_instance" "example" {
      - ami                                  = "ami-04a81a99f5ec58529" -> null
      - arn                                  = "arn:aws:ec2:us-east-1:XXXXXXXXXXXX:instance/i-004efc41442cb0e25" -> null
      - associate_public_ip_address          = true -> null
      - availability_zone                    = "us-east-1d" -> null
      - cpu_core_count                       = 1 -> null
      - cpu_threads_per_core                 = 1 -> null
      - disable_api_stop                     = false -> null
      - disable_api_termination              = false -> null
      - ebs_optimized                        = false -> null
      - get_password_data                    = false -> null
      - hibernation                          = false -> null
      - id                                   = "i-004efc41442cb0e25" -> null
      - instance_initiated_shutdown_behavior = "stop" -> null
      - instance_state                       = "running" -> null
      - instance_type                        = "t2.micro" -> null
      - ipv6_address_count                   = 0 -> null
      - ipv6_addresses                       = [] -> null
      - monitoring                           = false -> null
      - placement_partition_number           = 0 -> null
      - primary_network_interface_id         = "eni-0c8283a6071a36980" -> null
      - private_dns                          = "ip-172-31-40-67.ec2.internal" -> null
      - private_ip                           = "172.31.40.67" -> null
      - public_dns                           = "ec2-54-162-21-71.compute-1.amazonaws.com" -> null
      - public_ip                            = "54.162.21.71" -> null
      - secondary_private_ips                = [] -> null
      - security_groups                      = [
          - "default",
        ] -> null
      - source_dest_check                    = true -> null
      - subnet_id                            = "subnet-0d4ee4abd7af71880" -> null
      - tags                                 = {
          - "Name" = "ubuntu1"
        } -> null
      - tags_all                             = {
          - "Name" = "ubuntu1"
        } -> null
      - tenancy                              = "default" -> null
      - user_data_replace_on_change          = false -> null
      - vpc_security_group_ids               = [
          - "sg-02e00a1d80b93788d",
        ] -> null
        # (8 unchanged attributes hidden)

      - capacity_reservation_specification {
          - capacity_reservation_preference = "open" -> null
        }

      - cpu_options {
          - core_count       = 1 -> null
          - threads_per_core = 1 -> null
            # (1 unchanged attribute hidden)
        }

      - credit_specification {
          - cpu_credits = "standard" -> null
        }

      - enclave_options {
          - enabled = false -> null
        }

      - maintenance_options {
          - auto_recovery = "default" -> null
        }

      - metadata_options {
          - http_endpoint               = "enabled" -> null
          - http_protocol_ipv6          = "disabled" -> null
          - http_put_response_hop_limit = 2 -> null
          - http_tokens                 = "required" -> null
          - instance_metadata_tags      = "disabled" -> null
        }

      - private_dns_name_options {
          - enable_resource_name_dns_a_record    = false -> null
          - enable_resource_name_dns_aaaa_record = false -> null
          - hostname_type                        = "ip-name" -> null
        }

      - root_block_device {
          - delete_on_termination = true -> null
          - device_name           = "/dev/sda1" -> null
          - encrypted             = false -> null
          - iops                  = 3000 -> null
          - tags                  = {} -> null
          - tags_all              = {} -> null
          - throughput            = 125 -> null
          - volume_id             = "vol-0e829992d08156dc2" -> null
          - volume_size           = 8 -> null
          - volume_type           = "gp3" -> null
            # (1 unchanged attribute hidden)
        }
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources in workspace "third_workspace"?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_instance.example: Destroying... [id=i-004efc41442cb0e25]
aws_instance.example: Still destroying... [id=i-004efc41442cb0e25, 10s elapsed]
aws_instance.example: Still destroying... [id=i-004efc41442cb0e25, 20s elapsed]
aws_instance.example: Still destroying... [id=i-004efc41442cb0e25, 30s elapsed]
aws_instance.example: Still destroying... [id=i-004efc41442cb0e25, 40s elapsed]
aws_instance.example: Destruction complete after 40s

Destroy complete! Resources: 1 destroyed.
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace select workspace_test
Switched to workspace "workspace_test".
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform destroy
aws_instance.example: Refreshing state... [id=i-0eea388d1345423b5]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_instance.example will be destroyed
  - resource "aws_instance" "example" {
      - ami                                  = "ami-04a81a99f5ec58529" -> null
      - arn                                  = "arn:aws:ec2:us-east-1:XXXXXXXXXXXX:instance/i-0eea388d1345423b5" -> null
      - associate_public_ip_address          = true -> null
      - availability_zone                    = "us-east-1d" -> null
      - cpu_core_count                       = 1 -> null
      - cpu_threads_per_core                 = 1 -> null
      - disable_api_stop                     = false -> null
      - disable_api_termination              = false -> null
      - ebs_optimized                        = false -> null
      - get_password_data                    = false -> null
      - hibernation                          = false -> null
      - id                                   = "i-0eea388d1345423b5" -> null
      - instance_initiated_shutdown_behavior = "stop" -> null
      - instance_state                       = "running" -> null
      - instance_type                        = "t2.micro" -> null
      - ipv6_address_count                   = 0 -> null
      - ipv6_addresses                       = [] -> null
      - monitoring                           = false -> null
      - placement_partition_number           = 0 -> null
      - primary_network_interface_id         = "eni-0e864556efbf0a6e6" -> null
      - private_dns                          = "ip-172-31-40-35.ec2.internal" -> null
      - private_ip                           = "172.31.40.35" -> null
      - public_dns                           = "ec2-52-55-44-89.compute-1.amazonaws.com" -> null
      - public_ip                            = "52.55.44.89" -> null
      - secondary_private_ips                = [] -> null
      - security_groups                      = [
          - "default",
        ] -> null
      - source_dest_check                    = true -> null
      - subnet_id                            = "subnet-0d4ee4abd7af71880" -> null
      - tags                                 = {
          - "Name" = "ubuntu1"
        } -> null
      - tags_all                             = {
          - "Name" = "ubuntu1"
        } -> null
      - tenancy                              = "default" -> null
      - user_data_replace_on_change          = false -> null
      - vpc_security_group_ids               = [
          - "sg-02e00a1d80b93788d",
        ] -> null
        # (8 unchanged attributes hidden)

      - capacity_reservation_specification {
          - capacity_reservation_preference = "open" -> null
        }

      - cpu_options {
          - core_count       = 1 -> null
          - threads_per_core = 1 -> null
            # (1 unchanged attribute hidden)
        }

      - credit_specification {
          - cpu_credits = "standard" -> null
        }

      - enclave_options {
          - enabled = false -> null
        }

      - maintenance_options {
          - auto_recovery = "default" -> null
        }

      - metadata_options {
          - http_endpoint               = "enabled" -> null
          - http_protocol_ipv6          = "disabled" -> null
          - http_put_response_hop_limit = 2 -> null
          - http_tokens                 = "required" -> null
          - instance_metadata_tags      = "disabled" -> null
        }

      - private_dns_name_options {
          - enable_resource_name_dns_a_record    = false -> null
          - enable_resource_name_dns_aaaa_record = false -> null
          - hostname_type                        = "ip-name" -> null
        }

      - root_block_device {
          - delete_on_termination = true -> null
          - device_name           = "/dev/sda1" -> null
          - encrypted             = false -> null
          - iops                  = 3000 -> null
          - tags                  = {} -> null
          - tags_all              = {} -> null
          - throughput            = 125 -> null
          - volume_id             = "vol-0e8a29f703246554f" -> null
          - volume_size           = 8 -> null
          - volume_type           = "gp3" -> null
            # (1 unchanged attribute hidden)
        }
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources in workspace "workspace_test"?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_instance.example: Destroying... [id=i-0eea388d1345423b5]
aws_instance.example: Still destroying... [id=i-0eea388d1345423b5, 10s elapsed]
aws_instance.example: Still destroying... [id=i-0eea388d1345423b5, 20s elapsed]
aws_instance.example: Still destroying... [id=i-0eea388d1345423b5, 30s elapsed]
aws_instance.example: Still destroying... [id=i-0eea388d1345423b5, 40s elapsed]
aws_instance.example: Destruction complete after 40s

Destroy complete! Resources: 1 destroyed.
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace select default
Switched to workspace "default".
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform destroy
aws_instance.example: Refreshing state... [id=i-0819f0b5fe8304e96]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_instance.example will be destroyed
  - resource "aws_instance" "example" {
      - ami                                  = "ami-04a81a99f5ec58529" -> null
      - arn                                  = "arn:aws:ec2:us-east-1:XXXXXXXXXXXX:instance/i-0819f0b5fe8304e96" -> null
      - associate_public_ip_address          = true -> null
      - availability_zone                    = "us-east-1d" -> null
      - cpu_core_count                       = 1 -> null
      - cpu_threads_per_core                 = 1 -> null
      - disable_api_stop                     = false -> null
      - disable_api_termination              = false -> null
      - ebs_optimized                        = false -> null
      - get_password_data                    = false -> null
      - hibernation                          = false -> null
      - id                                   = "i-0819f0b5fe8304e96" -> null
      - instance_initiated_shutdown_behavior = "stop" -> null
      - instance_state                       = "running" -> null
      - instance_type                        = "t2.micro" -> null
      - ipv6_address_count                   = 0 -> null
      - ipv6_addresses                       = [] -> null
      - monitoring                           = false -> null
      - placement_partition_number           = 0 -> null
      - primary_network_interface_id         = "eni-0b9afd80bc42d7e39" -> null
      - private_dns                          = "ip-172-31-35-116.ec2.internal" -> null
      - private_ip                           = "172.31.35.116" -> null
      - public_dns                           = "ec2-34-235-142-167.compute-1.amazonaws.com" -> null
      - public_ip                            = "34.235.142.167" -> null
      - secondary_private_ips                = [] -> null
      - security_groups                      = [
          - "default",
        ] -> null
      - source_dest_check                    = true -> null
      - subnet_id                            = "subnet-0d4ee4abd7af71880" -> null
      - tags                                 = {
          - "Name" = "ubuntu1"
        } -> null
      - tags_all                             = {
          - "Name" = "ubuntu1"
        } -> null
      - tenancy                              = "default" -> null
      - user_data_replace_on_change          = false -> null
      - vpc_security_group_ids               = [
          - "sg-02e00a1d80b93788d",
        ] -> null
        # (8 unchanged attributes hidden)

      - capacity_reservation_specification {
          - capacity_reservation_preference = "open" -> null
        }

      - cpu_options {
          - core_count       = 1 -> null
          - threads_per_core = 1 -> null
            # (1 unchanged attribute hidden)
        }

      - credit_specification {
          - cpu_credits = "standard" -> null
        }

      - enclave_options {
          - enabled = false -> null
        }

      - maintenance_options {
          - auto_recovery = "default" -> null
        }

      - metadata_options {
          - http_endpoint               = "enabled" -> null
          - http_protocol_ipv6          = "disabled" -> null
          - http_put_response_hop_limit = 2 -> null
          - http_tokens                 = "required" -> null
          - instance_metadata_tags      = "disabled" -> null
        }

      - private_dns_name_options {
          - enable_resource_name_dns_a_record    = false -> null
          - enable_resource_name_dns_aaaa_record = false -> null
          - hostname_type                        = "ip-name" -> null
        }

      - root_block_device {
          - delete_on_termination = true -> null
          - device_name           = "/dev/sda1" -> null
          - encrypted             = false -> null
          - iops                  = 3000 -> null
          - tags                  = {} -> null
          - tags_all              = {} -> null
          - throughput            = 125 -> null
          - volume_id             = "vol-0391d5f2b0afd3f70" -> null
          - volume_size           = 8 -> null
          - volume_type           = "gp3" -> null
            # (1 unchanged attribute hidden)
        }
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_instance.example: Destroying... [id=i-0819f0b5fe8304e96]
aws_instance.example: Still destroying... [id=i-0819f0b5fe8304e96, 10s elapsed]
aws_instance.example: Still destroying... [id=i-0819f0b5fe8304e96, 20s elapsed]
aws_instance.example: Still destroying... [id=i-0819f0b5fe8304e96, 30s elapsed]
aws_instance.example: Still destroying... [id=i-0819f0b5fe8304e96, 40s elapsed]
aws_instance.example: Still destroying... [id=i-0819f0b5fe8304e96, 50s elapsed]
aws_instance.example: Destruction complete after 51s

Destroy complete! Resources: 1 destroyed.
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace delete third_workspace
Deleted workspace "third_workspace"!
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace delete  workspace_test
Deleted workspace "workspace_test"!
cloud_user@553b1e446c1c:~/workspaces_terraform_poc$

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$ terraform workspace list
* default

cloud_user@553b1e446c1c:~/workspaces_terraform_poc$
```
</details><br />

All three EC2 instances were terminated.

![]({{ site.baseurl }}/images/2024/07-25-Terraform-workspaces/04-All-EC2-instances-terminated.png)

## References

- [Terraform: Up & Running By Yevgeniy Brikman](https://www.terraformupandrunning.com/)
