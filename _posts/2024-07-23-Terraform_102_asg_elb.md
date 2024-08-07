---
title: Terraform 102 deploying ASG and ELB
date: 2024-07-23 19:00:00 -0700
categories: [TERRAFORM]
tags: [terraform, terraform aws asg elb]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/terraform_logo.png)

## Introduction

In this guide, we are going to create **EC2 instances** using **Auto Scaling groups** (ASG). The **EC2 instances** will run a very simple web servers and these will be behind an **Elastic Load Balancer** (ELB). This infrastructure will be provisioned with **Terraform**.

Here a summary of all the AWS resources to deploy along with a link to their corresponding Terraform documentation.

|      AWS element      |                                                         Terraform Documentation                                                        |
|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| Launch Configuration  | [Resource: aws_launch_configuration](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/launch_configuration) |
| Security Group        | [Resource: aws_security_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)             |
| Data Source AWS VPC   | [Data Source: aws_vpc](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/vpc)                             |
| Auto Scaling groups   | [Resource: aws_autoscaling_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/autoscaling_group)       |
| Elastic Load Balancer | [Resource: aws_lb](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb)                                     |
| Listener              | [Resource: aws_lb_listener](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_listener)                   |
| Listener Rule         | [Resource: aws_lb_listener_rule](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_listener_rule)         |
| Target Group          | [Resource: aws_lb_target_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_target_group)           |

## Launch Configuration

The **Launch Configuration** will be referenced by the **Auto Scaling group**. It is the template to know the characteristics about the EC2 instances to launch.

The `aws_launch_configuration` resource is very similar to the `aws_instance` resource with some subtle differences:

|                `aws_launch_configuration`                 |                        `aws_instance`                        |
|-----------------------------------------------------------|--------------------------------------------------------------|
| Does not support tags                                     | Supports tags                                                |
| Does not need the `user_data_replace_on_change` argument  | Supports the optional `user_data_replace_on_change` argument |
| `image_id` argument to specify the AMI ID                 | `ami` argument to specify the AMI ID                         |
| `security_groups` argument to specify the SG              | `vpc_security_group_ids` argument to specify the SG          |

Here the Terraform code to configure the **Launch Configuration**. Since the EC2 instances will have a Security Group attached, we also need to explicitly attach a Security Group opened with the **tcp port 80** in the Inbound direction.

```tf
resource "aws_launch_configuration" "example" {
  image_id        = "ami-04a81a99f5ec58529"
  instance_type   = "t2.micro"
  security_groups = [aws_security_group.instance.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p ${var.server_port} &
              EOF

  # Required when using a launch configuration with an auto scaling group.
  lifecycle {
    create_before_destroy = true
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

variable "server_port" {
  description = "The port the server will use for HTTP requests"
  type        = number
  default     = 80
}
```

According to the documentation: [Resource: aws_launch_configuration](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/launch_configuration)

*Launch Configurations cannot be updated after creation with the Amazon Web Service API. In order to update a Launch Configuration, Terraform will destroy the existing resource and create a replacement. In order to effectively use a Launch Configuration resource with an AutoScaling Group resource, it's recommended to specify create_before_destroy in a lifecycle block.*

**NOTE:** The AMI is AWS region dependent. So, it is different on every region even for the same OS. This is the one for **Ubuntu Server 24.04 LTS** in **us-east-1**.

![]({{ site.baseurl }}/images/2024/07-22-Terraform_101_getting_started/01-Ubuntu-Server-2004-AMI-in-us-east-1.png)

**NOTE:** We are using **Launch Configurations** in this guide. However, these are no longer supported by AWS and we should be using **Launch Templates** instead.

![]({{ site.baseurl }}/images/2024/07-23-Terraform_102_asg_elb/01-Use-Launch-Templates-instead.png)

## Data Source

To extract data to be used in the Terraform HCL file.

The **Data Source** syntax is:

```bash
data "<PROVIDER>_<TYPE>" "<NAME>" {
  [CONFIG ...]
}
```

- `PROVIDER` is the name of the provider (in this case `aws`)
- `TYPE` type of data source (example `vpc`)
- `NAME` identifier in the Terraform code
- `CONFIG` arguments

The arguments are typically filters. Refer to the documentation [Data Source: aws_vpc](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/vpc)

To obtain the data:

```bash
data.<PROVIDER>_<TYPE>.<NAME>.<ATTRIBUTE>
```

Example, to obtaion the **ID** of the **Default VPC**:

```bash
data "aws_vpc" "default" {
  default = true
}
```

and to obtain the data:

```bash
data.aws_vpc.default.id
```

This can be used to obtain the IDs of the **Subnets** for a specific **VPC**:

```bash
data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}
```

## Auto Scaling group

We now configure the **Auto Scaling group**. The **Launch Configuration** is referenced in the argument `launch_configuration`.

```tf
resource "aws_autoscaling_group" "example" {
  launch_configuration = aws_launch_configuration.example.name
  vpc_zone_identifier  = data.aws_subnets.default.ids

  min_size = 2
  max_size = 10

  tag {
    key                 = "Name"
    value               = "terraform-asg-example"
    propagate_at_launch = true
  }
}

data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}
```

## Launch configuration + ASG in the terraform file

