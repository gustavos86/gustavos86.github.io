---
title: Terraform 101 Getting Started
date: 2024-07-22 20:00:00 -0700
categories: [TERRAFORM]
tags: [terraform, terraform aws ec2]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/terraform_logo.png)

## Installing Terraform

Covered in: [Installing Terraform on Ubuntu 24.04]({% post_url 2024-07-21-Terraform-install %})

## Setup the AWS cretendials

There are at least 2 options that we can use to setup the credentials to the AWS account:

- Using env variables
- Using AWS configuration file

### Using env variables

```bash
export AWS_ACCESS_KEY_ID=(your access key id)
export AWS_SECRET_ACCESS_KEY=(your secret access key)
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ export AWS_ACCESS_KEY_ID=A...
cloud_user@553b1e446c1c:~$ export AWS_SECRET_ACCESS_KEY=5s...
cloud_user@553b1e446c1c:~$
cloud_user@553b1e446c1c:~$ echo $AWS_ACCESS_KEY_ID
A...
cloud_user@553b1e446c1c:~$ echo $AWS_SECRET_ACCESS_KEY
5s...
cloud_user@553b1e446c1c:~$
```
</details><br />

### Using AWS configuration file

```bash
aws configure
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ aws configure
AWS Access Key ID [None]: A...
AWS Secret Access Key [None]: 5s...
Default region name [None]: us-east-1
Default output format [None]: json
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ cat ~/.aws/config
[default]
region = us-east-1
output = json
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ cat ~/.aws/credentials
[default]
aws_access_key_id = A...
aws_secret_access_key = 5s...
cloud_user@553b1e446c1c:~$
```
</details><br />

## Creating the Terraform script

Terraform code is written in the *HashiCorp Configuration Language* (HCL) in files with the extension `.tf`

### Provider

Terraform can create infrastructure across a wide variety of platforms, or what it calls **providers**, including AWS, Azure, Google Cloud, DigitalOcean, etc.

Create a new folder. In the new folder, create a new file named `main.tf`.

This tells Terraform that you are going to be using **AWS** as the **provider** and that you want to deploy your infrastructure into the `us-east-1` region.

```tf
provider "aws" {
  region = "us-east-1"
}
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ mkdir ec2_instances
cloud_user@553b1e446c1c:~$ cd ec2_instances/

cloud_user@553b1e446c1c:~/ec2_instances$ vim main.tf
cloud_user@553b1e446c1c:~/ec2_instances$

cloud_user@553b1e446c1c:~/ec2_instances$ cat main.tf
provider "aws" {
  region = "us-east-1"
}
cloud_user@553b1e446c1c:~/ec2_instances$
```
</details><br />

## Resource

The general syntax for creating a **resource** in Terraform is as follows:

```tf
resource "<PROVIDER>_<TYPE>" "<NAME>" {
  [CONFIG ...]
}
```

- **PROVIDER** is the name of a provider (e.g., aws)
- **TYPE** is the type of resource to create in that provider (e.g., instance)
- **NAME** is an identifier you can use throughout the Terraform code to refer to this resource (e.g., my_instance)
- **CONFIG** consists of one or more arguments that are specific to that resource

To deploy a single EC2 instance:

```tf
resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"
}
```

**NOTE:** The AMI is AWS region dependent. So, it is different on every region even for the same OS. This is the one for **Ubuntu Server 24.04 LTS** in **us-east-1**.

![]({{ site.baseurl }}/images/2024/07-22-Terraform_101_getting_started/01-Ubuntu-Server-2004-AMI-in-us-east-1.png)

## Arguments

The `aws_instance` resource supports many different `arguments`. Here the required ones. All other ones can be found in the Terraform documentation [Resource: aws_instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance).

- `ami` The Amazon Machine Image (AMI) to run on the EC2 Instance.
- `instance_type` The type of EC2 Instance to run. 

## terraform init

At this point, the `main.tf` file should look like:

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"
}
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ vim main.tf
cloud_user@553b1e446c1c:~/ec2_instances$
cloud_user@553b1e446c1c:~/ec2_instances$ pwd
/home/cloud_user/ec2_instances
cloud_user@553b1e446c1c:~/ec2_instances$ cat main.tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"
}
cloud_user@553b1e446c1c:~/ec2_instances$
```
</details><br />

While located in the folder run `terraform init`.

`terraform init` will scan the HCL code, and download the code of the specified providers into the `.terraform` folder.

**NOTES:**
- The `.terraform` folder is the scratch directory and should be added to `.gitignore` file if the code is uploaded to a control version system like GitHub.
- Terraform will also record information about the provider code it downloaded into a `.terraform.lock.hcl` file. This file should be included in the version control repository.

```bash
terraform init
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ terraform init
Initializing the backend...
Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v5.59.0...
- Installed hashicorp/aws v5.59.0 (signed by HashiCorp)
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
cloud_user@553b1e446c1c:~/ec2_instances$
```
</details><br />

## terraform plan

The `terraform plan` command lets you see what Terraform will do before actually making any changes. Useful for quick sanity checks and during code reviews.

```bash
terraform plan
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ terraform plan

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
      + tags_all                             = (known after apply)
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

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
cloud_user@553b1e446c1c:~/ec2_instances$
```
</details><br />

## terraform apply

To execute the terraform script, use `terraform apply`

```bash
terraform apply
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ terraform apply

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
      + tags_all                             = (known after apply)
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
aws_instance.example: Creation complete after 33s [id=i-0799c62e3c25356a3]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
cloud_user@553b1e446c1c:~/ec2_instances$
```
</details><br />

The EC2 instance has been created!

![]({{ site.baseurl }}/images/2024/07-22-Terraform_101_getting_started/02-EC2-instance-created.png)

## tag argument

According to the documentation:

> tags - (Optional) Map of tags to assign to the resource. Note that these tags apply to the instance and not block storage devices. If configured with a provider default_tags configuration block present, tags with matching keys will overwrite those defined at the provider-level.

The `tag` argument can be used to add a name to the EC2 instance. add the `tag` to the `main.tf` file.

The `main.tf` file should looks like as following now:

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"
  
  tags = {
    Name = "ubuntu1-named-with-terraform"
  }
}
```

