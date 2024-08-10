---
title: Terraform Modules
date: 2024-08-09 20:00:00 -0700
categories: [TERRAFORM, MODULES]
tags: [terraform]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/terraform_logo.png)

Terraform Modules are similar to **functions** in programming languages such as Python. It has inputs and outputs.

- Any set of Terraform configuration files in a folder is a **module**
  - **root module** is when you run `terraform apply` to execute **terraform code** in a directory
  - **reusable module**, is **terraform code** meant to be used by other modules

- Providers should be configured only in **root modules** and not in **reusable modules**

## Folder hierarchy

The folder hierarchy outlined for this terraform practice is

- for the **root module**

```bash
cloud_user@553b1e446c1c:~$ tree root_module/
root_module/
├── main.tf
├── outputs.tf
└── variables.tf

0 directories, 3 files
cloud_user@553b1e446c1c:~$
```

- for the **reusable module**

```bash
cloud_user@553b1e446c1c:~$ tree reusable_module/
reusable_module/
├── main.tf
├── outputs.tf
├── user-data.sh
└── variables.tf

0 directories, 4 files
cloud_user@553b1e446c1c:~$
```

## Calling a module

This is the syntax for calling a module. This needs to go in `main.tf` in the **root module**. It is just used to call the **reusable module**

```tf
module "<NAME>" {
  source = "<SOURCE>"

  [CONFIG ...]
}
```

This is an example of a `main.tf` showing how to call a module. **NOTE** that this example has no **input variables** defined yet

```tf
provider "aws" {
  region = "us-east-1"
}

module "webserver_cluster" {
  source = "../modules/services/webserver-cluster"
}
```

When adding a module to the Terraform configurations or modify the source parameter of a module, we need to run the `terraform init` command before running `terraform plan` or `terraform apply`.

The `terraform init` command does:

1. installs providers
2. configures the backends
3. downloads modules

<details markdown=1>
<summary markdown="span">output</summary>

**NOTE:** We need to have the module created first before running `terraform init`. The error seen here is expected since we have not created the module yet.

```bash
cloud_user@553b1e446c1c:~$ mkdir root_module
cloud_user@553b1e446c1c:~$ cd root_module/
cloud_user@553b1e446c1c:~/root_module$

cloud_user@553b1e446c1c:~/root_module$ pwd
/home/cloud_user/root_module
cloud_user@553b1e446c1c:~/root_module$

cloud_user@553b1e446c1c:~/root_module$ cat main.tf
provider "aws" {
  region = "us-east-1"
}

module "webserver_cluster" {
  source = "../modules/services/webserver-cluster"
cloud_user@553b1e446c1c:~/root_module$

cloud_user@553b1e446c1c:~/root_module$ terraform init
Initializing the backend...
Initializing modules...
- webserver_cluster in
╷
│ Error: Unreadable module directory
│
│ Unable to evaluate directory symlink: lstat ../modules: no such file or directory
╵
╷
│ Error: Unreadable module directory
│
│ The directory  could not be read for module "webserver_cluster" at main.tf:5.
╵
cloud_user@553b1e446c1c:~/root_module$
```
</details><br />

## Creating the Reusable Module

We will now create a module that deploys and **Application Load Balancer** and a **Auto Scaling Group** that will spin-up **EC2 instances**

All these files are part of

```bash
cloud_user@553b1e446c1c:~$ tree reusable_module/
reusable_module/
├── main.tf
├── outputs.tf
├── user-data.sh
└── variables.tf

0 directories, 4 files
cloud_user@553b1e446c1c:~$
```

### Module Inputs

**Input variables** for a module are set by using the same syntax as setting **arguments** for a **resource**.

The **input variable** are the API of the module, controlling how it will behave in different environments.

These are the **Input variables** of this Terraform module.

- `variables.tf`

```tf
# -----------------------------------------------------
# REQUIRED PARAMETERS
# You must provide a value for each of these parameters
# -----------------------------------------------------

variable "cluster_name" {
  description = "The name to use for all the cluster resources"
  type        = string
}

variable "instance_type" {
  description = "The type of EC2 Instances to run (e.g. t2.micro)"
  type        = string
}

variable "min_size" {
  description = "The minimum number of EC2 Instances in the ASG"
  type        = number
}

variable "max_size" {
  description = "The maximum number of EC2 Instances in the ASG"
  type        = number
}

# -----------------------------------------------------
# REQUIRED PARAMETERS
# You must provide a value for each of these parameters
# These will be displayed in the HTML page of the WebServer
# -----------------------------------------------------

variable "main_message" {
  description = "Main message to be displayed in the WebSite"
  type        = string
}

variable "sub_message" {
  description = "Sub message to be displayed in the WebSite"
  type        = string
}

# -----------------------------------------------------
# OPTIONAL PARAMETERS
# These parameters have reasonable defaults
# -----------------------------------------------------

variable "server_port" {
  description = "The port the EC2 instaces will use for HTTP requests"
  type        = number
  default     = 8080
}
```

Example of how to use the variable *"cluster_name"* in the Terraform code when it needs to be concatenated with a **String**

```tf
"${var.cluster_name}-some-text"
```

## Main for the Module

We will cover sevaral interesting Terraform constructs for the `main.tf` module.

### Modules Locals

**Module locals** are a way to define a variable locals inside a module to do some intermediary calculation and keep the code [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). Using **module locals** these variables are not exposed as a configurable input

The syntaxis is

```tf
local.<NAME>
```

### File paths

The `user-data.sh` file is references with the `templatefile` built-in function to read this file from disk. By default, Terraform interprets the path relative to the current working directory.

The catch is that we are running `terraform apply` not from the same working directory as the **reusable module**

We use **path reference** to solve this escenario. The syntaxis is

```bash
path.<TYPE>
```

- `path.module` Returns the filesystem path of the module where the expression is defined.

- `path.root` Returns the filesystem path of the root module.

- `path.cwd` Returns the filesystem path of the current working directory. In normal use of Terraform, this is the same as `path.root`, but some advanced uses of Terraform run it from a directory other than the root module directory, causing these paths to be different.

In this case, `path.module` fulfills this need as since in this snippet