So far, if we put everything together in a single `main.tf` file.

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_launch_configuration" "example" {
  image_id        = "ami-04a81a99f5ec58529"
  instance_type   = "t2.micro"
  security_groups = [aws_security_group.instance.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p ${var.server_port} &
              EOF

  # Required when using a launch configuration with an auto scaling group.
  lifecycle {
    create_before_destroy = true
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

variable "server_port" {
  description = "The port the server will use for HTTP requests"
  type        = number
  default     = 8080
}

resource "aws_autoscaling_group" "example" {
  launch_configuration = aws_launch_configuration.example.name
  vpc_zone_identifier  = data.aws_subnets.default.ids

  min_size = 2
  max_size = 10

  tag {
    key                 = "Name"
    value               = "terraform-asg-example"
    propagate_at_launch = true
  }
}

data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}
```

<details markdown=1>
<summary markdown="span">terraform init</summary>

```bash
cloud_user@553b1e446c1c:~/asg_elb_terraform$ terraform init
Initializing the backend...
Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file
- Using previously-installed hashicorp/aws v5.59.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
cloud_user@553b1e446c1c:~/asg_elb_terraform$
```
</details><br />

<details markdown=1>
<summary markdown="span">terraform plan</summary>

```bash
cloud_user@553b1e446c1c:~/asg_elb_terraform$ terraform plan
data.aws_vpc.default: Reading...
data.aws_vpc.default: Read complete after 1s [id=vpc-01dd39250fbbe13c6]
data.aws_subnets.default: Reading...
data.aws_subnets.default: Read complete after 0s [id=us-east-1]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_autoscaling_group.example will be created
  + resource "aws_autoscaling_group" "example" {
      + arn                              = (known after apply)
      + availability_zones               = (known after apply)
      + default_cooldown                 = (known after apply)
      + desired_capacity                 = (known after apply)
      + force_delete                     = false
      + force_delete_warm_pool           = false
      + health_check_grace_period        = 300
      + health_check_type                = (known after apply)
      + id                               = (known after apply)
      + ignore_failed_scaling_activities = false
      + launch_configuration             = (known after apply)
      + load_balancers                   = (known after apply)
      + max_size                         = 10
      + metrics_granularity              = "1Minute"
      + min_size                         = 2
      + name                             = (known after apply)
      + name_prefix                      = (known after apply)
      + predicted_capacity               = (known after apply)
      + protect_from_scale_in            = false
      + service_linked_role_arn          = (known after apply)
      + target_group_arns                = (known after apply)
      + vpc_zone_identifier              = [
          + "subnet-013196de977c33b7d",
          + "subnet-075866766bdfb54d4",
          + "subnet-0c316180bdeb61390",
          + "subnet-0cee7f5995ad6a557",
          + "subnet-0d9ac99195f2661b7",
          + "subnet-0fd7ed03d1c9cc941",
        ]
      + wait_for_capacity_timeout        = "10m"
      + warm_pool_size                   = (known after apply)

      + launch_template (known after apply)

      + mixed_instances_policy (known after apply)

      + tag {
          + key                 = "Name"
          + propagate_at_launch = true
          + value               = "terraform-asg-example"
        }

      + traffic_source (known after apply)
    }

  # aws_launch_configuration.example will be created
  + resource "aws_launch_configuration" "example" {
      + arn                         = (known after apply)
      + associate_public_ip_address = (known after apply)
      + ebs_optimized               = (known after apply)
      + enable_monitoring           = true
      + id                          = (known after apply)
      + image_id                    = "ami-04a81a99f5ec58529"
      + instance_type               = "t2.micro"
      + key_name                    = (known after apply)
      + name                        = (known after apply)
      + name_prefix                 = (known after apply)
      + security_groups             = (known after apply)
      + user_data                   = "f656cd9de8860addc5a82630adc18c1b3a4883e5"

      + ebs_block_device (known after apply)

      + metadata_options (known after apply)

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
              + from_port        = 80
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 80
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

Plan: 3 to add, 0 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
cloud_user@553b1e446c1c:~/asg_elb_terraform$
```
</details><br />

<details markdown=1>
<summary markdown="span">terraform apply</summary>

```bash
cloud_user@553b1e446c1c:~/asg_elb_terraform$ terraform apply
data.aws_vpc.default: Reading...
data.aws_vpc.default: Read complete after 0s [id=vpc-01dd39250fbbe13c6]
data.aws_subnets.default: Reading...
data.aws_subnets.default: Read complete after 0s [id=us-east-1]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_autoscaling_group.example will be created
  + resource "aws_autoscaling_group" "example" {
      + arn                              = (known after apply)
      + availability_zones               = (known after apply)
      + default_cooldown                 = (known after apply)
      + desired_capacity                 = (known after apply)
      + force_delete                     = false
      + force_delete_warm_pool           = false
      + health_check_grace_period        = 300
      + health_check_type                = (known after apply)
      + id                               = (known after apply)
      + ignore_failed_scaling_activities = false
      + launch_configuration             = (known after apply)
      + load_balancers                   = (known after apply)
      + max_size                         = 10
      + metrics_granularity              = "1Minute"
      + min_size                         = 2
      + name                             = (known after apply)
      + name_prefix                      = (known after apply)
      + predicted_capacity               = (known after apply)
      + protect_from_scale_in            = false
      + service_linked_role_arn          = (known after apply)
      + target_group_arns                = (known after apply)
      + vpc_zone_identifier              = [
          + "subnet-013196de977c33b7d",
          + "subnet-075866766bdfb54d4",
          + "subnet-0c316180bdeb61390",
          + "subnet-0cee7f5995ad6a557",
          + "subnet-0d9ac99195f2661b7",
          + "subnet-0fd7ed03d1c9cc941",
        ]
      + wait_for_capacity_timeout        = "10m"
      + warm_pool_size                   = (known after apply)

      + launch_template (known after apply)

      + mixed_instances_policy (known after apply)

      + tag {
          + key                 = "Name"
          + propagate_at_launch = true
          + value               = "terraform-asg-example"
        }

      + traffic_source (known after apply)
    }

  # aws_launch_configuration.example will be created
  + resource "aws_launch_configuration" "example" {
      + arn                         = (known after apply)
      + associate_public_ip_address = (known after apply)
      + ebs_optimized               = (known after apply)
      + enable_monitoring           = true
      + id                          = (known after apply)
      + image_id                    = "ami-04a81a99f5ec58529"
      + instance_type               = "t2.micro"
      + key_name                    = (known after apply)
      + name                        = (known after apply)
      + name_prefix                 = (known after apply)
      + security_groups             = (known after apply)
      + user_data                   = "f656cd9de8860addc5a82630adc18c1b3a4883e5"

      + ebs_block_device (known after apply)

      + metadata_options (known after apply)

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
              + from_port        = 80
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 80
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

Plan: 3 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_security_group.instance: Creating...
aws_security_group.instance: Creation complete after 2s [id=sg-06154f2fcdb47daf6]
aws_launch_configuration.example: Creating...
aws_launch_configuration.example: Creation complete after 1s [id=terraform-20240724024038018100000001]
aws_autoscaling_group.example: Creating...
aws_autoscaling_group.example: Still creating... [10s elapsed]
aws_autoscaling_group.example: Still creating... [20s elapsed]
aws_autoscaling_group.example: Still creating... [30s elapsed]
aws_autoscaling_group.example: Still creating... [40s elapsed]
aws_autoscaling_group.example: Still creating... [50s elapsed]
aws_autoscaling_group.example: Still creating... [1m0s elapsed]
aws_autoscaling_group.example: Creation complete after 1m6s [id=terraform-20240724024038564900000002]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
cloud_user@553b1e446c1c:~/asg_elb_terraform$
```
</details><br />

<details markdown=1>
<summary markdown="span">terraform destroy</summary>

```bash
cloud_user@553b1e446c1c:~/asg_elb_terraform$ terraform destroy
data.aws_vpc.default: Reading...
aws_security_group.instance: Refreshing state... [id=sg-06154f2fcdb47daf6]
aws_launch_configuration.example: Refreshing state... [id=terraform-20240724024038018100000001]
data.aws_vpc.default: Read complete after 1s [id=vpc-01dd39250fbbe13c6]
data.aws_subnets.default: Reading...
data.aws_subnets.default: Read complete after 0s [id=us-east-1]
aws_autoscaling_group.example: Refreshing state... [id=terraform-20240724024038564900000002]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_autoscaling_group.example will be destroyed
  - resource "aws_autoscaling_group" "example" {
      - arn                              = "arn:aws:autoscaling:us-east-1:XXXXXXXXXXXX:autoScalingGroup:d2761b27-8adf-4db9-8f47-89474ae0bbef:autoScalingGroupName/terraform-20240724024038564900000002" -> null
      - availability_zones               = [
          - "us-east-1a",
          - "us-east-1b",
          - "us-east-1c",
          - "us-east-1d",
          - "us-east-1e",
          - "us-east-1f",
        ] -> null
      - capacity_rebalance               = false -> null
      - default_cooldown                 = 300 -> null
      - default_instance_warmup          = 0 -> null
      - desired_capacity                 = 2 -> null
      - enabled_metrics                  = [] -> null
      - force_delete                     = false -> null
      - force_delete_warm_pool           = false -> null
      - health_check_grace_period        = 300 -> null
      - health_check_type                = "EC2" -> null
      - id                               = "terraform-20240724024038564900000002" -> null
      - ignore_failed_scaling_activities = false -> null
      - launch_configuration             = "terraform-20240724024038018100000001" -> null
      - load_balancers                   = [] -> null
      - max_instance_lifetime            = 0 -> null
      - max_size                         = 10 -> null
      - metrics_granularity              = "1Minute" -> null
      - min_size                         = 2 -> null
      - name                             = "terraform-20240724024038564900000002" -> null
      - name_prefix                      = "terraform-" -> null
      - predicted_capacity               = 0 -> null
      - protect_from_scale_in            = false -> null
      - service_linked_role_arn          = "arn:aws:iam::XXXXXXXXXXXX:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling" -> null
      - suspended_processes              = [] -> null
      - target_group_arns                = [] -> null
      - termination_policies             = [] -> null
      - vpc_zone_identifier              = [
          - "subnet-013196de977c33b7d",
          - "subnet-075866766bdfb54d4",
          - "subnet-0c316180bdeb61390",
          - "subnet-0cee7f5995ad6a557",
          - "subnet-0d9ac99195f2661b7",
          - "subnet-0fd7ed03d1c9cc941",
        ] -> null
      - wait_for_capacity_timeout        = "10m" -> null
      - warm_pool_size                   = 0 -> null
        # (3 unchanged attributes hidden)

      - tag {
          - key                 = "Name" -> null
          - propagate_at_launch = true -> null
          - value               = "terraform-asg-example" -> null
        }
    }

  # aws_launch_configuration.example will be destroyed
  - resource "aws_launch_configuration" "example" {
      - arn                         = "arn:aws:autoscaling:us-east-1:XXXXXXXXXXXX:launchConfiguration:77ca68c0-b75e-4b9f-9b1f-6c8d9f7a7804:launchConfigurationName/terraform-20240724024038018100000001" -> null
      - associate_public_ip_address = false -> null
      - ebs_optimized               = false -> null
      - enable_monitoring           = true -> null
      - id                          = "terraform-20240724024038018100000001" -> null
      - image_id                    = "ami-04a81a99f5ec58529" -> null
      - instance_type               = "t2.micro" -> null
      - name                        = "terraform-20240724024038018100000001" -> null
      - name_prefix                 = "terraform-" -> null
      - security_groups             = [
          - "sg-06154f2fcdb47daf6",
        ] -> null
      - user_data                   = "f656cd9de8860addc5a82630adc18c1b3a4883e5" -> null
        # (4 unchanged attributes hidden)
    }

  # aws_security_group.instance will be destroyed
  - resource "aws_security_group" "instance" {
      - arn                    = "arn:aws:ec2:us-east-1:XXXXXXXXXXXX:security-group/sg-06154f2fcdb47daf6" -> null
      - description            = "Managed by Terraform" -> null
      - egress                 = [] -> null
      - id                     = "sg-06154f2fcdb47daf6" -> null
      - ingress                = [
          - {
              - cidr_blocks      = [
                  - "0.0.0.0/0",
                ]
              - from_port        = 80
              - ipv6_cidr_blocks = []
              - prefix_list_ids  = []
              - protocol         = "tcp"
              - security_groups  = []
              - self             = false
              - to_port          = 80
                # (1 unchanged attribute hidden)
            },
        ] -> null
      - name                   = "terraform-example-instance" -> null
      - owner_id               = "XXXXXXXXXXXX" -> null
      - revoke_rules_on_delete = false -> null
      - tags                   = {} -> null
      - tags_all               = {} -> null
      - vpc_id                 = "vpc-01dd39250fbbe13c6" -> null
        # (1 unchanged attribute hidden)
    }

Plan: 0 to add, 0 to change, 3 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_autoscaling_group.example: Destroying... [id=terraform-20240724024038564900000002]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724024038564900000002, 10s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724024038564900000002, 20s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724024038564900000002, 30s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724024038564900000002, 40s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724024038564900000002, 50s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724024038564900000002, 1m0s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724024038564900000002, 1m11s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724024038564900000002, 1m21s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724024038564900000002, 1m31s elapsed]
aws_autoscaling_group.example: Destruction complete after 1m33s
aws_launch_configuration.example: Destroying... [id=terraform-20240724024038018100000001]
aws_launch_configuration.example: Destruction complete after 0s
aws_security_group.instance: Destroying... [id=sg-06154f2fcdb47daf6]
aws_security_group.instance: Destruction complete after 1s

Destroy complete! Resources: 3 destroyed.
cloud_user@553b1e446c1c:~/asg_elb_terraform$
```
</details><br />

## Elastic Load Balancer

We will be using an **Application Load Balancer** and we will need the following configurations:

- **Listener:** Makes the ELB listen on a specific port
- **Listener rule:** Forwarding based on specific criterias in the URL
- **Target groups:** EC2 instances that will be "behind" the ELB

This resource creates the ELB.

We will need a new **Security Group** attached to the ELB to allow Inbound traffic to it and Outbound traffic from it to monitor the health of the EC2 instances.

```tf
resource "aws_lb" "example" {
  name               = "terraform-asg-example"
  load_balancer_type = "application"
  subnets            = data.aws_subnets.default.ids
  security_groups    = [aws_security_group.alb.id]
}

resource "aws_security_group" "alb" {
  name = "terraform-example-alb"

  # Allow inbound HTTP requests
  ingress {
    from_port   = var.elb_port
    to_port     = var.elb_port
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow all outbound requests
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

variable "elb_port" {
  description = "The port the server will use for HTTP requests"
  type        = number
  default     = 80
}
```

### Listener

```tf
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.example.arn
  port              = var.elb_port
  protocol          = "HTTP"

  # By default, return a simple 404 page
  default_action {
    type = "fixed-response"

    fixed_response {
      content_type = "text/plain"
      message_body = "404: page not found"
      status_code  = 404
    }
  }
}
```

### Listener rule

```tf
resource "aws_lb_listener_rule" "asg" {
  listener_arn = aws_lb_listener.http.arn
  priority     = 100

  condition {
    path_pattern {
      values = ["*"]
    }
  }

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.asg.arn
  }
}
```

### Target Group

```tf
resource "aws_lb_target_group" "asg" {
  name     = "terraform-asg-example"
  port     = var.server_port
  protocol = "HTTP"
  vpc_id   = data.aws_vpc.default.id

  health_check {
    path                = "/"
    protocol            = "HTTP"
    matcher             = "200"
    interval            = 15
    timeout             = 3
    healthy_threshold   = 2
    unhealthy_threshold = 2
  }
}
```

## Uptating Auto Scaling group

For the target group know which EC2 Instances to send requests to, we add these arguments to the `aws_autoscaling_group`:

- `target_group_arns` -  Set of `aws_alb_target_group` ARNs, for use with Application or Network Load Balancing. 
- `health_check_type` - "EC2" or "ELB". Controls how health checking is done.

```tf
resource "aws_autoscaling_group" "example" {
  launch_configuration = aws_launch_configuration.example.name
  vpc_zone_identifier  = data.aws_subnets.default.ids

  target_group_arns = [aws_lb_target_group.asg.arn]
  health_check_type = "ELB"

  min_size = 2
  max_size = 10

  tag {
    key                 = "Name"
    value               = "terraform-asg-example"
    propagate_at_launch = true
  }
}
```

## Output variable to obtain the URL to the ELB

```tf
output "alb_dns_name" {
  value       = aws_lb.example.dns_name
  description = "The domain name of the load balancer"
}
```

## Complete Terraform file

`main.tf`

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_launch_configuration" "example" {
  image_id        = "ami-04a81a99f5ec58529"
  instance_type   = "t2.micro"
  security_groups = [aws_security_group.instance.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p ${var.server_port} &
              EOF

  # Required when using a launch configuration with an auto scaling group.
  lifecycle {
    create_before_destroy = true
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

variable "server_port" {
  description = "The port the server will use for HTTP requests"
  type        = number
  default     = 8080
}

resource "aws_autoscaling_group" "example" {
  launch_configuration = aws_launch_configuration.example.name
  vpc_zone_identifier  = data.aws_subnets.default.ids

  target_group_arns = [aws_lb_target_group.asg.arn]
  health_check_type = "ELB"

  min_size = 2
  max_size = 10

  tag {
    key                 = "Name"
    value               = "terraform-asg-example"
    propagate_at_launch = true
  }
}

data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

resource "aws_lb" "example" {
  name               = "terraform-asg-example"
  load_balancer_type = "application"
  subnets            = data.aws_subnets.default.ids
  security_groups    = [aws_security_group.alb.id]
}

resource "aws_security_group" "alb" {
  name = "terraform-example-alb"

  # Allow inbound HTTP requests
  ingress {
    from_port   = var.elb_port
    to_port     = var.elb_port
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow all outbound requests
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

variable "elb_port" {
  description = "The port the server will use for HTTP requests"
  type        = number
  default     = 80
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.example.arn
  port              = var.elb_port
  protocol          = "HTTP"

  # By default, return a simple 404 page
  default_action {
    type = "fixed-response"

    fixed_response {
      content_type = "text/plain"
      message_body = "404: page not found"
      status_code  = 404
    }
  }
}

resource "aws_lb_listener_rule" "asg" {
  listener_arn = aws_lb_listener.http.arn
  priority     = 100

  condition {
    path_pattern {
      values = ["*"]
    }
  }

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.asg.arn
  }
}

resource "aws_lb_target_group" "asg" {
  name     = "terraform-asg-example"
  port     = var.server_port
  protocol = "HTTP"
  vpc_id   = data.aws_vpc.default.id

  health_check {
    path                = "/"
    protocol            = "HTTP"
    matcher             = "200"
    interval            = 15
    timeout             = 3
    healthy_threshold   = 2
    unhealthy_threshold = 2
  }
}

output "alb_dns_name" {
  value       = aws_lb.example.dns_name
  description = "The domain name of the load balancer"
}
```

<details markdown=1>
<summary markdown="span">terraform plan</summary>

```bash
cloud_user@553b1e446c1c:~/asg_elb_terraform$ terraform plan
data.aws_vpc.default: Reading...
data.aws_vpc.default: Read complete after 0s [id=vpc-01dd39250fbbe13c6]
data.aws_subnets.default: Reading...
data.aws_subnets.default: Read complete after 1s [id=us-east-1]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_autoscaling_group.example will be created
  + resource "aws_autoscaling_group" "example" {
      + arn                              = (known after apply)
      + availability_zones               = (known after apply)
      + default_cooldown                 = (known after apply)
      + desired_capacity                 = (known after apply)
      + force_delete                     = false
      + force_delete_warm_pool           = false
      + health_check_grace_period        = 300
      + health_check_type                = "ELB"
      + id                               = (known after apply)
      + ignore_failed_scaling_activities = false
      + launch_configuration             = (known after apply)
      + load_balancers                   = (known after apply)
      + max_size                         = 10
      + metrics_granularity              = "1Minute"
      + min_size                         = 2
      + name                             = (known after apply)
      + name_prefix                      = (known after apply)
      + predicted_capacity               = (known after apply)
      + protect_from_scale_in            = false
      + service_linked_role_arn          = (known after apply)
      + target_group_arns                = (known after apply)
      + vpc_zone_identifier              = [
          + "subnet-013196de977c33b7d",
          + "subnet-075866766bdfb54d4",
          + "subnet-0c316180bdeb61390",
          + "subnet-0cee7f5995ad6a557",
          + "subnet-0d9ac99195f2661b7",
          + "subnet-0fd7ed03d1c9cc941",
        ]
      + wait_for_capacity_timeout        = "10m"
      + warm_pool_size                   = (known after apply)

      + launch_template (known after apply)

      + mixed_instances_policy (known after apply)

      + tag {
          + key                 = "Name"
          + propagate_at_launch = true
          + value               = "terraform-asg-example"
        }

      + traffic_source (known after apply)
    }

  # aws_launch_configuration.example will be created
  + resource "aws_launch_configuration" "example" {
      + arn                         = (known after apply)
      + associate_public_ip_address = (known after apply)
      + ebs_optimized               = (known after apply)
      + enable_monitoring           = true
      + id                          = (known after apply)
      + image_id                    = "ami-04a81a99f5ec58529"
      + instance_type               = "t2.micro"
      + key_name                    = (known after apply)
      + name                        = (known after apply)
      + name_prefix                 = (known after apply)
      + security_groups             = (known after apply)
      + user_data                   = "c765373c563b260626d113c4a56a46e8a8c5ca33"

      + ebs_block_device (known after apply)

      + metadata_options (known after apply)

      + root_block_device (known after apply)
    }

  # aws_lb.example will be created
  + resource "aws_lb" "example" {
      + arn                                                          = (known after apply)
      + arn_suffix                                                   = (known after apply)
      + client_keep_alive                                            = 3600
      + desync_mitigation_mode                                       = "defensive"
      + dns_name                                                     = (known after apply)
      + drop_invalid_header_fields                                   = false
      + enable_deletion_protection                                   = false
      + enable_http2                                                 = true
      + enable_tls_version_and_cipher_suite_headers                  = false
      + enable_waf_fail_open                                         = false
      + enable_xff_client_port                                       = false
      + enforce_security_group_inbound_rules_on_private_link_traffic = (known after apply)
      + id                                                           = (known after apply)
      + idle_timeout                                                 = 60
      + internal                                                     = (known after apply)
      + ip_address_type                                              = (known after apply)
      + load_balancer_type                                           = "application"
      + name                                                         = "terraform-asg-example"
      + name_prefix                                                  = (known after apply)
      + preserve_host_header                                         = false
      + security_groups                                              = (known after apply)
      + subnets                                                      = [
          + "subnet-013196de977c33b7d",
          + "subnet-075866766bdfb54d4",
          + "subnet-0c316180bdeb61390",
          + "subnet-0cee7f5995ad6a557",
          + "subnet-0d9ac99195f2661b7",
          + "subnet-0fd7ed03d1c9cc941",
        ]
      + tags_all                                                     = (known after apply)
      + vpc_id                                                       = (known after apply)
      + xff_header_processing_mode                                   = "append"
      + zone_id                                                      = (known after apply)

      + subnet_mapping (known after apply)
    }

  # aws_lb_listener.http will be created
  + resource "aws_lb_listener" "http" {
      + arn               = (known after apply)
      + id                = (known after apply)
      + load_balancer_arn = (known after apply)
      + port              = 80
      + protocol          = "HTTP"
      + ssl_policy        = (known after apply)
      + tags_all          = (known after apply)

      + default_action {
          + order = (known after apply)
          + type  = "fixed-response"

          + fixed_response {
              + content_type = "text/plain"
              + message_body = "404: page not found"
              + status_code  = "404"
            }
        }

      + mutual_authentication (known after apply)
    }

  # aws_lb_listener_rule.asg will be created
  + resource "aws_lb_listener_rule" "asg" {
      + arn          = (known after apply)
      + id           = (known after apply)
      + listener_arn = (known after apply)
      + priority     = 100
      + tags_all     = (known after apply)

      + action {
          + order            = (known after apply)
          + target_group_arn = (known after apply)
          + type             = "forward"
        }

      + condition {
          + path_pattern {
              + values = [
                  + "*",
                ]
            }
        }
    }

  # aws_lb_target_group.asg will be created
  + resource "aws_lb_target_group" "asg" {
      + arn                                = (known after apply)
      + arn_suffix                         = (known after apply)
      + connection_termination             = (known after apply)
      + deregistration_delay               = "300"
      + id                                 = (known after apply)
      + ip_address_type                    = (known after apply)
      + lambda_multi_value_headers_enabled = false
      + load_balancer_arns                 = (known after apply)
      + load_balancing_algorithm_type      = (known after apply)
      + load_balancing_anomaly_mitigation  = (known after apply)
      + load_balancing_cross_zone_enabled  = (known after apply)
      + name                               = "terraform-asg-example"
      + name_prefix                        = (known after apply)
      + port                               = 8080
      + preserve_client_ip                 = (known after apply)
      + protocol                           = "HTTP"
      + protocol_version                   = (known after apply)
      + proxy_protocol_v2                  = false
      + slow_start                         = 0
      + tags_all                           = (known after apply)
      + target_type                        = "instance"
      + vpc_id                             = "vpc-01dd39250fbbe13c6"

      + health_check {
          + enabled             = true
          + healthy_threshold   = 2
          + interval            = 15
          + matcher             = "200"
          + path                = "/"
          + port                = "traffic-port"
          + protocol            = "HTTP"
          + timeout             = 3
          + unhealthy_threshold = 2
        }

      + stickiness (known after apply)

      + target_failover (known after apply)

      + target_group_health (known after apply)

      + target_health_state (known after apply)
    }

  # aws_security_group.alb will be created
  + resource "aws_security_group" "alb" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + from_port        = 0
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "-1"
              + security_groups  = []
              + self             = false
              + to_port          = 0
                # (1 unchanged attribute hidden)
            },
        ]
      + id                     = (known after apply)
      + ingress                = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + from_port        = 80
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 80
                # (1 unchanged attribute hidden)
            },
        ]
      + name                   = "terraform-example-alb"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags_all               = (known after apply)
      + vpc_id                 = (known after apply)
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

Plan: 8 to add, 0 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
cloud_user@553b1e446c1c:~/asg_elb_terraform$
```
</details><br />

<details markdown=1>
<summary markdown="span">terraform apply</summary>

```bash
cloud_user@553b1e446c1c:~/asg_elb_terraform$ terraform apply
data.aws_vpc.default: Reading...
data.aws_vpc.default: Read complete after 1s [id=vpc-01dd39250fbbe13c6]
data.aws_subnets.default: Reading...
data.aws_subnets.default: Read complete after 0s [id=us-east-1]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_autoscaling_group.example will be created
  + resource "aws_autoscaling_group" "example" {
      + arn                              = (known after apply)
      + availability_zones               = (known after apply)
      + default_cooldown                 = (known after apply)
      + desired_capacity                 = (known after apply)
      + force_delete                     = false
      + force_delete_warm_pool           = false
      + health_check_grace_period        = 300
      + health_check_type                = "ELB"
      + id                               = (known after apply)
      + ignore_failed_scaling_activities = false
      + launch_configuration             = (known after apply)
      + load_balancers                   = (known after apply)
      + max_size                         = 10
      + metrics_granularity              = "1Minute"
      + min_size                         = 2
      + name                             = (known after apply)
      + name_prefix                      = (known after apply)
      + predicted_capacity               = (known after apply)
      + protect_from_scale_in            = false
      + service_linked_role_arn          = (known after apply)
      + target_group_arns                = (known after apply)
      + vpc_zone_identifier              = [
          + "subnet-013196de977c33b7d",
          + "subnet-075866766bdfb54d4",
          + "subnet-0c316180bdeb61390",
          + "subnet-0cee7f5995ad6a557",
          + "subnet-0d9ac99195f2661b7",
          + "subnet-0fd7ed03d1c9cc941",
        ]
      + wait_for_capacity_timeout        = "10m"
      + warm_pool_size                   = (known after apply)

      + launch_template (known after apply)

      + mixed_instances_policy (known after apply)

      + tag {
          + key                 = "Name"
          + propagate_at_launch = true
          + value               = "terraform-asg-example"
        }

      + traffic_source (known after apply)
    }

  # aws_launch_configuration.example will be created
  + resource "aws_launch_configuration" "example" {
      + arn                         = (known after apply)
      + associate_public_ip_address = (known after apply)
      + ebs_optimized               = (known after apply)
      + enable_monitoring           = true
      + id                          = (known after apply)
      + image_id                    = "ami-04a81a99f5ec58529"
      + instance_type               = "t2.micro"
      + key_name                    = (known after apply)
      + name                        = (known after apply)
      + name_prefix                 = (known after apply)
      + security_groups             = (known after apply)
      + user_data                   = "c765373c563b260626d113c4a56a46e8a8c5ca33"

      + ebs_block_device (known after apply)

      + metadata_options (known after apply)

      + root_block_device (known after apply)
    }

  # aws_lb.example will be created
  + resource "aws_lb" "example" {
      + arn                                                          = (known after apply)
      + arn_suffix                                                   = (known after apply)
      + client_keep_alive                                            = 3600
      + desync_mitigation_mode                                       = "defensive"
      + dns_name                                                     = (known after apply)
      + drop_invalid_header_fields                                   = false
      + enable_deletion_protection                                   = false
      + enable_http2                                                 = true
      + enable_tls_version_and_cipher_suite_headers                  = false
      + enable_waf_fail_open                                         = false
      + enable_xff_client_port                                       = false
      + enforce_security_group_inbound_rules_on_private_link_traffic = (known after apply)
      + id                                                           = (known after apply)
      + idle_timeout                                                 = 60
      + internal                                                     = (known after apply)
      + ip_address_type                                              = (known after apply)
      + load_balancer_type                                           = "application"
      + name                                                         = "terraform-asg-example"
      + name_prefix                                                  = (known after apply)
      + preserve_host_header                                         = false
      + security_groups                                              = (known after apply)
      + subnets                                                      = [
          + "subnet-013196de977c33b7d",
          + "subnet-075866766bdfb54d4",
          + "subnet-0c316180bdeb61390",
          + "subnet-0cee7f5995ad6a557",
          + "subnet-0d9ac99195f2661b7",
          + "subnet-0fd7ed03d1c9cc941",
        ]
      + tags_all                                                     = (known after apply)
      + vpc_id                                                       = (known after apply)
      + xff_header_processing_mode                                   = "append"
      + zone_id                                                      = (known after apply)

      + subnet_mapping (known after apply)
    }

  # aws_lb_listener.http will be created
  + resource "aws_lb_listener" "http" {
      + arn               = (known after apply)
      + id                = (known after apply)
      + load_balancer_arn = (known after apply)
      + port              = 80
      + protocol          = "HTTP"
      + ssl_policy        = (known after apply)
      + tags_all          = (known after apply)

      + default_action {
          + order = (known after apply)
          + type  = "fixed-response"

          + fixed_response {
              + content_type = "text/plain"
              + message_body = "404: page not found"
              + status_code  = "404"
            }
        }

      + mutual_authentication (known after apply)
    }

  # aws_lb_listener_rule.asg will be created
  + resource "aws_lb_listener_rule" "asg" {
      + arn          = (known after apply)
      + id           = (known after apply)
      + listener_arn = (known after apply)
      + priority     = 100
      + tags_all     = (known after apply)

      + action {
          + order            = (known after apply)
          + target_group_arn = (known after apply)
          + type             = "forward"
        }

      + condition {
          + path_pattern {
              + values = [
                  + "*",
                ]
            }
        }
    }

  # aws_lb_target_group.asg will be created
  + resource "aws_lb_target_group" "asg" {
      + arn                                = (known after apply)
      + arn_suffix                         = (known after apply)
      + connection_termination             = (known after apply)
      + deregistration_delay               = "300"
      + id                                 = (known after apply)
      + ip_address_type                    = (known after apply)
      + lambda_multi_value_headers_enabled = false
      + load_balancer_arns                 = (known after apply)
      + load_balancing_algorithm_type      = (known after apply)
      + load_balancing_anomaly_mitigation  = (known after apply)
      + load_balancing_cross_zone_enabled  = (known after apply)
      + name                               = "terraform-asg-example"
      + name_prefix                        = (known after apply)
      + port                               = 8080
      + preserve_client_ip                 = (known after apply)
      + protocol                           = "HTTP"
      + protocol_version                   = (known after apply)
      + proxy_protocol_v2                  = false
      + slow_start                         = 0
      + tags_all                           = (known after apply)
      + target_type                        = "instance"
      + vpc_id                             = "vpc-01dd39250fbbe13c6"

      + health_check {
          + enabled             = true
          + healthy_threshold   = 2
          + interval            = 15
          + matcher             = "200"
          + path                = "/"
          + port                = "traffic-port"
          + protocol            = "HTTP"
          + timeout             = 3
          + unhealthy_threshold = 2
        }

      + stickiness (known after apply)

      + target_failover (known after apply)

      + target_group_health (known after apply)

      + target_health_state (known after apply)
    }

  # aws_security_group.alb will be created
  + resource "aws_security_group" "alb" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + from_port        = 0
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "-1"
              + security_groups  = []
              + self             = false
              + to_port          = 0
                # (1 unchanged attribute hidden)
            },
        ]
      + id                     = (known after apply)
      + ingress                = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + from_port        = 80
              + ipv6_cidr_blocks = []
              + prefix_list_ids  = []
              + protocol         = "tcp"
              + security_groups  = []
              + self             = false
              + to_port          = 80
                # (1 unchanged attribute hidden)
            },
        ]
      + name                   = "terraform-example-alb"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags_all               = (known after apply)
      + vpc_id                 = (known after apply)
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

Plan: 8 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_lb_target_group.asg: Creating...
aws_security_group.instance: Creating...
aws_security_group.alb: Creating...
aws_lb_target_group.asg: Creation complete after 0s [id=arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:targetgroup/terraform-asg-example/f7291f3c616cee2c]
aws_security_group.instance: Creation complete after 2s [id=sg-0ebaa3fd8741f2114]
aws_launch_configuration.example: Creating...
aws_security_group.alb: Creation complete after 2s [id=sg-0e7699ba72233ec10]
aws_lb.example: Creating...
aws_launch_configuration.example: Creation complete after 0s [id=terraform-20240724032221829400000001]
aws_autoscaling_group.example: Creating...
aws_lb.example: Still creating... [10s elapsed]
aws_autoscaling_group.example: Still creating... [10s elapsed]
aws_lb.example: Still creating... [20s elapsed]
aws_autoscaling_group.example: Still creating... [20s elapsed]
aws_lb.example: Still creating... [30s elapsed]
aws_autoscaling_group.example: Still creating... [30s elapsed]
aws_lb.example: Still creating... [40s elapsed]
aws_autoscaling_group.example: Still creating... [40s elapsed]
aws_autoscaling_group.example: Creation complete after 46s [id=terraform-20240724032222449100000002]
aws_lb.example: Still creating... [50s elapsed]
aws_lb.example: Still creating... [1m0s elapsed]
aws_lb.example: Still creating... [1m10s elapsed]
aws_lb.example: Still creating... [1m20s elapsed]
aws_lb.example: Still creating... [1m30s elapsed]
aws_lb.example: Still creating... [1m40s elapsed]
aws_lb.example: Still creating... [1m50s elapsed]
aws_lb.example: Still creating... [2m0s elapsed]
aws_lb.example: Still creating... [2m10s elapsed]
aws_lb.example: Still creating... [2m20s elapsed]
aws_lb.example: Still creating... [2m30s elapsed]
aws_lb.example: Still creating... [2m40s elapsed]
aws_lb.example: Still creating... [2m50s elapsed]
aws_lb.example: Still creating... [3m0s elapsed]
aws_lb.example: Still creating... [3m10s elapsed]
aws_lb.example: Creation complete after 3m12s [id=arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:loadbalancer/app/terraform-asg-example/e8f8ce77a4f9d74c]
aws_lb_listener.http: Creating...
aws_lb_listener.http: Creation complete after 1s [id=arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:listener/app/terraform-asg-example/e8f8ce77a4f9d74c/ab66348839329aee]
aws_lb_listener_rule.asg: Creating...
aws_lb_listener_rule.asg: Creation complete after 0s [id=arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:listener-rule/app/terraform-asg-example/e8f8ce77a4f9d74c/ab66348839329aee/36b4f1e473ed77ee]

Apply complete! Resources: 8 added, 0 changed, 0 destroyed.
cloud_user@553b1e446c1c:~/asg_elb_terraform$
```
</details><br />


```bash
cloud_user@553b1e446c1c:~/asg_elb_terraform$ terraform output
alb_dns_name = "terraform-asg-example-XXXXXXXXXX.us-east-1.elb.amazonaws.com"
cloud_user@553b1e446c1c:~/asg_elb_terraform$
```

## Verification

```bash
$ curl terraform-asg-example-XXXXXXXXXX.us-east-1.elb.amazonaws.com
Hello, World
$
```

EC2 instances created

![]({{ site.baseurl }}/images/2024/07-23-Terraform_102_asg_elb/02-EC2-instances.png)

Security Groups

![]({{ site.baseurl }}/images/2024/07-23-Terraform_102_asg_elb/03-Security_Groups.png)

Elastic Load Balancer

![]({{ site.baseurl }}/images/2024/07-23-Terraform_102_asg_elb/04-ELB.png)

Target Groups

![]({{ site.baseurl }}/images/2024/07-23-Terraform_102_asg_elb/05-Target-groups.png)

![]({{ site.baseurl }}/images/2024/07-23-Terraform_102_asg_elb/06-Registered-targets.png)

Listeners

![]({{ site.baseurl }}/images/2024/07-23-Terraform_102_asg_elb/07-Listeners.png)

Auto Scaling groups

![]({{ site.baseurl }}/images/2024/07-23-Terraform_102_asg_elb/08-Auto-Scaling-groups.png)

Launch configuration

![]({{ site.baseurl }}/images/2024/07-23-Terraform_102_asg_elb/09-Launch-configuration.png)

## Removing the infrastructure

```bash
terraform destroy
```

<details markdown=1>
<summary markdown="span">terrafom destroy</summary>

```bash
cloud_user@553b1e446c1c:~/asg_elb_terraform$ terraform destroy
aws_security_group.instance: Refreshing state... [id=sg-0ebaa3fd8741f2114]
aws_security_group.alb: Refreshing state... [id=sg-0e7699ba72233ec10]
data.aws_vpc.default: Reading...
aws_launch_configuration.example: Refreshing state... [id=terraform-20240724032221829400000001]
data.aws_vpc.default: Read complete after 0s [id=vpc-01dd39250fbbe13c6]
data.aws_subnets.default: Reading...
aws_lb_target_group.asg: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:targetgroup/terraform-asg-example/f7291f3c616cee2c]
data.aws_subnets.default: Read complete after 0s [id=us-east-1]
aws_lb.example: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:loadbalancer/app/terraform-asg-example/e8f8ce77a4f9d74c]
aws_autoscaling_group.example: Refreshing state... [id=terraform-20240724032222449100000002]
aws_lb_listener.http: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:listener/app/terraform-asg-example/e8f8ce77a4f9d74c/ab66348839329aee]
aws_lb_listener_rule.asg: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:listener-rule/app/terraform-asg-example/e8f8ce77a4f9d74c/ab66348839329aee/36b4f1e473ed77ee]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_autoscaling_group.example will be destroyed
  - resource "aws_autoscaling_group" "example" {
      - arn                              = "arn:aws:autoscaling:us-east-1:XXXXXXXXXXXX:autoScalingGroup:7b82e885-eb1a-4c85-862f-72df516c1f2e:autoScalingGroupName/terraform-20240724032222449100000002" -> null
      - availability_zones               = [
          - "us-east-1a",
          - "us-east-1b",
          - "us-east-1c",
          - "us-east-1d",
          - "us-east-1e",
          - "us-east-1f",
        ] -> null
      - capacity_rebalance               = false -> null
      - default_cooldown                 = 300 -> null
      - default_instance_warmup          = 0 -> null
      - desired_capacity                 = 2 -> null
      - enabled_metrics                  = [] -> null
      - force_delete                     = false -> null
      - force_delete_warm_pool           = false -> null
      - health_check_grace_period        = 300 -> null
      - health_check_type                = "ELB" -> null
      - id                               = "terraform-20240724032222449100000002" -> null
      - ignore_failed_scaling_activities = false -> null
      - launch_configuration             = "terraform-20240724032221829400000001" -> null
      - load_balancers                   = [] -> null
      - max_instance_lifetime            = 0 -> null
      - max_size                         = 10 -> null
      - metrics_granularity              = "1Minute" -> null
      - min_size                         = 2 -> null
      - name                             = "terraform-20240724032222449100000002" -> null
      - name_prefix                      = "terraform-" -> null
      - predicted_capacity               = 0 -> null
      - protect_from_scale_in            = false -> null
      - service_linked_role_arn          = "arn:aws:iam::XXXXXXXXXXXX:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling" -> null
      - suspended_processes              = [] -> null
      - target_group_arns                = [
          - "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:targetgroup/terraform-asg-example/f7291f3c616cee2c",
        ] -> null
      - termination_policies             = [] -> null
      - vpc_zone_identifier              = [
          - "subnet-013196de977c33b7d",
          - "subnet-075866766bdfb54d4",
          - "subnet-0c316180bdeb61390",
          - "subnet-0cee7f5995ad6a557",
          - "subnet-0d9ac99195f2661b7",
          - "subnet-0fd7ed03d1c9cc941",
        ] -> null
      - wait_for_capacity_timeout        = "10m" -> null
      - warm_pool_size                   = 0 -> null
        # (3 unchanged attributes hidden)

      - tag {
          - key                 = "Name" -> null
          - propagate_at_launch = true -> null
          - value               = "terraform-asg-example" -> null
        }

      - traffic_source {
          - identifier = "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:targetgroup/terraform-asg-example/f7291f3c616cee2c" -> null
          - type       = "elbv2" -> null
        }
    }

  # aws_launch_configuration.example will be destroyed
  - resource "aws_launch_configuration" "example" {
      - arn                         = "arn:aws:autoscaling:us-east-1:XXXXXXXXXXXX:launchConfiguration:89effdc0-5c09-4858-b413-d4e8aa637cc7:launchConfigurationName/terraform-20240724032221829400000001" -> null
      - associate_public_ip_address = false -> null
      - ebs_optimized               = false -> null
      - enable_monitoring           = true -> null
      - id                          = "terraform-20240724032221829400000001" -> null
      - image_id                    = "ami-04a81a99f5ec58529" -> null
      - instance_type               = "t2.micro" -> null
      - name                        = "terraform-20240724032221829400000001" -> null
      - name_prefix                 = "terraform-" -> null
      - security_groups             = [
          - "sg-0ebaa3fd8741f2114",
        ] -> null
      - user_data                   = "c765373c563b260626d113c4a56a46e8a8c5ca33" -> null
        # (4 unchanged attributes hidden)
    }

  # aws_lb.example will be destroyed
  - resource "aws_lb" "example" {
      - arn                                                          = "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:loadbalancer/app/terraform-asg-example/e8f8ce77a4f9d74c" -> null
      - arn_suffix                                                   = "app/terraform-asg-example/e8f8ce77a4f9d74c" -> null
      - client_keep_alive                                            = 3600 -> null
      - desync_mitigation_mode                                       = "defensive" -> null
      - dns_name                                                     = "terraform-asg-example-1620849536.us-east-1.elb.amazonaws.com" -> null
      - drop_invalid_header_fields                                   = false -> null
      - enable_cross_zone_load_balancing                             = true -> null
      - enable_deletion_protection                                   = false -> null
      - enable_http2                                                 = true -> null
      - enable_tls_version_and_cipher_suite_headers                  = false -> null
      - enable_waf_fail_open                                         = false -> null
      - enable_xff_client_port                                       = false -> null
      - id                                                           = "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:loadbalancer/app/terraform-asg-example/e8f8ce77a4f9d74c" -> null
      - idle_timeout                                                 = 60 -> null
      - internal                                                     = false -> null
      - ip_address_type                                              = "ipv4" -> null
      - load_balancer_type                                           = "application" -> null
      - name                                                         = "terraform-asg-example" -> null
      - preserve_host_header                                         = false -> null
      - security_groups                                              = [
          - "sg-0e7699ba72233ec10",
        ] -> null
      - subnets                                                      = [
          - "subnet-013196de977c33b7d",
          - "subnet-075866766bdfb54d4",
          - "subnet-0c316180bdeb61390",
          - "subnet-0cee7f5995ad6a557",
          - "subnet-0d9ac99195f2661b7",
          - "subnet-0fd7ed03d1c9cc941",
        ] -> null
      - tags                                                         = {} -> null
      - tags_all                                                     = {} -> null
      - vpc_id                                                       = "vpc-01dd39250fbbe13c6" -> null
      - xff_header_processing_mode                                   = "append" -> null
      - zone_id                                                      = "Z35SXDOTRQ7X7K" -> null
        # (3 unchanged attributes hidden)

      - access_logs {
          - enabled = false -> null
            # (2 unchanged attributes hidden)
        }

      - connection_logs {
          - enabled = false -> null
            # (2 unchanged attributes hidden)
        }

      - subnet_mapping {
          - subnet_id            = "subnet-013196de977c33b7d" -> null
            # (4 unchanged attributes hidden)
        }
      - subnet_mapping {
          - subnet_id            = "subnet-075866766bdfb54d4" -> null
            # (4 unchanged attributes hidden)
        }
      - subnet_mapping {
          - subnet_id            = "subnet-0c316180bdeb61390" -> null
            # (4 unchanged attributes hidden)
        }
      - subnet_mapping {
          - subnet_id            = "subnet-0cee7f5995ad6a557" -> null
            # (4 unchanged attributes hidden)
        }
      - subnet_mapping {
          - subnet_id            = "subnet-0d9ac99195f2661b7" -> null
            # (4 unchanged attributes hidden)
        }
      - subnet_mapping {
          - subnet_id            = "subnet-0fd7ed03d1c9cc941" -> null
            # (4 unchanged attributes hidden)
        }
    }

  # aws_lb_listener.http will be destroyed
  - resource "aws_lb_listener" "http" {
      - arn               = "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:listener/app/terraform-asg-example/e8f8ce77a4f9d74c/ab66348839329aee" -> null
      - id                = "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:listener/app/terraform-asg-example/e8f8ce77a4f9d74c/ab66348839329aee" -> null
      - load_balancer_arn = "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:loadbalancer/app/terraform-asg-example/e8f8ce77a4f9d74c" -> null
      - port              = 80 -> null
      - protocol          = "HTTP" -> null
      - tags              = {} -> null
      - tags_all          = {} -> null
        # (1 unchanged attribute hidden)

      - default_action {
          - order            = 1 -> null
          - type             = "fixed-response" -> null
            # (1 unchanged attribute hidden)

          - fixed_response {
              - content_type = "text/plain" -> null
              - message_body = "404: page not found" -> null
              - status_code  = "404" -> null
            }
        }
    }

  # aws_lb_listener_rule.asg will be destroyed
  - resource "aws_lb_listener_rule" "asg" {
      - arn          = "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:listener-rule/app/terraform-asg-example/e8f8ce77a4f9d74c/ab66348839329aee/36b4f1e473ed77ee" -> null
      - id           = "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:listener-rule/app/terraform-asg-example/e8f8ce77a4f9d74c/ab66348839329aee/36b4f1e473ed77ee" -> null
      - listener_arn = "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:listener/app/terraform-asg-example/e8f8ce77a4f9d74c/ab66348839329aee" -> null
      - priority     = 100 -> null
      - tags         = {} -> null
      - tags_all     = {} -> null

      - action {
          - order            = 1 -> null
          - target_group_arn = "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:targetgroup/terraform-asg-example/f7291f3c616cee2c" -> null
          - type             = "forward" -> null
        }

      - condition {
          - path_pattern {
              - values = [
                  - "*",
                ] -> null
            }
        }
    }

  # aws_lb_target_group.asg will be destroyed
  - resource "aws_lb_target_group" "asg" {
      - arn                                = "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:targetgroup/terraform-asg-example/f7291f3c616cee2c" -> null
      - arn_suffix                         = "targetgroup/terraform-asg-example/f7291f3c616cee2c" -> null
      - deregistration_delay               = "300" -> null
      - id                                 = "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:targetgroup/terraform-asg-example/f7291f3c616cee2c" -> null
      - ip_address_type                    = "ipv4" -> null
      - lambda_multi_value_headers_enabled = false -> null
      - load_balancer_arns                 = [
          - "arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:loadbalancer/app/terraform-asg-example/e8f8ce77a4f9d74c",
        ] -> null
      - load_balancing_algorithm_type      = "round_robin" -> null
      - load_balancing_anomaly_mitigation  = "off" -> null
      - load_balancing_cross_zone_enabled  = "use_load_balancer_configuration" -> null
      - name                               = "terraform-asg-example" -> null
      - port                               = 8080 -> null
      - protocol                           = "HTTP" -> null
      - protocol_version                   = "HTTP1" -> null
      - proxy_protocol_v2                  = false -> null
      - slow_start                         = 0 -> null
      - tags                               = {} -> null
      - tags_all                           = {} -> null
      - target_type                        = "instance" -> null
      - vpc_id                             = "vpc-01dd39250fbbe13c6" -> null
        # (1 unchanged attribute hidden)

      - health_check {
          - enabled             = true -> null
          - healthy_threshold   = 2 -> null
          - interval            = 15 -> null
          - matcher             = "200" -> null
          - path                = "/" -> null
          - port                = "traffic-port" -> null
          - protocol            = "HTTP" -> null
          - timeout             = 3 -> null
          - unhealthy_threshold = 2 -> null
        }

      - stickiness {
          - cookie_duration = 86400 -> null
          - enabled         = false -> null
          - type            = "lb_cookie" -> null
            # (1 unchanged attribute hidden)
        }

      - target_failover {}

      - target_group_health {
          - dns_failover {
              - minimum_healthy_targets_count      = "1" -> null
              - minimum_healthy_targets_percentage = "off" -> null
            }
          - unhealthy_state_routing {
              - minimum_healthy_targets_count      = 1 -> null
              - minimum_healthy_targets_percentage = "off" -> null
            }
        }

      - target_health_state {}
    }

  # aws_security_group.alb will be destroyed
  - resource "aws_security_group" "alb" {
      - arn                    = "arn:aws:ec2:us-east-1:XXXXXXXXXXXX:security-group/sg-0e7699ba72233ec10" -> null
      - description            = "Managed by Terraform" -> null
      - egress                 = [
          - {
              - cidr_blocks      = [
                  - "0.0.0.0/0",
                ]
              - from_port        = 0
              - ipv6_cidr_blocks = []
              - prefix_list_ids  = []
              - protocol         = "-1"
              - security_groups  = []
              - self             = false
              - to_port          = 0
                # (1 unchanged attribute hidden)
            },
        ] -> null
      - id                     = "sg-0e7699ba72233ec10" -> null
      - ingress                = [
          - {
              - cidr_blocks      = [
                  - "0.0.0.0/0",
                ]
              - from_port        = 80
              - ipv6_cidr_blocks = []
              - prefix_list_ids  = []
              - protocol         = "tcp"
              - security_groups  = []
              - self             = false
              - to_port          = 80
                # (1 unchanged attribute hidden)
            },
        ] -> null
      - name                   = "terraform-example-alb" -> null
      - owner_id               = "XXXXXXXXXXXX" -> null
      - revoke_rules_on_delete = false -> null
      - tags                   = {} -> null
      - tags_all               = {} -> null
      - vpc_id                 = "vpc-01dd39250fbbe13c6" -> null
        # (1 unchanged attribute hidden)
    }

  # aws_security_group.instance will be destroyed
  - resource "aws_security_group" "instance" {
      - arn                    = "arn:aws:ec2:us-east-1:XXXXXXXXXXXX:security-group/sg-0ebaa3fd8741f2114" -> null
      - description            = "Managed by Terraform" -> null
      - egress                 = [] -> null
      - id                     = "sg-0ebaa3fd8741f2114" -> null
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
      - owner_id               = "XXXXXXXXXXXX" -> null
      - revoke_rules_on_delete = false -> null
      - tags                   = {} -> null
      - tags_all               = {} -> null
      - vpc_id                 = "vpc-01dd39250fbbe13c6" -> null
        # (1 unchanged attribute hidden)
    }

Plan: 0 to add, 0 to change, 8 to destroy.

Changes to Outputs:
  - alb_dns_name = "terraform-asg-example-1620849536.us-east-1.elb.amazonaws.com" -> null

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

aws_lb_listener_rule.asg: Destroying... [id=arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:listener-rule/app/terraform-asg-example/e8f8ce77a4f9d74c/ab66348839329aee/36b4f1e473ed77ee]
aws_autoscaling_group.example: Destroying... [id=terraform-20240724032222449100000002]
aws_lb_listener_rule.asg: Destruction complete after 0s
aws_lb_listener.http: Destroying... [id=arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:listener/app/terraform-asg-example/e8f8ce77a4f9d74c/ab66348839329aee]
aws_lb_listener.http: Destruction complete after 0s
aws_lb.example: Destroying... [id=arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:loadbalancer/app/terraform-asg-example/e8f8ce77a4f9d74c]
aws_lb.example: Destruction complete after 3s
aws_security_group.alb: Destroying... [id=sg-0e7699ba72233ec10]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724032222449100000002, 10s elapsed]
aws_security_group.alb: Still destroying... [id=sg-0e7699ba72233ec10, 10s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724032222449100000002, 20s elapsed]
aws_security_group.alb: Still destroying... [id=sg-0e7699ba72233ec10, 20s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724032222449100000002, 30s elapsed]
aws_security_group.alb: Still destroying... [id=sg-0e7699ba72233ec10, 30s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724032222449100000002, 40s elapsed]
aws_security_group.alb: Still destroying... [id=sg-0e7699ba72233ec10, 40s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724032222449100000002, 50s elapsed]
aws_security_group.alb: Still destroying... [id=sg-0e7699ba72233ec10, 50s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724032222449100000002, 1m0s elapsed]
aws_security_group.alb: Destruction complete after 59s
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724032222449100000002, 1m10s elapsed]
aws_autoscaling_group.example: Still destroying... [id=terraform-20240724032222449100000002, 1m20s elapsed]
aws_autoscaling_group.example: Destruction complete after 1m23s
aws_launch_configuration.example: Destroying... [id=terraform-20240724032221829400000001]
aws_lb_target_group.asg: Destroying... [id=arn:aws:elasticloadbalancing:us-east-1:XXXXXXXXXXXX:targetgroup/terraform-asg-example/f7291f3c616cee2c]
aws_lb_target_group.asg: Destruction complete after 0s
aws_launch_configuration.example: Destruction complete after 0s
aws_security_group.instance: Destroying... [id=sg-0ebaa3fd8741f2114]
aws_security_group.instance: Destruction complete after 1s

Destroy complete! Resources: 8 destroyed.
cloud_user@553b1e446c1c:~/asg_elb_terraform$
```
</details><br />

References:

- [Terraform: Up & Running By Yevgeniy Brikman](https://www.terraformupandrunning.com/)
- [alexlouden / Terraform.tmLanguage](https://github.com/alexlouden/Terraform.tmLanguage)