```bash
terraform apply
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ cat main.tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"

  tags = {
    Name = "ubuntu1-named-with-terraform"
  }
}
cloud_user@553b1e446c1c:~/ec2_instances$

cloud_user@553b1e446c1c:~/ec2_instances$ terraform apply
aws_instance.example: Refreshing state... [id=i-0799c62e3c25356a3]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # aws_instance.example will be updated in-place
  ~ resource "aws_instance" "example" {
        id                                   = "i-0799c62e3c25356a3"
      ~ tags                                 = {
          + "Name" = "ubuntu1-named-with-terraform"
        }
      ~ tags_all                             = {
          + "Name" = "ubuntu1-named-with-terraform"
        }
        # (38 unchanged attributes hidden)

        # (8 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.example: Modifying... [id=i-0799c62e3c25356a3]
aws_instance.example: Modifications complete after 2s [id=i-0799c62e3c25356a3]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
cloud_user@553b1e446c1c:~/ec2_instances$
```
</details><br />

The EC2 instance now shows up with a name in the **AWS Management Console**

![]({{ site.baseurl }}/images/2024/07-22-Terraform_101_getting_started/03-EC2-instance-created-with-tag.png)

## Git

When using version control for the terraform code, make sure to add these files to `.gitignore`.

```
.terraform
*.tfstate
*.tfstate.backup
```

Besides `main.tf`, you should add `.terraform.lock.hcl` to the repo.

```
git add main.tf .terraform.lock.hcl .gitignore
```

## user_data parameter

According to the documentation:

> user_data - (Optional) User data to provide when launching the instance. Do not pass gzip-compressed data via this argument; see user_data_base64 instead. Updates to this field will trigger a stop/start of the EC2 instance by default. If the user_data_replace_on_change is set then updates to this field will trigger a destroy and recreate.


```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p 8080 &
              EOF

  user_data_replace_on_change = true

  tags = {
    Name = "ubuntu1-named-with-terraform"
  }
}
```

- The `<<-EOF` and `EOF` are Terraform’s heredoc syntax, which allows you to create multiline strings without having to insert `\n` characters all over the place.
- The `user_data_replace_on_change` parameter is set to `true` so that when you change the `user_data` parameter and run apply, Terraform will terminate the original instance and launch a totally new one. Terraform’s default behavior is to update the original instance in place, but since User Data runs only on the very first boot, and your original instance already went through that boot process, you need to force the creation of a new instance to ensure your new User Data script actually gets executed.

## resource aws_security_group

Creates a Security Group (SG)

```tf
resource "aws_security_group" "instance" {
  name = "terraform-example-instance"

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

## expressions

An expression in Terraform is anything that returns a value. The simplest type of expressions are literals, such as strings (e.g., `"ami-0fb653ca2d3203ac1"`) and numbers (e.g., `5`).

A **reference expresssion** has the following syntax:

```
<PROVIDER>_<TYPE>.<NAME>.<ATTRIBUTE>
```

- `PROVIDER` is the name of the provider (e.g., `aws`)
- `TYPE` is the type of resource (e.g., `security_group`)
- `NAME` is the name of that resource (e.g., the security group is named `"instance"`)
- `ATTRIBUTE` is either one of the arguments of that resource (e.g., `name`), or one of the attributes *exported* by the resource

The security group exports an attribute called `id`, so the expression to reference it will look like this:

```
aws_security_group.instance.id
```

The `main.tf` should now look like this:

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_security_group" "instance" {
  name = "terraform-example-instance"

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.instance.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p 8080 &
              EOF

  user_data_replace_on_change = true

  tags = {
    Name = "ubuntu1-named-with-terraform"
  }
}
```

## implicit dependency

Defines the order in which the resources should be created since some resources must refer to others already created. For example a Security Group should be created before an EC2 instance can reference it.

```bash
terraform graph
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ terraform graph
digraph G {
  rankdir = "RL";
  node [shape = rect, fontname = "sans-serif"];
  "aws_instance.example" [label="aws_instance.example"];
  "aws_security_group.instance" [label="aws_security_group.instance"];
  "aws_instance.example" -> "aws_security_group.instance";
}
cloud_user@553b1e446c1c:~/ec2_instances$
```
</details><br />

## terraform apply