```tf
  user_data = templatefile("${path.module}/user-data.sh", {
    server_port  = var.server_port
    main_message = var.main_message
    sub_message  = var.sub_message
  })
```

## Inline Resources

It is best to separate the resources. For example, for **Security Group**

- Not a good practice is including the **ingress** and **egress** inside the resource `aws_security_group`

```tf
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
```

- Best Practice is to define `aws_security_group_rule` and from them reference the resource `aws_security_group`

```tf
resource "aws_security_group" "alb" {
  name = "${var.cluster_name}-alb"
}

resource "aws_security_group_rule" "allow_http_inbound" {
  type              = "ingress"
  security_group_id = aws_security_group.alb.id

  from_port   = local.http_port
  to_port     = local.http_port
  protocol    = local.tcp_protocol
  cidr_blocks = local.all_ips
}

resource "aws_security_group_rule" "allow_all_outbound" {
  type              = "egress"
  security_group_id = aws_security_group.alb.id

  from_port   = local.any_port
  to_port     = local.any_port
  protocol    = local.any_protocol
  cidr_blocks = local.all_ips
}
```

Other examples

- `aws_security_group` and `aws_security_group_rule`
- `aws_route_table` and `aws_route`
- `aws_network_acl` and `aws_network_acl_rule`



### Finally, the main.tf file

- `main.tf`

```tf
/* 
****************************************

Locals

****************************************
*/

locals {
  http_port    = 80
  any_port     = 0
  any_protocol = "-1"
  tcp_protocol = "tcp"
  all_ips      = ["0.0.0.0/0"]
}

/* 
****************************************

Launch Configuration

****************************************
*/

resource "aws_launch_configuration" "example" {
  image_id        = "ami-04a81a99f5ec58529"
  instance_type   = var.instance_type
  security_groups = [aws_security_group.instance.id]

  user_data = templatefile("${path.module}/user-data.sh", {
    server_port  = var.server_port
    main_message = var.main_message
    sub_message  = var.sub_message
  })

  # Required when using a launch configuration with an auto scaling group.
  lifecycle {
    create_before_destroy = true
  }
}

/* 
****************************************

Auto Scaling

****************************************
*/

resource "aws_autoscaling_group" "example" {
  launch_configuration = aws_launch_configuration.example.name
  vpc_zone_identifier  = data.aws_subnets.default.ids
  target_group_arns    = [aws_lb_target_group.asg.arn]
  health_check_type    = "ELB"

  min_size = var.min_size
  max_size = var.max_size

  tag {
    key                 = "Name"
    value               = var.cluster_name
    propagate_at_launch = true
  }
}

/* 
****************************************

Security Groups

****************************************
*/

/*
********************
SG for EC2 instances
********************
*/

resource "aws_security_group" "instance" {
  name = "${var.cluster_name}-instance"
}

resource "aws_security_group_rule" "allow_server_http_inbound" {
  type              = "ingress"
  security_group_id = aws_security_group.instance.id

  from_port   = var.server_port
  to_port     = var.server_port
  protocol    = local.tcp_protocol
  cidr_blocks = local.all_ips
}

/*
********************
SG for ALB
********************
*/

resource "aws_security_group" "alb" {
  name = "${var.cluster_name}-alb"
}

resource "aws_security_group_rule" "allow_http_inbound" {
  type              = "ingress"
  security_group_id = aws_security_group.alb.id

  from_port   = local.http_port
  to_port     = local.http_port
  protocol    = local.tcp_protocol
  cidr_blocks = local.all_ips
}

resource "aws_security_group_rule" "allow_all_outbound" {
  type              = "egress"
  security_group_id = aws_security_group.alb.id

  from_port   = local.any_port
  to_port     = local.any_port
  protocol    = local.any_protocol
  cidr_blocks = local.all_ips
}

/* 
****************************************

Load Balancer

****************************************
*/

resource "aws_lb" "example" {
  name               = var.cluster_name
  load_balancer_type = "application"
  subnets            = data.aws_subnets.default.ids
  security_groups    = [aws_security_group.alb.id]
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.example.arn
  port              = local.http_port
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

resource "aws_lb_target_group" "asg" {
  name     = var.cluster_name
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

/* 
****************************************

   Data

****************************************
*/

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

### Module Outputs

- `outputs.tf`

```tf
output "alb_dns_name" {
  value       = aws_lb.example.dns_name
  description = "The domain name of the load balancer"
}

output "asg_name" {
  value       = aws_autoscaling_group.example.name
  description = "The name of the Auto Scaling Group"
}

output "alb_security_group_id" {
  value       = aws_security_group.alb.id
  description = "The ID of the Security Group attached to the load balancer"
}
```

Syntax to access module output variables

```tf
module.<MODULE_NAME>.<OUTPUT_NAME>
```

### user-data.sh file

```bash
#!/bin/bash

cat > index.html <<EOF
<h1>Main message: ${main_message}</h1>
<p>Sub message: ${sub_message}</p>
EOF

nohup busybox httpd -f -p ${server_port} &
```

## Creating the Root Module

We now define the files that are part of the **Root Module**. This **Root Module** will call the **Reusable Module**

All these files are part of

```bash
cloud_user@553b1e446c1c:~$ tree root_module/
root_module/
├── main.tf
├── outputs.tf
└── variables.tf

0 directories, 3 files
cloud_user@553b1e446c1c:~$
```

- `main.tf`

The **Provider** goes in the **Root Module**. 

Here, we also make use of the **output variable** `alb_security_group_id` exposed in the **Reusable Module**

We can have the **Input Variable** hard coded when calling the **Resusable Module** 

```bash
provider "aws" {
  region = "us-east-1"
}

module "webserver_cluster" {
  source = "../reusable_module"

  # (parameters hidden for clarity)

  cluster_name  = var.cluster_name
  instance_type = "t2.micro"
  min_size      = 2
  max_size      = 2

  main_message  = "Hello World!"
  sub_message   = "Practice creating a Terraform Module"
}

resource "aws_security_group_rule" "allow_testing_inbound" {
  type              = "ingress"
  security_group_id = module.webserver_cluster.alb_security_group_id

  from_port   = 12345
  to_port     = 12345
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}
```

- `outputs.tf`

```bash
output "alb_dns_name" {
  value       = module.webserver_cluster.alb_dns_name
  description = "The domain name of the load balancer"
}
```

- `variables.tf`

If we don't hard code the **input variable** in the `module`, we can definte them indepentendly in the `variables.tf` file

```bash
# ---------------------------------------------------------------------------------------------------------------------
# OPTIONAL PARAMETERS
# These parameters have reasonable defaults.
# ---------------------------------------------------------------------------------------------------------------------

variable "cluster_name" {
  description = "The name to use to namespace all the resources in the cluster"
  type        = string
  default     = "webservers-stage"
}
```

## Testing everything together

Now, we will test this.

Setting the credentials to access and AWS playground

```bash
export AWS_ACCESS_KEY_ID=(your access key id)
export AWS_SECRET_ACCESS_KEY=(your secret access key)
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ export AWS_ACCESS_KEY_ID=AKI...
cloud_user@553b1e446c1c:~$ export AWS_SECRET_ACCESS_KEY=TmQ...
cloud_user@553b1e446c1c:~$
```
</details><br />

- `terraform init`

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/root_module$ terraform init
Initializing the backend...
Initializing modules...
- webserver_cluster in ../reusable_module
Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v5.62.0...
- Installed hashicorp/aws v5.62.0 (signed by HashiCorp)
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
cloud_user@553b1e446c1c:~/root_module$
```
</details><br />

- `terraform plan`

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/root_module$ terraform plan
module.webserver_cluster.data.aws_vpc.default: Reading...
module.webserver_cluster.data.aws_vpc.default: Read complete after 1s [id=vpc-0fcc29a16d49cacc0]
module.webserver_cluster.data.aws_subnets.default: Reading...
module.webserver_cluster.data.aws_subnets.default: Read complete after 0s [id=us-east-1]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_security_group_rule.allow_testing_inbound will be created
  + resource "aws_security_group_rule" "allow_testing_inbound" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 12345
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 12345
      + type                     = "ingress"
    }

  # module.webserver_cluster.aws_autoscaling_group.example will be created
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
      + max_size                         = 2
      + metrics_granularity              = "1Minute"
      + min_size                         = 2
      + name                             = (known after apply)
      + name_prefix                      = (known after apply)
      + predicted_capacity               = (known after apply)
      + protect_from_scale_in            = false
      + service_linked_role_arn          = (known after apply)
      + target_group_arns                = (known after apply)
      + vpc_zone_identifier              = [
          + "subnet-010ccd4d9fe2de73e",
          + "subnet-02342200ba08b6aec",
          + "subnet-0625e693c258e65ea",
          + "subnet-0b48041dc152a9a6f",
          + "subnet-0c76cd8bbb7097897",
          + "subnet-0ce915465bda4b943",
        ]
      + wait_for_capacity_timeout        = "10m"
      + warm_pool_size                   = (known after apply)

      + launch_template (known after apply)

      + mixed_instances_policy (known after apply)

      + tag {
          + key                 = "Name"
          + propagate_at_launch = true
          + value               = "webservers-stage"
        }

      + traffic_source (known after apply)
    }

  # module.webserver_cluster.aws_launch_configuration.example will be created
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
      + user_data                   = "7ad39f7c195a5d37eb62e490be677ab93080badf"

      + ebs_block_device (known after apply)

      + metadata_options (known after apply)

      + root_block_device (known after apply)
    }

  # module.webserver_cluster.aws_lb.example will be created
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
      + name                                                         = "webservers-stage"
      + name_prefix                                                  = (known after apply)
      + preserve_host_header                                         = false
      + security_groups                                              = (known after apply)
      + subnets                                                      = [
          + "subnet-010ccd4d9fe2de73e",
          + "subnet-02342200ba08b6aec",
          + "subnet-0625e693c258e65ea",
          + "subnet-0b48041dc152a9a6f",
          + "subnet-0c76cd8bbb7097897",
          + "subnet-0ce915465bda4b943",
        ]
      + tags_all                                                     = (known after apply)
      + vpc_id                                                       = (known after apply)
      + xff_header_processing_mode                                   = "append"
      + zone_id                                                      = (known after apply)

      + subnet_mapping (known after apply)
    }

  # module.webserver_cluster.aws_lb_listener.http will be created
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

  # module.webserver_cluster.aws_lb_listener_rule.asg will be created
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

  # module.webserver_cluster.aws_lb_target_group.asg will be created
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
      + name                               = "webservers-stage"
      + name_prefix                        = (known after apply)
      + port                               = 8080
      + preserve_client_ip                 = (known after apply)
      + protocol                           = "HTTP"
      + protocol_version                   = (known after apply)
      + proxy_protocol_v2                  = false
      + slow_start                         = 0
      + tags_all                           = (known after apply)
      + target_type                        = "instance"
      + vpc_id                             = "vpc-0fcc29a16d49cacc0"

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

  # module.webserver_cluster.aws_security_group.alb will be created
  + resource "aws_security_group" "alb" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = (known after apply)
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = "webservers-stage-alb"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags_all               = (known after apply)
      + vpc_id                 = (known after apply)
    }

  # module.webserver_cluster.aws_security_group.instance will be created
  + resource "aws_security_group" "instance" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = (known after apply)
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = "webservers-stage-instance"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags_all               = (known after apply)
      + vpc_id                 = (known after apply)
    }

  # module.webserver_cluster.aws_security_group_rule.allow_all_outbound will be created
  + resource "aws_security_group_rule" "allow_all_outbound" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 0
      + id                       = (known after apply)
      + protocol                 = "-1"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 0
      + type                     = "egress"
    }

  # module.webserver_cluster.aws_security_group_rule.allow_http_inbound will be created
  + resource "aws_security_group_rule" "allow_http_inbound" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 80
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 80
      + type                     = "ingress"
    }

  # module.webserver_cluster.aws_security_group_rule.allow_server_http_inbound will be created
  + resource "aws_security_group_rule" "allow_server_http_inbound" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 8080
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 8080
      + type                     = "ingress"
    }