Applyting the latest version of the `main.tf` file:

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_security_group" "instance" {
  name = "terraform-example-instance"

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.instance.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p 8080 &
              EOF

  user_data_replace_on_change = true

  tags = {
    Name = "ubuntu1-named-with-terraform"
  }
}
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ cat main.tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_security_group" "instance" {
  name = "terraform-example-instance"

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.instance.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p 8080 &
              EOF

  user_data_replace_on_change = true

  tags = {
    Name = "ubuntu1-named-with-terraform"
  }
}
cloud_user@553b1e446c1c:~/ec2_instances$ terraform apply
aws_instance.example: Refreshing state... [id=i-0799c62e3c25356a3]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # aws_instance.example must be replaced
-/+ resource "aws_instance" "example" {
      ~ arn                                  = "arn:aws:ec2:us-east-1:381492126797:instance/i-0799c62e3c25356a3" -> (known after apply)
      ~ associate_public_ip_address          = true -> (known after apply)
      ~ availability_zone                    = "us-east-1a" -> (known after apply)
      ~ cpu_core_count                       = 1 -> (known after apply)
      ~ cpu_threads_per_core                 = 1 -> (known after apply)
      ~ disable_api_stop                     = false -> (known after apply)
      ~ disable_api_termination              = false -> (known after apply)
      ~ ebs_optimized                        = false -> (known after apply)
      - hibernation                          = false -> null
      + host_id                              = (known after apply)
      + host_resource_group_arn              = (known after apply)
      + iam_instance_profile                 = (known after apply)
      ~ id                                   = "i-0799c62e3c25356a3" -> (known after apply)
      ~ instance_initiated_shutdown_behavior = "stop" -> (known after apply)
      + instance_lifecycle                   = (known after apply)
      ~ instance_state                       = "running" -> (known after apply)
      ~ ipv6_address_count                   = 0 -> (known after apply)
      ~ ipv6_addresses                       = [] -> (known after apply)
      + key_name                             = (known after apply)
      ~ monitoring                           = false -> (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      ~ placement_partition_number           = 0 -> (known after apply)
      ~ primary_network_interface_id         = "eni-067824cab2cf93a89" -> (known after apply)
      ~ private_dns                          = "ip-172-31-83-133.ec2.internal" -> (known after apply)
      ~ private_ip                           = "172.31.83.133" -> (known after apply)
      ~ public_dns                           = "ec2-3-93-15-248.compute-1.amazonaws.com" -> (known after apply)
      ~ public_ip                            = "3.93.15.248" -> (known after apply)
      ~ secondary_private_ips                = [] -> (known after apply)
      ~ security_groups                      = [
          - "default",
        ] -> (known after apply)
      + spot_instance_request_id             = (known after apply)
      ~ subnet_id                            = "subnet-047d0842ca61eda50" -> (known after apply)
        tags                                 = {
            "Name" = "ubuntu1-named-with-terraform"
        }
      ~ tenancy                              = "default" -> (known after apply)
      + user_data                            = "c765373c563b260626d113c4a56a46e8a8c5ca33" # forces replacement
      + user_data_base64                     = (known after apply)
      ~ user_data_replace_on_change          = false -> true
      ~ vpc_security_group_ids               = [
          - "sg-06f073b6381b13ee6",
        ] -> (known after apply)
        # (5 unchanged attributes hidden)

      ~ capacity_reservation_specification {
          + ami                                  = (known after apply)
          + arn                                  = (known after apply)
          + associate_public_ip_address          = (known after apply)
          + availability_zone                    = (known after apply)
          + cpu_core_count                       = (known after apply)
          + cpu_threads_per_core                 = (known after apply)
          + disable_api_stop                     = (known after apply)
          + disable_api_termination              = (known after apply)
          + ebs_optimized                        = (known after apply)
          + get_password_data                    = (known after apply)
          + hibernation                          = (known after apply)
          + host_id                              = (known after apply)
          + host_resource_group_arn              = (known after apply)
          + iam_instance_profile                 = (known after apply)
          + id                                   = (known after apply)
          + instance_initiated_shutdown_behavior = (known after apply)
          + instance_lifecycle                   = (known after apply)
          + instance_state                       = (known after apply)
          + instance_type                        = (known after apply)
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
          + source_dest_check                    = (known after apply)
          + spot_instance_request_id             = (known after apply)
          + subnet_id                            = (known after apply)
          + tags                                 = (known after apply)
          + tags_all                             = (known after apply)
          + tenancy                              = (known after apply)
          + user_data                            = (known after apply)
          + user_data_base64                     = (known after apply)
          + user_data_replace_on_change          = (known after apply)
          + volume_tags                          = (known after apply)
          + vpc_security_group_ids               = (known after apply)
        } -> (known after apply)

      ~ cpu_options {
          + ami                                  = (known after apply)
          + arn                                  = (known after apply)
          + associate_public_ip_address          = (known after apply)
          + availability_zone                    = (known after apply)
          + cpu_core_count                       = (known after apply)
          + cpu_threads_per_core                 = (known after apply)
          + disable_api_stop                     = (known after apply)
          + disable_api_termination              = (known after apply)
          + ebs_optimized                        = (known after apply)
          + get_password_data                    = (known after apply)
          + hibernation                          = (known after apply)
          + host_id                              = (known after apply)
          + host_resource_group_arn              = (known after apply)
          + iam_instance_profile                 = (known after apply)
          + id                                   = (known after apply)
          + instance_initiated_shutdown_behavior = (known after apply)
          + instance_lifecycle                   = (known after apply)
          + instance_state                       = (known after apply)
          + instance_type                        = (known after apply)
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
          + source_dest_check                    = (known after apply)
          + spot_instance_request_id             = (known after apply)
          + subnet_id                            = (known after apply)
          + tags                                 = (known after apply)
          + tags_all                             = (known after apply)
          + tenancy                              = (known after apply)
          + user_data                            = (known after apply)
          + user_data_base64                     = (known after apply)
          + user_data_replace_on_change          = (known after apply)
          + volume_tags                          = (known after apply)
          + vpc_security_group_ids               = (known after apply)
        } -> (known after apply)

      - credit_specification {
          - cpu_credits = "standard" -> null
        }

      ~ ebs_block_device {
          + ami                                  = (known after apply)
          + arn                                  = (known after apply)
          + associate_public_ip_address          = (known after apply)
          + availability_zone                    = (known after apply)
          + cpu_core_count                       = (known after apply)
          + cpu_threads_per_core                 = (known after apply)
          + disable_api_stop                     = (known after apply)
          + disable_api_termination              = (known after apply)
          + ebs_optimized                        = (known after apply)
          + get_password_data                    = (known after apply)
          + hibernation                          = (known after apply)
          + host_id                              = (known after apply)
          + host_resource_group_arn              = (known after apply)
          + iam_instance_profile                 = (known after apply)
          + id                                   = (known after apply)
          + instance_initiated_shutdown_behavior = (known after apply)
          + instance_lifecycle                   = (known after apply)
          + instance_state                       = (known after apply)
          + instance_type                        = (known after apply)
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
          + source_dest_check                    = (known after apply)
          + spot_instance_request_id             = (known after apply)
          + subnet_id                            = (known after apply)
          + tags                                 = (known after apply)
          + tags_all                             = (known after apply)
          + tenancy                              = (known after apply)
          + user_data                            = (known after apply)
          + user_data_base64                     = (known after apply)
          + user_data_replace_on_change          = (known after apply)
          + volume_tags                          = (known after apply)
          + vpc_security_group_ids               = (known after apply)
        } -> (known after apply)

      ~ enclave_options {
          + ami                                  = (known after apply)
          + arn                                  = (known after apply)
          + associate_public_ip_address          = (known after apply)
          + availability_zone                    = (known after apply)
          + cpu_core_count                       = (known after apply)
          + cpu_threads_per_core                 = (known after apply)
          + disable_api_stop                     = (known after apply)
          + disable_api_termination              = (known after apply)
          + ebs_optimized                        = (known after apply)
          + get_password_data                    = (known after apply)
          + hibernation                          = (known after apply)
          + host_id                              = (known after apply)
          + host_resource_group_arn              = (known after apply)
          + iam_instance_profile                 = (known after apply)
          + id                                   = (known after apply)
          + instance_initiated_shutdown_behavior = (known after apply)
          + instance_lifecycle                   = (known after apply)
          + instance_state                       = (known after apply)
          + instance_type                        = (known after apply)
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
          + source_dest_check                    = (known after apply)
          + spot_instance_request_id             = (known after apply)
          + subnet_id                            = (known after apply)
          + tags                                 = (known after apply)
          + tags_all                             = (known after apply)
          + tenancy                              = (known after apply)
          + user_data                            = (known after apply)
          + user_data_base64                     = (known after apply)
          + user_data_replace_on_change          = (known after apply)
          + volume_tags                          = (known after apply)
          + vpc_security_group_ids               = (known after apply)
        } -> (known after apply)

      ~ ephemeral_block_device {
          + ami                                  = (known after apply)
          + arn                                  = (known after apply)
          + associate_public_ip_address          = (known after apply)
          + availability_zone                    = (known after apply)
          + cpu_core_count                       = (known after apply)
          + cpu_threads_per_core                 = (known after apply)
          + disable_api_stop                     = (known after apply)
          + disable_api_termination              = (known after apply)
          + ebs_optimized                        = (known after apply)
          + get_password_data                    = (known after apply)
          + hibernation                          = (known after apply)
          + host_id                              = (known after apply)
          + host_resource_group_arn              = (known after apply)
          + iam_instance_profile                 = (known after apply)
          + id                                   = (known after apply)
          + instance_initiated_shutdown_behavior = (known after apply)
          + instance_lifecycle                   = (known after apply)
          + instance_state                       = (known after apply)
          + instance_type                        = (known after apply)
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
          + source_dest_check                    = (known after apply)
          + spot_instance_request_id             = (known after apply)
          + subnet_id                            = (known after apply)
          + tags                                 = (known after apply)
          + tags_all                             = (known after apply)
          + tenancy                              = (known after apply)
          + user_data                            = (known after apply)
          + user_data_base64                     = (known after apply)
          + user_data_replace_on_change          = (known after apply)
          + volume_tags                          = (known after apply)
          + vpc_security_group_ids               = (known after apply)
        } -> (known after apply)

      ~ instance_market_options {
          + ami                                  = (known after apply)
          + arn                                  = (known after apply)
          + associate_public_ip_address          = (known after apply)
          + availability_zone                    = (known after apply)
          + cpu_core_count                       = (known after apply)
          + cpu_threads_per_core                 = (known after apply)
          + disable_api_stop                     = (known after apply)
          + disable_api_termination              = (known after apply)
          + ebs_optimized                        = (known after apply)
          + get_password_data                    = (known after apply)
          + hibernation                          = (known after apply)
          + host_id                              = (known after apply)
          + host_resource_group_arn              = (known after apply)
          + iam_instance_profile                 = (known after apply)
          + id                                   = (known after apply)
          + instance_initiated_shutdown_behavior = (known after apply)
          + instance_lifecycle                   = (known after apply)
          + instance_state                       = (known after apply)
          + instance_type                        = (known after apply)
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
          + source_dest_check                    = (known after apply)
          + spot_instance_request_id             = (known after apply)
          + subnet_id                            = (known after apply)
          + tags                                 = (known after apply)
          + tags_all                             = (known after apply)
          + tenancy                              = (known after apply)
          + user_data                            = (known after apply)
          + user_data_base64                     = (known after apply)
          + user_data_replace_on_change          = (known after apply)
          + volume_tags                          = (known after apply)
          + vpc_security_group_ids               = (known after apply)
        } -> (known after apply)

      ~ maintenance_options {
          + ami                                  = (known after apply)
          + arn                                  = (known after apply)
          + associate_public_ip_address          = (known after apply)
          + availability_zone                    = (known after apply)
          + cpu_core_count                       = (known after apply)
          + cpu_threads_per_core                 = (known after apply)
          + disable_api_stop                     = (known after apply)
          + disable_api_termination              = (known after apply)
          + ebs_optimized                        = (known after apply)
          + get_password_data                    = (known after apply)
          + hibernation                          = (known after apply)
          + host_id                              = (known after apply)
          + host_resource_group_arn              = (known after apply)
          + iam_instance_profile                 = (known after apply)
          + id                                   = (known after apply)
          + instance_initiated_shutdown_behavior = (known after apply)
          + instance_lifecycle                   = (known after apply)
          + instance_state                       = (known after apply)
          + instance_type                        = (known after apply)
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
          + source_dest_check                    = (known after apply)
          + spot_instance_request_id             = (known after apply)
          + subnet_id                            = (known after apply)
          + tags                                 = (known after apply)
          + tags_all                             = (known after apply)
          + tenancy                              = (known after apply)
          + user_data                            = (known after apply)
          + user_data_base64                     = (known after apply)
          + user_data_replace_on_change          = (known after apply)
          + volume_tags                          = (known after apply)
          + vpc_security_group_ids               = (known after apply)
        } -> (known after apply)

      ~ metadata_options {
          + ami                                  = (known after apply)
          + arn                                  = (known after apply)
          + associate_public_ip_address          = (known after apply)
          + availability_zone                    = (known after apply)
          + cpu_core_count                       = (known after apply)
          + cpu_threads_per_core                 = (known after apply)
          + disable_api_stop                     = (known after apply)
          + disable_api_termination              = (known after apply)
          + ebs_optimized                        = (known after apply)
          + get_password_data                    = (known after apply)
          + hibernation                          = (known after apply)
          + host_id                              = (known after apply)
          + host_resource_group_arn              = (known after apply)
          + iam_instance_profile                 = (known after apply)
          + id                                   = (known after apply)
          + instance_initiated_shutdown_behavior = (known after apply)
          + instance_lifecycle                   = (known after apply)
          + instance_state                       = (known after apply)
          + instance_type                        = (known after apply)
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
          + source_dest_check                    = (known after apply)
          + spot_instance_request_id             = (known after apply)
          + subnet_id                            = (known after apply)
          + tags                                 = (known after apply)
          + tags_all                             = (known after apply)
          + tenancy                              = (known after apply)
          + user_data                            = (known after apply)
          + user_data_base64                     = (known after apply)
          + user_data_replace_on_change          = (known after apply)
          + volume_tags                          = (known after apply)
          + vpc_security_group_ids               = (known after apply)
        } -> (known after apply)

      ~ network_interface {
          + ami                                  = (known after apply)
          + arn                                  = (known after apply)
          + associate_public_ip_address          = (known after apply)
          + availability_zone                    = (known after apply)
          + cpu_core_count                       = (known after apply)
          + cpu_threads_per_core                 = (known after apply)
          + disable_api_stop                     = (known after apply)
          + disable_api_termination              = (known after apply)
          + ebs_optimized                        = (known after apply)
          + get_password_data                    = (known after apply)
          + hibernation                          = (known after apply)
          + host_id                              = (known after apply)
          + host_resource_group_arn              = (known after apply)
          + iam_instance_profile                 = (known after apply)
          + id                                   = (known after apply)
          + instance_initiated_shutdown_behavior = (known after apply)
          + instance_lifecycle                   = (known after apply)
          + instance_state                       = (known after apply)
          + instance_type                        = (known after apply)
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
          + source_dest_check                    = (known after apply)
          + spot_instance_request_id             = (known after apply)
          + subnet_id                            = (known after apply)
          + tags                                 = (known after apply)
          + tags_all                             = (known after apply)
          + tenancy                              = (known after apply)
          + user_data                            = (known after apply)
          + user_data_base64                     = (known after apply)
          + user_data_replace_on_change          = (known after apply)
          + volume_tags                          = (known after apply)
          + vpc_security_group_ids               = (known after apply)
        } -> (known after apply)

      ~ private_dns_name_options {
          + ami                                  = (known after apply)
          + arn                                  = (known after apply)
          + associate_public_ip_address          = (known after apply)
          + availability_zone                    = (known after apply)
          + cpu_core_count                       = (known after apply)
          + cpu_threads_per_core                 = (known after apply)
          + disable_api_stop                     = (known after apply)
          + disable_api_termination              = (known after apply)
          + ebs_optimized                        = (known after apply)
          + get_password_data                    = (known after apply)
          + hibernation                          = (known after apply)
          + host_id                              = (known after apply)
          + host_resource_group_arn              = (known after apply)
          + iam_instance_profile                 = (known after apply)
          + id                                   = (known after apply)
          + instance_initiated_shutdown_behavior = (known after apply)
          + instance_lifecycle                   = (known after apply)
          + instance_state                       = (known after apply)
          + instance_type                        = (known after apply)
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
          + source_dest_check                    = (known after apply)
          + spot_instance_request_id             = (known after apply)
          + subnet_id                            = (known after apply)
          + tags                                 = (known after apply)
          + tags_all                             = (known after apply)
          + tenancy                              = (known after apply)
          + user_data                            = (known after apply)
          + user_data_base64                     = (known after apply)
          + user_data_replace_on_change          = (known after apply)
          + volume_tags                          = (known after apply)
          + vpc_security_group_ids               = (known after apply)
        } -> (known after apply)

      ~ root_block_device {
          + ami                                  = (known after apply)
          + arn                                  = (known after apply)
          + associate_public_ip_address          = (known after apply)
          + availability_zone                    = (known after apply)
          + cpu_core_count                       = (known after apply)
          + cpu_threads_per_core                 = (known after apply)
          + disable_api_stop                     = (known after apply)
          + disable_api_termination              = (known after apply)
          + ebs_optimized                        = (known after apply)
          + get_password_data                    = (known after apply)
          + hibernation                          = (known after apply)
          + host_id                              = (known after apply)
          + host_resource_group_arn              = (known after apply)
          + iam_instance_profile                 = (known after apply)
          + id                                   = (known after apply)
          + instance_initiated_shutdown_behavior = (known after apply)
          + instance_lifecycle                   = (known after apply)
          + instance_state                       = (known after apply)
          + instance_type                        = (known after apply)
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
          + source_dest_check                    = (known after apply)
          + spot_instance_request_id             = (known after apply)
          + subnet_id                            = (known after apply)
          + tags                                 = (known after apply)
          + tags_all                             = (known after apply)
          + tenancy                              = (known after apply)
          + user_data                            = (known after apply)
          + user_data_base64                     = (known after apply)
          + user_data_replace_on_change          = (known after apply)
          + volume_tags                          = (known after apply)
          + vpc_security_group_ids               = (known after apply)
        } -> (known after apply)
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

Plan: 2 to add, 0 to change, 1 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.example: Destroying... [id=i-0799c62e3c25356a3]
aws_instance.example: Still destroying... [id=i-0799c62e3c25356a3, 10s elapsed]
aws_instance.example: Still destroying... [id=i-0799c62e3c25356a3, 20s elapsed]
aws_instance.example: Still destroying... [id=i-0799c62e3c25356a3, 30s elapsed]
aws_instance.example: Still destroying... [id=i-0799c62e3c25356a3, 40s elapsed]
aws_instance.example: Destruction complete after 40s
aws_security_group.instance: Creating...
aws_security_group.instance: Creation complete after 2s [id=sg-0391be99c372d8d97]
aws_instance.example: Creating...
aws_instance.example: Still creating... [10s elapsed]
aws_instance.example: Still creating... [20s elapsed]
aws_instance.example: Still creating... [30s elapsed]
aws_instance.example: Creation complete after 32s [id=i-05e720cdb10911268]

Apply complete! Resources: 2 added, 0 changed, 1 destroyed.
cloud_user@553b1e446c1c:~/ec2_instances$
```
</details><br />

![]({{ site.baseurl }}/images/2024/07-22-Terraform_101_getting_started/04-EC2-instance-created-with-sg-attached.png)

Port 8080 is exposed!

```bash
$ curl <public_ip_address>:8080
Hello, World
$
```

## Input variables

The syntax of **Input variables** is:

```tf
variable "NAME" {
  [CONFIG ...]
}
```

- `description` shows up when running `terraform plan` and `terraform apply`
- `default` default value for the variable. Can be overwritten with:
  - `-var` arg
  - `-var-file` arg
  - env var `TF_VAR_<variable_name>`
- `type` can be `string`, `number`, `bool`, `list`, `map`, `set`, `object`, `tuple`, and `any`. Default is `any`
- `validation` custom validation rules
- `sensitive` hides the output when running `terraform plan` and `terraform apply`

Example. Checks to verify that the value passed in is a number

```tf
variable "number_example" {
  description = "An example of a number variable in Terraform"
  type        = number
  default     = 42
}
```

Another example. Checks whether the value is a list

```tf
variable "list_example" {
  description = "An example of a list in Terraform"
  type        = list
  default     = ["a", "b", "c"]
}
```

And another example. List input variable that requires all of the items in the list to be numbers

```tf
variable "list_numeric_example" {
  description = "An example of a numeric list in Terraform"
  type        = list(number)
  default     = [1, 2, 3]
}
```

Yet, another example. Map that requires all of the values to be strings:

```tf
variable "map_example" {
  description = "An example of a map in Terraform"
  type        = map(string)

  default = {
    key1 = "value1"
    key2 = "value2"
    key3 = "value3"
  }
}
```

Example of complicated *structural types* using the `object` type constraint

```tf
variable "object_example" {
  description = "An example of a structural type in Terraform"
  type        = object({
    name    = string
    age     = number
    tags    = list(string)
    enabled = bool
  })

  default = {
    name    = "value1"
    age     = 42
    tags    = ["a", "b", "c"]
    enabled = true
  }
}
```

## Include variable "server_port" to main.tf

Adding a variable that stores the port number:

```tf
provider "aws" {
  region = "us-east-1"
}

variable "server_port" {
  description = "The port the server will use for HTTP requests"
  type        = number
}

resource "aws_security_group" "instance" {
  name = "terraform-example-instance"

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.instance.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p 8080 &
              EOF

  user_data_replace_on_change = true

  tags = {
    Name = "ubuntu1-named-with-terraform"
  }
}
```

## Use server_port variable in main.tf

Use the variable with the syntax `var.<VARIABLE_NAME>`. This is called *variable reference*

Instead of

```tf
  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
```

we should have

```tf
  ingress {
    from_port   = var.server_port
    to_port     = var.server_port
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
```

Also, instead of

```tf
  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p 8080 &
              EOF
```

we should have

```tf
  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p ${var.server_port} &
              EOF
```

Here the `main.tf` file including this change:

```tf
provider "aws" {
  region = "us-east-1"
}

variable "server_port" {
  description = "The port the server will use for HTTP requests"
  type        = number
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

resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.instance.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p ${var.server_port} &
              EOF

  user_data_replace_on_change = true

  tags = {
    Name = "ubuntu1-named-with-terraform"
  }
}
```

## Examples of how to set a value to the Input variables

### Interactively prompt

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ terraform apply
var.server_port
  The port the server will use for HTTP requests

  Enter a value:
```

### Passing the value as argument

```bash
terraform plan -var "server_port=8080"
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ terraform plan -var "server_port=8080"
aws_security_group.instance: Refreshing state... [id=sg-0391be99c372d8d97]
aws_instance.example: Refreshing state... [id=i-05e720cdb10911268]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # aws_security_group.instance will be updated in-place
  ~ resource "aws_security_group" "instance" {
        id                     = "sg-0391be99c372d8d97"
      ~ ingress                = [
          - {
              - cidr_blocks      = [
                  - "0.0.0.0/0",
                ]
              - from_port        = 22
              - ipv6_cidr_blocks = []
              - prefix_list_ids  = []
              - protocol         = "tcp"
              - security_groups  = []
              - self             = false
              - to_port          = 22
                # (1 unchanged attribute hidden)
            },
          - {
              - cidr_blocks      = [
                  - "0.0.0.0/0",
                ]
              - from_port        = 8080
              - ipv6_cidr_blocks = []
              - prefix_list_ids  = []
              - protocol         = "tcp"
              - security_groups  = []
              - self             = false
              - to_port          = 8080
                # (1 unchanged attribute hidden)
            },
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
            },
        ]
        name                   = "terraform-example-instance"
        tags                   = {}
        # (8 unchanged attributes hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
cloud_user@553b1e446c1c:~/ec2_instances$
```
</details><br />

### Using env vars

```bash
$ export TF_VAR_server_port=8080
$ terraform plan
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ export TF_VAR_server_port=8080
cloud_user@553b1e446c1c:~/ec2_instances$ terraform plan
aws_security_group.instance: Refreshing state... [id=sg-0391be99c372d8d97]
aws_instance.example: Refreshing state... [id=i-05e720cdb10911268]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # aws_security_group.instance will be updated in-place
  ~ resource "aws_security_group" "instance" {
        id                     = "sg-0391be99c372d8d97"
      ~ ingress                = [
          - {
              - cidr_blocks      = [
                  - "0.0.0.0/0",
                ]
              - from_port        = 22
              - ipv6_cidr_blocks = []
              - prefix_list_ids  = []
              - protocol         = "tcp"
              - security_groups  = []
              - self             = false
              - to_port          = 22
                # (1 unchanged attribute hidden)
            },
          - {
              - cidr_blocks      = [
                  - "0.0.0.0/0",
                ]
              - from_port        = 8080
              - ipv6_cidr_blocks = []
              - prefix_list_ids  = []
              - protocol         = "tcp"
              - security_groups  = []
              - self             = false
              - to_port          = 8080
                # (1 unchanged attribute hidden)
            },
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
            },
        ]
        name                   = "terraform-example-instance"
        tags                   = {}
        # (8 unchanged attributes hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
cloud_user@553b1e446c1c:~/ec2_instances$
```
</details><br />

### Setting a default value

```bash
variable "server_port" {
  description = "The port the server will use for HTTP requests"
  type        = number
  default     = 8080
}
```

## Output variables

The syntax of output variables is:

```bash
output "<NAME>" {
  value = <VALUE>
  [CONFIG ...]
}
```

- `description` shows up when running `terraform plan` and `terraform apply`
- `sensitive` hides the output when running `terraform plan` and `terraform apply`
- `depends_on` explicitly tell Terraform there is a dependency before this output variable can be used

Example

```tf
output "public_ip" {
  value       = aws_instance.example.public_ip
  description = "The public IP address of the web server"
}
```

## Use output variable in main.tf

```tf
provider "aws" {
  region = "us-east-1"
}

variable "server_port" {
  description = "The port the server will use for HTTP requests"
  type        = number
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

resource "aws_instance" "example" {
  ami           = "ami-04a81a99f5ec58529"
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.instance.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p ${var.server_port} &
              EOF

  user_data_replace_on_change = true

  tags = {
    Name = "ubuntu1-named-with-terraform"
  }
}

output "public_ip" {
  value       = aws_instance.example.public_ip
  description = "The public IP address of the web server"
}
```

```bash
terraform apply
```

Snippet of the output, we see the **output variable**.

```
...
aws_security_group.instance: Modifying... [id=sg-0391be99c372d8d97]
aws_security_group.instance: Modifications complete after 1s [id=sg-0391be99c372d8d97]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.

Outputs:

public_ip = "54.XXX.X.XXX"
cloud_user@553b1e446c1c:~/ec2_instances$
```

## terraform output

We can see the output again after `terraform apply` with `terraform output`

```bash
terraform output
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ terraform output
public_ip = "54.XXX.X.XXX"
cloud_user@553b1e446c1c:~/ec2_instances$
```
</details><br />

To get the specific output `terraform output <OUTPUT_NAME>`

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ terraform output public_ip
"54.XXX.X.XXX"
cloud_user@553b1e446c1c:~/ec2_instances$
```

## terraform destroy

Removes all resources created with terraform

```bash
terraform destroy
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/ec2_instances$ terraform destroy
aws_security_group.instance: Refreshing state... [id=sg-0391be99c372d8d97]
aws_instance.example: Refreshing state... [id=i-05e720cdb10911268]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_instance.example will be destroyed
  - resource "aws_instance" "example" {
      - ami                                  = "ami-04a81a99f5ec58529" -> null
      - arn                                  = "arn:aws:ec2:us-east-1:381492126797:instance/i-05e720cdb10911268" -> null
      - associate_public_ip_address          = true -> null
      - availability_zone                    = "us-east-1e" -> null
      - cpu_core_count                       = 1 -> null
      - cpu_threads_per_core                 = 1 -> null
      - disable_api_stop                     = false -> null
      - disable_api_termination              = false -> null
      - ebs_optimized                        = false -> null
      - get_password_data                    = false -> null
      - hibernation                          = false -> null
      - id                                   = "i-05e720cdb10911268" -> null
      - instance_initiated_shutdown_behavior = "stop" -> null
      - instance_state                       = "running" -> null
      - instance_type                        = "t2.micro" -> null
      - ipv6_address_count                   = 0 -> null
      - ipv6_addresses                       = [] -> null
      - monitoring                           = false -> null
      - placement_partition_number           = 0 -> null
      - primary_network_interface_id         = "eni-0378449983db228fd" -> null
      - private_dns                          = "ip-172-31-59-109.ec2.internal" -> null
      - private_ip                           = "172.31.59.109" -> null
      - public_dns                           = "ec2-54-237-8-159.compute-1.amazonaws.com" -> null
      - public_ip                            = "54.237.8.159" -> null
      - secondary_private_ips                = [] -> null
      - security_groups                      = [
          - "terraform-example-instance",
        ] -> null
      - source_dest_check                    = true -> null
      - subnet_id                            = "subnet-023fb475470145a1e" -> null
      - tags                                 = {
          - "Name" = "ubuntu1-named-with-terraform"
        } -> null
      - tags_all                             = {
          - "Name" = "ubuntu1-named-with-terraform"
        } -> null
      - tenancy                              = "default" -> null
      - user_data                            = "c765373c563b260626d113c4a56a46e8a8c5ca33" -> null
      - user_data_replace_on_change          = true -> null
      - vpc_security_group_ids               = [
          - "sg-0391be99c372d8d97",
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
          - volume_id             = "vol-01fcc03dd34e5f76d" -> null
          - volume_size           = 8 -> null
          - volume_type           = "gp3" -> null
            # (1 unchanged attribute hidden)
        }
    }

  # aws_security_group.instance will be destroyed
  - resource "aws_security_group" "instance" {
      - arn                    = "arn:aws:ec2:us-east-1:381492126797:security-group/sg-0391be99c372d8d97" -> null
      - description            = "Managed by Terraform" -> null
      - egress                 = [] -> null
      - id                     = "sg-0391be99c372d8d97" -> null
      - ingress                = [
          - {
              - cidr_blocks      = [
                  - "0.0.0.0/0",
                ]
              - from_port        = 8080
              - ipv6_cidr_blocks = []
              - prefix_list_ids  = []
              - protocol         = "tcp"
              - security_groups  = []
              - self             = false
              - to_port          = 8080
                # (1 unchanged attribute hidden)
            },
        ] -> null
      - name                   = "terraform-example-instance" -> null
      - owner_id               = "381492126797" -> null
      - revoke_rules_on_delete = false -> null
      - tags                   = {} -> null
      - tags_all               = {} -> null
      - vpc_id                 = "vpc-00cbdfa4bfae3a926" -> null
        # (1 unchanged attribute hidden)
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Changes to Outputs:
  - public_ip = "54.XXX.X.XXX" -> null

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_instance.example: Destroying... [id=i-05e720cdb10911268]
aws_instance.example: Still destroying... [id=i-05e720cdb10911268, 10s elapsed]
aws_instance.example: Still destroying... [id=i-05e720cdb10911268, 20s elapsed]
aws_instance.example: Still destroying... [id=i-05e720cdb10911268, 30s elapsed]
aws_instance.example: Still destroying... [id=i-05e720cdb10911268, 40s elapsed]
aws_instance.example: Still destroying... [id=i-05e720cdb10911268, 50s elapsed]
aws_instance.example: Still destroying... [id=i-05e720cdb10911268, 1m0s elapsed]
aws_instance.example: Destruction complete after 1m0s
aws_security_group.instance: Destroying... [id=sg-0391be99c372d8d97]
aws_security_group.instance: Destruction complete after 1s

Destroy complete! Resources: 2 destroyed.
cloud_user@553b1e446c1c:~/ec2_instances$
```
</details><br />

The EC2 instance is terminated

![]({{ site.baseurl }}/images/2024/07-22-Terraform_101_getting_started/05-EC2-instance-terminated.png)

## References

- [Terraform: Up & Running By Yevgeniy Brikman](https://www.terraformupandrunning.com/)
- [Resource: aws_instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)
- [Markdown Code blocks Terraform Syntax highlighting](https://marketplace.visualstudio.com/items?itemName=64kramsystem.markdown-code-blocks-terraform-syntax-highlighting)
- [How to check whether my user data passing to EC2 instance is working](https://stackoverflow.com/questions/15904095/how-to-check-whether-my-user-data-passing-to-ec2-instance-is-working)
- [Unsure why user_data isn't seemingly working](https://www.reddit.com/r/Terraform/comments/xe6nnp/unsure_why_user_data_isnt_seemingly_working/)