Plan: 12 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
cloud_user@553b1e446c1c:~/root_module$
```
</details><br />

- `terraform apply`

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/root_module$ terraform apply
module.webserver_cluster.data.aws_vpc.default: Reading...
module.webserver_cluster.data.aws_vpc.default: Read complete after 1s [id=vpc-0fcc29a16d49cacc0]
module.webserver_cluster.data.aws_subnets.default: Reading...
module.webserver_cluster.data.aws_subnets.default: Read complete after 0s [id=us-east-1]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_security_group_rule.allow_testing_inbound will be created
  + resource "aws_security_group_rule" "allow_testing_inbound" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 12345
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 12345
      + type                     = "ingress"
    }

  # module.webserver_cluster.aws_autoscaling_group.example will be created
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
      + max_size                         = 2
      + metrics_granularity              = "1Minute"
      + min_size                         = 2
      + name                             = (known after apply)
      + name_prefix                      = (known after apply)
      + predicted_capacity               = (known after apply)
      + protect_from_scale_in            = false
      + service_linked_role_arn          = (known after apply)
      + target_group_arns                = (known after apply)
      + vpc_zone_identifier              = [
          + "subnet-010ccd4d9fe2de73e",
          + "subnet-02342200ba08b6aec",
          + "subnet-0625e693c258e65ea",
          + "subnet-0b48041dc152a9a6f",
          + "subnet-0c76cd8bbb7097897",
          + "subnet-0ce915465bda4b943",
        ]
      + wait_for_capacity_timeout        = "10m"
      + warm_pool_size                   = (known after apply)

      + launch_template (known after apply)

      + mixed_instances_policy (known after apply)

      + tag {
          + key                 = "Name"
          + propagate_at_launch = true
          + value               = "webservers-stage"
        }

      + traffic_source (known after apply)
    }

  # module.webserver_cluster.aws_launch_configuration.example will be created
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
      + user_data                   = "7ad39f7c195a5d37eb62e490be677ab93080badf"

      + ebs_block_device (known after apply)

      + metadata_options (known after apply)

      + root_block_device (known after apply)
    }

  # module.webserver_cluster.aws_lb.example will be created
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
      + name                                                         = "webservers-stage"
      + name_prefix                                                  = (known after apply)
      + preserve_host_header                                         = false
      + security_groups                                              = (known after apply)
      + subnets                                                      = [
          + "subnet-010ccd4d9fe2de73e",
          + "subnet-02342200ba08b6aec",
          + "subnet-0625e693c258e65ea",
          + "subnet-0b48041dc152a9a6f",
          + "subnet-0c76cd8bbb7097897",
          + "subnet-0ce915465bda4b943",
        ]
      + tags_all                                                     = (known after apply)
      + vpc_id                                                       = (known after apply)
      + xff_header_processing_mode                                   = "append"
      + zone_id                                                      = (known after apply)

      + subnet_mapping (known after apply)
    }

  # module.webserver_cluster.aws_lb_listener.http will be created
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

  # module.webserver_cluster.aws_lb_listener_rule.asg will be created
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

  # module.webserver_cluster.aws_lb_target_group.asg will be created
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
      + name                               = "webservers-stage"
      + name_prefix                        = (known after apply)
      + port                               = 8080
      + preserve_client_ip                 = (known after apply)
      + protocol                           = "HTTP"
      + protocol_version                   = (known after apply)
      + proxy_protocol_v2                  = false
      + slow_start                         = 0
      + tags_all                           = (known after apply)
      + target_type                        = "instance"
      + vpc_id                             = "vpc-0fcc29a16d49cacc0"

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

  # module.webserver_cluster.aws_security_group.alb will be created
  + resource "aws_security_group" "alb" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = (known after apply)
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = "webservers-stage-alb"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags_all               = (known after apply)
      + vpc_id                 = (known after apply)
    }

  # module.webserver_cluster.aws_security_group.instance will be created
  + resource "aws_security_group" "instance" {
      + arn                    = (known after apply)
      + description            = "Managed by Terraform"
      + egress                 = (known after apply)
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = "webservers-stage-instance"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags_all               = (known after apply)
      + vpc_id                 = (known after apply)
    }

  # module.webserver_cluster.aws_security_group_rule.allow_all_outbound will be created
  + resource "aws_security_group_rule" "allow_all_outbound" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 0
      + id                       = (known after apply)
      + protocol                 = "-1"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 0
      + type                     = "egress"
    }

  # module.webserver_cluster.aws_security_group_rule.allow_http_inbound will be created
  + resource "aws_security_group_rule" "allow_http_inbound" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 80
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 80
      + type                     = "ingress"
    }

  # module.webserver_cluster.aws_security_group_rule.allow_server_http_inbound will be created
  + resource "aws_security_group_rule" "allow_server_http_inbound" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 8080
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 8080
      + type                     = "ingress"
    }

Plan: 12 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

module.webserver_cluster.aws_lb_target_group.asg: Creating...
module.webserver_cluster.aws_security_group.instance: Creating...
module.webserver_cluster.aws_security_group.alb: Creating...
module.webserver_cluster.aws_lb_target_group.asg: Creation complete after 1s [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:targetgroup/webservers-stage/942881c27b077f39]
module.webserver_cluster.aws_security_group.alb: Creation complete after 2s [id=sg-0f2541b00e8f6a055]
module.webserver_cluster.aws_security_group_rule.allow_http_inbound: Creating...
module.webserver_cluster.aws_security_group_rule.allow_all_outbound: Creating...
aws_security_group_rule.allow_testing_inbound: Creating...
module.webserver_cluster.aws_security_group.instance: Creation complete after 2s [id=sg-05add330648f664bd]
module.webserver_cluster.aws_lb.example: Creating...
module.webserver_cluster.aws_launch_configuration.example: Creating...
module.webserver_cluster.aws_security_group_rule.allow_server_http_inbound: Creating...
module.webserver_cluster.aws_launch_configuration.example: Creation complete after 0s [id=terraform-20240810024418794100000001]
module.webserver_cluster.aws_autoscaling_group.example: Creating...
module.webserver_cluster.aws_security_group_rule.allow_http_inbound: Creation complete after 0s [id=sgrule-3361417116]
module.webserver_cluster.aws_security_group_rule.allow_server_http_inbound: Creation complete after 0s [id=sgrule-3207932750]
aws_security_group_rule.allow_testing_inbound: Creation complete after 1s [id=sgrule-2599150364]
module.webserver_cluster.aws_security_group_rule.allow_all_outbound: Creation complete after 1s [id=sgrule-2343032155]
module.webserver_cluster.aws_lb.example: Still creating... [10s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still creating... [10s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [20s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still creating... [20s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [30s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still creating... [30s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [40s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still creating... [40s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [50s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still creating... [50s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [1m0s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still creating... [1m0s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [1m10s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still creating... [1m10s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Creation complete after 1m17s [id=terraform-20240810024419295800000002]
module.webserver_cluster.aws_lb.example: Still creating... [1m20s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [1m30s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [1m40s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [1m50s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [2m0s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [2m10s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [2m20s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [2m30s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [2m40s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [2m50s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [3m0s elapsed]
module.webserver_cluster.aws_lb.example: Still creating... [3m10s elapsed]
module.webserver_cluster.aws_lb.example: Creation complete after 3m12s [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:loadbalancer/app/webservers-stage/d6089bf5d50863a2]
module.webserver_cluster.aws_lb_listener.http: Creating...
module.webserver_cluster.aws_lb_listener.http: Creation complete after 0s [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:listener/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367]
module.webserver_cluster.aws_lb_listener_rule.asg: Creating...
module.webserver_cluster.aws_lb_listener_rule.asg: Creation complete after 0s [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:listener-rule/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367/42cf801764cc505d]

Apply complete! Resources: 12 added, 0 changed, 0 destroyed.
cloud_user@553b1e446c1c:~/root_module$
```
</details><br />

## Verification

I forgot to add the file `outputs.tf` ...

I added it an ran `terraform output`

```bash
cloud_user@553b1e446c1c:~/root_module$ terraform output
╷
│ Warning: No outputs found
│
│ The state file either has no outputs defined, or all the defined outputs are empty. Please define an output in your configuration with the `output` keyword and run `terraform refresh` for it to become available. If
│ you are using interpolation, please verify the interpolated value is not empty. You can use the `terraform console` command to assist.
╵
cloud_user@553b1e446c1c:~/root_module$
```

I then ran `terraform refresh`

```bash
cloud_user@553b1e446c1c:~/root_module$ terraform refresh
module.webserver_cluster.data.aws_vpc.default: Reading...
module.webserver_cluster.aws_security_group.alb: Refreshing state... [id=sg-0f2541b00e8f6a055]
module.webserver_cluster.aws_security_group.instance: Refreshing state... [id=sg-05add330648f664bd]
module.webserver_cluster.aws_launch_configuration.example: Refreshing state... [id=terraform-20240810024418794100000001]
module.webserver_cluster.aws_security_group_rule.allow_server_http_inbound: Refreshing state... [id=sgrule-3207932750]
module.webserver_cluster.aws_security_group_rule.allow_all_outbound: Refreshing state... [id=sgrule-2343032155]
module.webserver_cluster.aws_security_group_rule.allow_http_inbound: Refreshing state... [id=sgrule-3361417116]
aws_security_group_rule.allow_testing_inbound: Refreshing state... [id=sgrule-2599150364]
module.webserver_cluster.data.aws_vpc.default: Read complete after 0s [id=vpc-0fcc29a16d49cacc0]
module.webserver_cluster.data.aws_subnets.default: Reading...
module.webserver_cluster.aws_lb_target_group.asg: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:targetgroup/webservers-stage/942881c27b077f39]
module.webserver_cluster.data.aws_subnets.default: Read complete after 1s [id=us-east-1]
module.webserver_cluster.aws_lb.example: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:loadbalancer/app/webservers-stage/d6089bf5d50863a2]
module.webserver_cluster.aws_autoscaling_group.example: Refreshing state... [id=terraform-20240810024419295800000002]
module.webserver_cluster.aws_lb_listener.http: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:listener/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367]
module.webserver_cluster.aws_lb_listener_rule.asg: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:listener-rule/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367/42cf801764cc505d]

Outputs:

alb_dns_name = "webservers-stage-241124624.us-east-1.elb.amazonaws.com"
cloud_user@553b1e446c1c:~/root_module$
```

Finally, `terraform output` again

```bash
cloud_user@553b1e446c1c:~/root_module$ terraform output
alb_dns_name = "webservers-stage-241124624.us-east-1.elb.amazonaws.com"
cloud_user@553b1e446c1c:~/root_module$
```

Finally, now that I know the URL for the ALB.

```bash
$ curl webservers-stage-241124624.us-east-1.elb.amazonaws.com
<h1>Main message: Hello World!</h1>
<p>Sub message: Practice creating a Terraform Module</p>
$
```

In a web browser

![]({{ site.baseurl }}/images/2024/08-09-Terraform-modules/01-Hello-World-in-Web-Browser.png)

## Removing the infrastructure

I ran `terraform destroy`

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/root_module$ terraform destroy
module.webserver_cluster.data.aws_vpc.default: Reading...
module.webserver_cluster.aws_security_group.instance: Refreshing state... [id=sg-05add330648f664bd]
module.webserver_cluster.aws_security_group.alb: Refreshing state... [id=sg-0f2541b00e8f6a055]
module.webserver_cluster.aws_security_group_rule.allow_all_outbound: Refreshing state... [id=sgrule-2343032155]
aws_security_group_rule.allow_testing_inbound: Refreshing state... [id=sgrule-2599150364]
module.webserver_cluster.aws_security_group_rule.allow_server_http_inbound: Refreshing state... [id=sgrule-3207932750]
module.webserver_cluster.aws_launch_configuration.example: Refreshing state... [id=terraform-20240810024418794100000001]
module.webserver_cluster.aws_security_group_rule.allow_http_inbound: Refreshing state... [id=sgrule-3361417116]
module.webserver_cluster.data.aws_vpc.default: Read complete after 0s [id=vpc-0fcc29a16d49cacc0]
module.webserver_cluster.data.aws_subnets.default: Reading...
module.webserver_cluster.aws_lb_target_group.asg: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:targetgroup/webservers-stage/942881c27b077f39]
module.webserver_cluster.data.aws_subnets.default: Read complete after 0s [id=us-east-1]
module.webserver_cluster.aws_lb.example: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:loadbalancer/app/webservers-stage/d6089bf5d50863a2]
module.webserver_cluster.aws_autoscaling_group.example: Refreshing state... [id=terraform-20240810024419295800000002]
module.webserver_cluster.aws_lb_listener.http: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:listener/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367]
module.webserver_cluster.aws_lb_listener_rule.asg: Refreshing state... [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:listener-rule/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367/42cf801764cc505d]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_security_group_rule.allow_testing_inbound will be destroyed
  - resource "aws_security_group_rule" "allow_testing_inbound" {
      - cidr_blocks            = [
          - "0.0.0.0/0",
        ] -> null
      - from_port              = 12345 -> null
      - id                     = "sgrule-2599150364" -> null
      - protocol               = "tcp" -> null
      - security_group_id      = "sg-0f2541b00e8f6a055" -> null
      - security_group_rule_id = "sgr-034db49bb2e520086" -> null
      - self                   = false -> null
      - to_port                = 12345 -> null
      - type                   = "ingress" -> null
    }

  # module.webserver_cluster.aws_autoscaling_group.example will be destroyed
  - resource "aws_autoscaling_group" "example" {
      - arn                              = "arn:aws:autoscaling:us-east-1:905418254045:autoScalingGroup:551d834f-bce9-4404-890d-6cc747675d47:autoScalingGroupName/terraform-20240810024419295800000002" -> null
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
      - id                               = "terraform-20240810024419295800000002" -> null
      - ignore_failed_scaling_activities = false -> null
      - launch_configuration             = "terraform-20240810024418794100000001" -> null
      - load_balancers                   = [] -> null
      - max_instance_lifetime            = 0 -> null
      - max_size                         = 2 -> null
      - metrics_granularity              = "1Minute" -> null
      - min_size                         = 2 -> null
      - name                             = "terraform-20240810024419295800000002" -> null
      - name_prefix                      = "terraform-" -> null
      - predicted_capacity               = 0 -> null
      - protect_from_scale_in            = false -> null
      - service_linked_role_arn          = "arn:aws:iam::905418254045:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling" -> null
      - suspended_processes              = [] -> null
      - target_group_arns                = [
          - "arn:aws:elasticloadbalancing:us-east-1:905418254045:targetgroup/webservers-stage/942881c27b077f39",
        ] -> null
      - termination_policies             = [] -> null
      - vpc_zone_identifier              = [
          - "subnet-010ccd4d9fe2de73e",
          - "subnet-02342200ba08b6aec",
          - "subnet-0625e693c258e65ea",
          - "subnet-0b48041dc152a9a6f",
          - "subnet-0c76cd8bbb7097897",
          - "subnet-0ce915465bda4b943",
        ] -> null
      - wait_for_capacity_timeout        = "10m" -> null
      - warm_pool_size                   = 0 -> null
        # (3 unchanged attributes hidden)

      - tag {
          - key                 = "Name" -> null
          - propagate_at_launch = true -> null
          - value               = "webservers-stage" -> null
        }

      - traffic_source {
          - identifier = "arn:aws:elasticloadbalancing:us-east-1:905418254045:targetgroup/webservers-stage/942881c27b077f39" -> null
          - type       = "elbv2" -> null
        }
    }

  # module.webserver_cluster.aws_launch_configuration.example will be destroyed
  - resource "aws_launch_configuration" "example" {
      - arn                         = "arn:aws:autoscaling:us-east-1:905418254045:launchConfiguration:31e60351-113a-49cc-a2ac-ae7a037d895f:launchConfigurationName/terraform-20240810024418794100000001" -> null
      - associate_public_ip_address = false -> null
      - ebs_optimized               = false -> null
      - enable_monitoring           = true -> null
      - id                          = "terraform-20240810024418794100000001" -> null
      - image_id                    = "ami-04a81a99f5ec58529" -> null
      - instance_type               = "t2.micro" -> null
      - name                        = "terraform-20240810024418794100000001" -> null
      - name_prefix                 = "terraform-" -> null
      - security_groups             = [
          - "sg-05add330648f664bd",
        ] -> null
      - user_data                   = "7ad39f7c195a5d37eb62e490be677ab93080badf" -> null
        # (4 unchanged attributes hidden)
    }

  # module.webserver_cluster.aws_lb.example will be destroyed
  - resource "aws_lb" "example" {
      - arn                                                          = "arn:aws:elasticloadbalancing:us-east-1:905418254045:loadbalancer/app/webservers-stage/d6089bf5d50863a2" -> null
      - arn_suffix                                                   = "app/webservers-stage/d6089bf5d50863a2" -> null
      - client_keep_alive                                            = 3600 -> null
      - desync_mitigation_mode                                       = "defensive" -> null
      - dns_name                                                     = "webservers-stage-241124624.us-east-1.elb.amazonaws.com" -> null
      - drop_invalid_header_fields                                   = false -> null
      - enable_cross_zone_load_balancing                             = true -> null
      - enable_deletion_protection                                   = false -> null
      - enable_http2                                                 = true -> null
      - enable_tls_version_and_cipher_suite_headers                  = false -> null
      - enable_waf_fail_open                                         = false -> null
      - enable_xff_client_port                                       = false -> null
      - id                                                           = "arn:aws:elasticloadbalancing:us-east-1:905418254045:loadbalancer/app/webservers-stage/d6089bf5d50863a2" -> null
      - idle_timeout                                                 = 60 -> null
      - internal                                                     = false -> null
      - ip_address_type                                              = "ipv4" -> null
      - load_balancer_type                                           = "application" -> null
      - name                                                         = "webservers-stage" -> null
      - preserve_host_header                                         = false -> null
      - security_groups                                              = [
          - "sg-0f2541b00e8f6a055",
        ] -> null
      - subnets                                                      = [
          - "subnet-010ccd4d9fe2de73e",
          - "subnet-02342200ba08b6aec",
          - "subnet-0625e693c258e65ea",
          - "subnet-0b48041dc152a9a6f",
          - "subnet-0c76cd8bbb7097897",
          - "subnet-0ce915465bda4b943",
        ] -> null
      - tags                                                         = {} -> null
      - tags_all                                                     = {} -> null
      - vpc_id                                                       = "vpc-0fcc29a16d49cacc0" -> null
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
          - subnet_id            = "subnet-010ccd4d9fe2de73e" -> null
            # (4 unchanged attributes hidden)
        }
      - subnet_mapping {
          - subnet_id            = "subnet-02342200ba08b6aec" -> null
            # (4 unchanged attributes hidden)
        }
      - subnet_mapping {
          - subnet_id            = "subnet-0625e693c258e65ea" -> null
            # (4 unchanged attributes hidden)
        }
      - subnet_mapping {
          - subnet_id            = "subnet-0b48041dc152a9a6f" -> null
            # (4 unchanged attributes hidden)
        }
      - subnet_mapping {
          - subnet_id            = "subnet-0c76cd8bbb7097897" -> null
            # (4 unchanged attributes hidden)
        }
      - subnet_mapping {
          - subnet_id            = "subnet-0ce915465bda4b943" -> null
            # (4 unchanged attributes hidden)
        }
    }

  # module.webserver_cluster.aws_lb_listener.http will be destroyed
  - resource "aws_lb_listener" "http" {
      - arn               = "arn:aws:elasticloadbalancing:us-east-1:905418254045:listener/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367" -> null
      - id                = "arn:aws:elasticloadbalancing:us-east-1:905418254045:listener/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367" -> null
      - load_balancer_arn = "arn:aws:elasticloadbalancing:us-east-1:905418254045:loadbalancer/app/webservers-stage/d6089bf5d50863a2" -> null
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

  # module.webserver_cluster.aws_lb_listener_rule.asg will be destroyed
  - resource "aws_lb_listener_rule" "asg" {
      - arn          = "arn:aws:elasticloadbalancing:us-east-1:905418254045:listener-rule/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367/42cf801764cc505d" -> null
      - id           = "arn:aws:elasticloadbalancing:us-east-1:905418254045:listener-rule/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367/42cf801764cc505d" -> null
      - listener_arn = "arn:aws:elasticloadbalancing:us-east-1:905418254045:listener/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367" -> null
      - priority     = 100 -> null
      - tags         = {} -> null
      - tags_all     = {} -> null

      - action {
          - order            = 1 -> null
          - target_group_arn = "arn:aws:elasticloadbalancing:us-east-1:905418254045:targetgroup/webservers-stage/942881c27b077f39" -> null
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

  # module.webserver_cluster.aws_lb_target_group.asg will be destroyed
  - resource "aws_lb_target_group" "asg" {
      - arn                                = "arn:aws:elasticloadbalancing:us-east-1:905418254045:targetgroup/webservers-stage/942881c27b077f39" -> null
      - arn_suffix                         = "targetgroup/webservers-stage/942881c27b077f39" -> null
      - deregistration_delay               = "300" -> null
      - id                                 = "arn:aws:elasticloadbalancing:us-east-1:905418254045:targetgroup/webservers-stage/942881c27b077f39" -> null
      - ip_address_type                    = "ipv4" -> null
      - lambda_multi_value_headers_enabled = false -> null
      - load_balancer_arns                 = [
          - "arn:aws:elasticloadbalancing:us-east-1:905418254045:loadbalancer/app/webservers-stage/d6089bf5d50863a2",
        ] -> null
      - load_balancing_algorithm_type      = "round_robin" -> null
      - load_balancing_anomaly_mitigation  = "off" -> null
      - load_balancing_cross_zone_enabled  = "use_load_balancer_configuration" -> null
      - name                               = "webservers-stage" -> null
      - port                               = 8080 -> null
      - protocol                           = "HTTP" -> null
      - protocol_version                   = "HTTP1" -> null
      - proxy_protocol_v2                  = false -> null
      - slow_start                         = 0 -> null
      - tags                               = {} -> null
      - tags_all                           = {} -> null
      - target_type                        = "instance" -> null
      - vpc_id                             = "vpc-0fcc29a16d49cacc0" -> null
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

  # module.webserver_cluster.aws_security_group.alb will be destroyed
  - resource "aws_security_group" "alb" {
      - arn                    = "arn:aws:ec2:us-east-1:905418254045:security-group/sg-0f2541b00e8f6a055" -> null
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
      - id                     = "sg-0f2541b00e8f6a055" -> null
      - ingress                = [
          - {
              - cidr_blocks      = [
                  - "0.0.0.0/0",
                ]
              - from_port        = 12345
              - ipv6_cidr_blocks = []
              - prefix_list_ids  = []
              - protocol         = "tcp"
              - security_groups  = []
              - self             = false
              - to_port          = 12345
                # (1 unchanged attribute hidden)
            },
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
      - name                   = "webservers-stage-alb" -> null
      - owner_id               = "905418254045" -> null
      - revoke_rules_on_delete = false -> null
      - tags                   = {} -> null
      - tags_all               = {} -> null
      - vpc_id                 = "vpc-0fcc29a16d49cacc0" -> null
        # (1 unchanged attribute hidden)
    }

  # module.webserver_cluster.aws_security_group.instance will be destroyed
  - resource "aws_security_group" "instance" {
      - arn                    = "arn:aws:ec2:us-east-1:905418254045:security-group/sg-05add330648f664bd" -> null
      - description            = "Managed by Terraform" -> null
      - egress                 = [] -> null
      - id                     = "sg-05add330648f664bd" -> null
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
      - name                   = "webservers-stage-instance" -> null
      - owner_id               = "905418254045" -> null
      - revoke_rules_on_delete = false -> null
      - tags                   = {} -> null
      - tags_all               = {} -> null
      - vpc_id                 = "vpc-0fcc29a16d49cacc0" -> null
        # (1 unchanged attribute hidden)
    }

  # module.webserver_cluster.aws_security_group_rule.allow_all_outbound will be destroyed
  - resource "aws_security_group_rule" "allow_all_outbound" {
      - cidr_blocks            = [
          - "0.0.0.0/0",
        ] -> null
      - from_port              = 0 -> null
      - id                     = "sgrule-2343032155" -> null
      - protocol               = "-1" -> null
      - security_group_id      = "sg-0f2541b00e8f6a055" -> null
      - security_group_rule_id = "sgr-06c5c60df0e4aeb13" -> null
      - self                   = false -> null
      - to_port                = 0 -> null
      - type                   = "egress" -> null
    }

  # module.webserver_cluster.aws_security_group_rule.allow_http_inbound will be destroyed
  - resource "aws_security_group_rule" "allow_http_inbound" {
      - cidr_blocks            = [
          - "0.0.0.0/0",
        ] -> null
      - from_port              = 80 -> null
      - id                     = "sgrule-3361417116" -> null
      - protocol               = "tcp" -> null
      - security_group_id      = "sg-0f2541b00e8f6a055" -> null
      - security_group_rule_id = "sgr-0563000d81f3b4ec3" -> null
      - self                   = false -> null
      - to_port                = 80 -> null
      - type                   = "ingress" -> null
    }

  # module.webserver_cluster.aws_security_group_rule.allow_server_http_inbound will be destroyed
  - resource "aws_security_group_rule" "allow_server_http_inbound" {
      - cidr_blocks            = [
          - "0.0.0.0/0",
        ] -> null
      - from_port              = 8080 -> null
      - id                     = "sgrule-3207932750" -> null
      - protocol               = "tcp" -> null
      - security_group_id      = "sg-05add330648f664bd" -> null
      - security_group_rule_id = "sgr-0b032e2a8c13bc388" -> null
      - self                   = false -> null
      - to_port                = 8080 -> null
      - type                   = "ingress" -> null
    }

Plan: 0 to add, 0 to change, 12 to destroy.

Changes to Outputs:
  - alb_dns_name = "webservers-stage-241124624.us-east-1.elb.amazonaws.com" -> null

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

module.webserver_cluster.aws_security_group_rule.allow_all_outbound: Destroying... [id=sgrule-2343032155]
module.webserver_cluster.aws_security_group_rule.allow_http_inbound: Destroying... [id=sgrule-3361417116]
aws_security_group_rule.allow_testing_inbound: Destroying... [id=sgrule-2599150364]
module.webserver_cluster.aws_autoscaling_group.example: Destroying... [id=terraform-20240810024419295800000002]
module.webserver_cluster.aws_lb_listener_rule.asg: Destroying... [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:listener-rule/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367/42cf801764cc505d]
module.webserver_cluster.aws_security_group_rule.allow_server_http_inbound: Destroying... [id=sgrule-3207932750]
module.webserver_cluster.aws_lb_listener_rule.asg: Destruction complete after 0s
module.webserver_cluster.aws_lb_listener.http: Destroying... [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:listener/app/webservers-stage/d6089bf5d50863a2/e8c957c241253367]
module.webserver_cluster.aws_lb_listener.http: Destruction complete after 0s
module.webserver_cluster.aws_lb.example: Destroying... [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:loadbalancer/app/webservers-stage/d6089bf5d50863a2]
aws_security_group_rule.allow_testing_inbound: Destruction complete after 0s
module.webserver_cluster.aws_security_group_rule.allow_server_http_inbound: Destruction complete after 0s
module.webserver_cluster.aws_security_group_rule.allow_http_inbound: Destruction complete after 0s
module.webserver_cluster.aws_security_group_rule.allow_all_outbound: Destruction complete after 1s
module.webserver_cluster.aws_lb.example: Destruction complete after 3s
module.webserver_cluster.aws_security_group.alb: Destroying... [id=sg-0f2541b00e8f6a055]
module.webserver_cluster.aws_autoscaling_group.example: Still destroying... [id=terraform-20240810024419295800000002, 10s elapsed]
module.webserver_cluster.aws_security_group.alb: Still destroying... [id=sg-0f2541b00e8f6a055, 10s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still destroying... [id=terraform-20240810024419295800000002, 20s elapsed]
module.webserver_cluster.aws_security_group.alb: Still destroying... [id=sg-0f2541b00e8f6a055, 20s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still destroying... [id=terraform-20240810024419295800000002, 30s elapsed]
module.webserver_cluster.aws_security_group.alb: Still destroying... [id=sg-0f2541b00e8f6a055, 30s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still destroying... [id=terraform-20240810024419295800000002, 40s elapsed]
module.webserver_cluster.aws_security_group.alb: Still destroying... [id=sg-0f2541b00e8f6a055, 40s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still destroying... [id=terraform-20240810024419295800000002, 50s elapsed]
module.webserver_cluster.aws_security_group.alb: Destruction complete after 48s
module.webserver_cluster.aws_autoscaling_group.example: Still destroying... [id=terraform-20240810024419295800000002, 1m0s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still destroying... [id=terraform-20240810024419295800000002, 1m10s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still destroying... [id=terraform-20240810024419295800000002, 1m20s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Still destroying... [id=terraform-20240810024419295800000002, 1m30s elapsed]
module.webserver_cluster.aws_autoscaling_group.example: Destruction complete after 1m32s
module.webserver_cluster.aws_launch_configuration.example: Destroying... [id=terraform-20240810024418794100000001]
module.webserver_cluster.aws_lb_target_group.asg: Destroying... [id=arn:aws:elasticloadbalancing:us-east-1:905418254045:targetgroup/webservers-stage/942881c27b077f39]
module.webserver_cluster.aws_launch_configuration.example: Destruction complete after 0s
module.webserver_cluster.aws_security_group.instance: Destroying... [id=sg-05add330648f664bd]
module.webserver_cluster.aws_lb_target_group.asg: Destruction complete after 0s
module.webserver_cluster.aws_security_group.instance: Destruction complete after 1s

Destroy complete! Resources: 12 destroyed.
cloud_user@553b1e446c1c:~/root_module$
```
</details><br />

## References

- [Comments in Terraform](https://developer.hashicorp.com/terraform/language/syntax/configuration#comments)
- [GitHub brikis98/terraform-up-and-running-code](https://github.com/brikis98/terraform-up-and-running-code/tree/3rd-edition)