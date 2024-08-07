---
title: Installing Terraform on Ubuntu 24.04
date: 2024-07-21 18:10:00 -0700
categories: [TERRAFORM, INSTALL]
tags: [terraform]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/terraform_logo.png)

## System

This guide is based on `Ubuntu 24.04 LTS`

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ cat /etc/*release*
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=24.04
DISTRIB_CODENAME=noble
DISTRIB_DESCRIPTION="Ubuntu 24.04 LTS"
PRETTY_NAME="Ubuntu 24.04 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
ubuntu@ip-XXX-XX-XX-XXX:~$ 
```

## Step 1. Update & Install dependencies

```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
Hit:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble InRelease
Get:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Get:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]
Get:4 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:5 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble/universe amd64 Packages [15.0 MB]
Get:6 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble/universe Translation-en [5982 kB]
Get:7 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [229 kB]
Get:8 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble/universe amd64 Components [3871 kB]  
Get:9 http://security.ubuntu.com/ubuntu noble-security/main Translation-en [56.2 kB]            
Get:10 http://security.ubuntu.com/ubuntu noble-security/main amd64 c-n-f Metadata [2680 B]
Get:11 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [239 kB]
Get:12 http://security.ubuntu.com/ubuntu noble-security/universe Translation-en [104 kB]           
Get:13 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [8632 B]                
Get:14 http://security.ubuntu.com/ubuntu noble-security/universe amd64 c-n-f Metadata [4564 B]            
Get:15 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [176 kB]                
Get:16 http://security.ubuntu.com/ubuntu noble-security/restricted Translation-en [34.4 kB]               
Get:17 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 c-n-f Metadata [420 B]             
Get:18 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 Packages [10.6 kB]                 
Get:19 http://security.ubuntu.com/ubuntu noble-security/multiverse Translation-en [2808 B]                  
Get:20 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 Components [208 B]                 
Get:21 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 c-n-f Metadata [344 B]             
Get:22 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble/universe amd64 c-n-f Metadata [301 kB]
Get:23 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble/multiverse amd64 Packages [269 kB]
Get:24 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble/multiverse Translation-en [118 kB]
Get:25 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble/multiverse amd64 Components [35.0 kB]
Get:26 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble/multiverse amd64 c-n-f Metadata [8328 B]
Get:27 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [266 kB]
Get:28 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/main Translation-en [70.7 kB]
Get:29 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/main amd64 c-n-f Metadata [4076 B]
Get:30 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [301 kB]
Get:31 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/universe Translation-en [127 kB]
Get:32 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Components [45.0 kB]
Get:33 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/universe amd64 c-n-f Metadata [7208 B]
Get:34 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Packages [176 kB]
Get:35 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/restricted Translation-en [34.4 kB]
Get:36 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 c-n-f Metadata [416 B]
Get:37 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Packages [14.1 kB]
Get:38 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/multiverse Translation-en [3608 B]
Get:39 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Components [212 B]
Get:40 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 c-n-f Metadata [532 B]
Get:41 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-backports/main amd64 Components [208 B]
Get:42 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-backports/main amd64 c-n-f Metadata [112 B]
Get:43 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-backports/universe amd64 Packages [11.4 kB]
Get:44 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-backports/universe Translation-en [10.5 kB]
Get:45 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-backports/universe amd64 Components [17.6 kB]
Get:46 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-backports/universe amd64 c-n-f Metadata [988 B]
Get:47 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-backports/restricted amd64 Components [216 B]
Get:48 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-backports/restricted amd64 c-n-f Metadata [116 B]
Get:49 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-backports/multiverse amd64 Components [212 B]
Get:50 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-backports/multiverse amd64 c-n-f Metadata [116 B]
Fetched 28.0 MB in 6s (4965 kB/s)               
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
gnupg is already the newest version (2.4.4-2ubuntu17).
gnupg set to manually installed.
software-properties-common is already the newest version (0.99.48).
software-properties-common set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 22 not upgraded.
ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

## Step 2. Install the HashiCorp GPG key

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
--2024-07-22 01:16:34--  https://apt.releases.hashicorp.com/gpg
Resolving apt.releases.hashicorp.com (apt.releases.hashicorp.com)... 99.84.108.74, 99.84.108.3, 99.84.108.36, ...
Connecting to apt.releases.hashicorp.com (apt.releases.hashicorp.com)|99.84.108.74|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3980 (3.9K) [binary/octet-stream]
Saving to: ‘STDOUT’

-                                                100%[=======================================================================================================>]   3.89K  --.-KB/s    in 0s      

2024-07-22 01:16:34 (642 MB/s) - written to stdout [3980/3980]

ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

## Step 3. Verify the key's fingerprint.

```bash
gpg --no-default-keyring \
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
--fingerprint
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ gpg --no-default-keyring \
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
--fingerprint
gpg: directory '/home/ubuntu/.gnupg' created
gpg: /home/ubuntu/.gnupg/trustdb.gpg: trustdb created
/usr/share/keyrings/hashicorp-archive-keyring.gpg
-------------------------------------------------
pub   rsa4096 2023-01-10 [SC] [expires: 2028-01-09]
      798A EC65 4E5C 1542 8C8E  42EE AA16 FCBC A621 E701
uid           [ unknown] HashiCorp Security (HashiCorp Package Signing) <security+packaging@hashicorp.com>
sub   rsa4096 2023-01-10 [S] [expires: 2028-01-09]

ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

**NOTE:** The `gpg` command will report the key fingerprint:

```bash
/usr/share/keyrings/hashicorp-archive-keyring.gpg
-------------------------------------------------
pub   rsa4096 XXXX-XX-XX [SC]
AAAA AAAA AAAA AAAA
uid           [ unknown] HashiCorp Security (HashiCorp Package Signing) <security+packaging@hashicorp.com>
sub   rsa4096 XXXX-XX-XX [E]
```

## Step 4. Add the official HashiCorp repository

```bash
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list
deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com noble main
ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

## Step 5. Download the package information from HashiCorp & Install Terraform

```bash
sudo apt update
sudo apt-get install terraform
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo apt update
sudo apt-get install terraform
Hit:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-updates InRelease                                                        
Hit:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu noble-backports InRelease                                                      
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease                                                                    
Get:5 https://apt.releases.hashicorp.com noble InRelease [12.9 kB]                                                                  
Get:6 https://apt.releases.hashicorp.com noble/main amd64 Packages [142 kB]
Fetched 155 kB in 1s (180 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
22 packages can be upgraded. Run 'apt list --upgradable' to see them.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  terraform
0 upgraded, 1 newly installed, 0 to remove and 22 not upgraded.
Need to get 28.0 MB of archives.
After this operation, 89.0 MB of additional disk space will be used.
Get:1 https://apt.releases.hashicorp.com noble/main amd64 terraform amd64 1.9.2-1 [28.0 MB]
Fetched 28.0 MB in 1s (55.6 MB/s)  
Selecting previously unselected package terraform.
(Reading database ... 67739 files and directories currently installed.)
Preparing to unpack .../terraform_1.9.2-1_amd64.deb ...
Unpacking terraform (1.9.2-1) ...
Setting up terraform (1.9.2-1) ...
Scanning processes...                                                                                                                                                                            
Scanning linux images...                                                                                                                                                                         

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

## Verify the installation

```bash
terraform -help
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ terraform -help
Usage: terraform [global options] <subcommand> [args]

The available commands for execution are listed below.
The primary workflow commands are given first, followed by
less common or more advanced commands.

Main commands:
  init          Prepare your working directory for other commands
  validate      Check whether the configuration is valid
  plan          Show changes required by the current configuration
  apply         Create or update infrastructure
  destroy       Destroy previously-created infrastructure

All other commands:
  console       Try Terraform expressions at an interactive command prompt
  fmt           Reformat your configuration in the standard style
  force-unlock  Release a stuck lock on the current workspace
  get           Install or upgrade remote Terraform modules
  graph         Generate a Graphviz graph of the steps in an operation
  import        Associate existing infrastructure with a Terraform resource
  login         Obtain and save credentials for a remote host
  logout        Remove locally-stored credentials for a remote host
  metadata      Metadata related commands
  output        Show output values from your root module
  providers     Show the providers required for this configuration
  refresh       Update the state to match remote systems
  show          Show the current state or a saved plan
  state         Advanced state management
  taint         Mark a resource instance as not fully functional
  test          Execute integration tests for Terraform modules
  untaint       Remove the 'tainted' state from a resource instance
  version       Show the current Terraform version
  workspace     Workspace management

Global options (use these before the subcommand, if any):
  -chdir=DIR    Switch to a different working directory before executing the
                given subcommand.
  -help         Show this help output, or the help for a specified subcommand.
  -version      An alias for the "version" subcommand.
ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

Add any subcommand to `terraform -help` to learn more about what it does and available options.

```bash
terraform -help plan
```

## Enable tab completion

In my case, I have `bash`.

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ echo $SHELL
/bin/bash
```

The commands are:

```bash
touch ~/.bashrc
terraform -install-autocomplete
```

## Start an NGINX container using Terraform

This is a "Hello World" kind of example.

**NOTE:** We need to have Docker Engine installed. Just follow the guide: [Install Docker Engine on Ubuntu]({% post_url 2024-07-16-Docker-installing-ce %})

```bash
mkdir learn-terraform-docker-container
cd learn-terraform-docker-container
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ pwd
/home/ubuntu

ubuntu@ip-XXX-XX-XX-XXX:~$ mkdir learn-terraform-docker-container
cd learn-terraform-docker-container

ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ pwd
/home/ubuntu/learn-terraform-docker-container
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ 
```
</details><br />

Create a file named `main.tf` with the following content:

```
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "tutorial"

  ports {
    internal = 80
    external = 8000
  }
}
```

Initialize the project, which downloads a plugin called a provider that lets Terraform interact with Docker.

```bash
terraform init
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ vim main.tf
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ 
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ terraform init
Initializing the backend...
Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 3.0.1"...
- Installing kreuzwerker/docker v3.0.2...
- Installed kreuzwerker/docker v3.0.2 (self-signed, key ID BD080C4571C6104C)
Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html
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
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ 
```
</details><br />

Provision the NGINX server container with `apply`. When Terraform asks you to confirm type `yes` and press `ENTER`.

```bash
terraform apply
```

**NOTE:** Since my Docker installation is running from `root` user, I had to use `sudo` for the `terraform apply` command.

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ terraform apply
╷
│ Error: Error pinging Docker server: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/_ping": dial unix /var/run/docker.sock: connect: permission denied
│ 
│   with provider["registry.terraform.io/kreuzwerker/docker"],
│   on main.tf line 10, in provider "docker":
│   10: provider "docker" {}
│ 
╵
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ sudo terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach                                      = false
      + bridge                                      = (known after apply)
      + command                                     = (known after apply)
      + container_logs                              = (known after apply)
      + container_read_refresh_timeout_milliseconds = 15000
      + entrypoint                                  = (known after apply)
      + env                                         = (known after apply)
      + exit_code                                   = (known after apply)
      + hostname                                    = (known after apply)
      + id                                          = (known after apply)
      + image                                       = (known after apply)
      + init                                        = (known after apply)
      + ipc_mode                                    = (known after apply)
      + log_driver                                  = (known after apply)
      + logs                                        = false
      + must_run                                    = true
      + name                                        = "tutorial"
      + network_data                                = (known after apply)
      + read_only                                   = false
      + remove_volumes                              = true
      + restart                                     = "no"
      + rm                                          = false
      + runtime                                     = (known after apply)
      + security_opts                               = (known after apply)
      + shm_size                                    = (known after apply)
      + start                                       = true
      + stdin_open                                  = false
      + stop_signal                                 = (known after apply)
      + stop_timeout                                = (known after apply)
      + tty                                         = false
      + wait                                        = false
      + wait_timeout                                = 60

      + healthcheck (known after apply)

      + labels (known after apply)

      + ports {
          + external = 8000
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id           = (known after apply)
      + image_id     = (known after apply)
      + keep_locally = false
      + name         = "nginx"
      + repo_digest  = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 5s [id=sha256:fffffc90d343cbcb01a5032edac86db5998c536cd0a366514121a45c6723765cnginx]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 1s [id=89ae46e20c71a44361400681e569fe767289faa63a5eff60ca8147b657546f20]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ 
```
</details><br />

Verify a Container based on Image NGINX is running in Docker locally

```bash
docker ps
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
89ae46e20c71   fffffc90d343   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8000->80/tcp   tutorial
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ 
```
</details><br />

```bash
docker inspect
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ sudo docker inspect tutorial
[
    {
        "Id": "89ae46e20c71a44361400681e569fe767289faa63a5eff60ca8147b657546f20",
        "Created": "2024-07-22T01:37:57.067259088Z",
        "Path": "/docker-entrypoint.sh",
        "Args": [
            "nginx",
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 3953,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2024-07-22T01:37:57.413000203Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:fffffc90d343cbcb01a5032edac86db5998c536cd0a366514121a45c6723765c",
        "ResolvConfPath": "/var/lib/docker/containers/89ae46e20c71a44361400681e569fe767289faa63a5eff60ca8147b657546f20/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/89ae46e20c71a44361400681e569fe767289faa63a5eff60ca8147b657546f20/hostname",
        "HostsPath": "/var/lib/docker/containers/89ae46e20c71a44361400681e569fe767289faa63a5eff60ca8147b657546f20/hosts",
        "LogPath": "/var/lib/docker/containers/89ae46e20c71a44361400681e569fe767289faa63a5eff60ca8147b657546f20/89ae46e20c71a44361400681e569fe767289faa63a5eff60ca8147b657546f20-json.log",
        "Name": "/tutorial",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "bridge",
            "PortBindings": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8000"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "ConsoleSize": [
                0,
                0
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": null,
            "DnsOptions": null,
            "DnsSearch": null,
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": null,
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware",
                "/sys/devices/virtual/powercap"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ],
            "Init": false
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/ecbc4a856a30367e15e6e45506eb8ac38cb377e9d14b3576b25d769e327b3d2d-init/diff:/var/lib/docker/overlay2/2f45cf88cf0b9d7d4a740a4b7f8d262ed9464a91c736e50eacb2884cb6d75706/diff:/var/lib/docker/overlay2/2d4785ec2e3fb0a39984c225d5639b31c09f57bc31562f8bfbef004160dd8488/diff:/var/lib/docker/overlay2/71ee67a8af3ea5f55706d90a6a3e6a512a25b6c52cd072ed9ac6584576d8beef/diff:/var/lib/docker/overlay2/0480a24d8c83b6c6a64ba1d73b54f446b2e0a284e91b9cfe154c5ba487491bb0/diff:/var/lib/docker/overlay2/5e1ac7d1857dab983a01d4e01b500a782e83e45c8cec7943540e102906ea3684/diff:/var/lib/docker/overlay2/24d07be7fce3aee42342c9fa4ce006d3df85f0b41cdd52b288bafa38d5f73570/diff:/var/lib/docker/overlay2/bdb61ea352cd4c179a9c6dd59329fc3400d3878cbc27200d8bddd6aa5790c16f/diff",
                "MergedDir": "/var/lib/docker/overlay2/ecbc4a856a30367e15e6e45506eb8ac38cb377e9d14b3576b25d769e327b3d2d/merged",
                "UpperDir": "/var/lib/docker/overlay2/ecbc4a856a30367e15e6e45506eb8ac38cb377e9d14b3576b25d769e327b3d2d/diff",
                "WorkDir": "/var/lib/docker/overlay2/ecbc4a856a30367e15e6e45506eb8ac38cb377e9d14b3576b25d769e327b3d2d/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "89ae46e20c71",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.27.0",
                "NJS_VERSION=0.8.4",
                "NJS_RELEASE=2~bookworm",
                "PKG_RELEASE=2~bookworm"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "sha256:fffffc90d343cbcb01a5032edac86db5998c536cd0a366514121a45c6723765c",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGQUIT"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "b4a0ebf90a516770a76588330e62cc484ea2b1c083e0c132e5f1a08783d99e0b",
            "SandboxKey": "/var/run/docker/netns/b4a0ebf90a51",
            "Ports": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8000"
                    }
                ]
            },
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "81b219cd34470441cbbec694ee2f8ec18a3ea4926e479b4c72ecd214ba786010",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null,
                    "NetworkID": "24f96888c2941f847bdbfa2b79c76f5e8406d61d3854346246fb86c3eddeecf5",
                    "EndpointID": "81b219cd34470441cbbec694ee2f8ec18a3ea4926e479b4c72ecd214ba786010",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": null
                }
            }
        }
    }
]
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ 
```
</details><br />

Opening the port `8000` in the web browser.

![]({{ site.baseurl }}/images/2024/07-21-Terraform-install/01-NGINX-welcome-screen.png)

To stop the container using terraform

```bash
terraform destroy
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ sudo terraform destroy
docker_image.nginx: Refreshing state... [id=sha256:fffffc90d343cbcb01a5032edac86db5998c536cd0a366514121a45c6723765cnginx]
docker_container.nginx: Refreshing state... [id=89ae46e20c71a44361400681e569fe767289faa63a5eff60ca8147b657546f20]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
      - attach                                      = false -> null
      - command                                     = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> null
      - container_read_refresh_timeout_milliseconds = 15000 -> null
      - cpu_shares                                  = 0 -> null
      - dns                                         = [] -> null
      - dns_opts                                    = [] -> null
      - dns_search                                  = [] -> null
      - entrypoint                                  = [
          - "/docker-entrypoint.sh",
        ] -> null
      - env                                         = [] -> null
      - group_add                                   = [] -> null
      - hostname                                    = "89ae46e20c71" -> null
      - id                                          = "89ae46e20c71a44361400681e569fe767289faa63a5eff60ca8147b657546f20" -> null
      - image                                       = "sha256:fffffc90d343cbcb01a5032edac86db5998c536cd0a366514121a45c6723765c" -> null
      - init                                        = false -> null
      - ipc_mode                                    = "private" -> null
      - log_driver                                  = "json-file" -> null
      - log_opts                                    = {} -> null
      - logs                                        = false -> null
      - max_retry_count                             = 0 -> null
      - memory                                      = 0 -> null
      - memory_swap                                 = 0 -> null
      - must_run                                    = true -> null
      - name                                        = "tutorial" -> null
      - network_data                                = [
          - {
              - gateway                   = "172.17.0.1"
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.17.0.2"
              - ip_prefix_length          = 16
              - mac_address               = "02:42:ac:11:00:02"
              - network_name              = "bridge"
                # (2 unchanged attributes hidden)
            },
        ] -> null
      - network_mode                                = "bridge" -> null
      - privileged                                  = false -> null
      - publish_all_ports                           = false -> null
      - read_only                                   = false -> null
      - remove_volumes                              = true -> null
      - restart                                     = "no" -> null
      - rm                                          = false -> null
      - runtime                                     = "runc" -> null
      - security_opts                               = [] -> null
      - shm_size                                    = 64 -> null
      - start                                       = true -> null
      - stdin_open                                  = false -> null
      - stop_signal                                 = "SIGQUIT" -> null
      - stop_timeout                                = 0 -> null
      - storage_opts                                = {} -> null
      - sysctls                                     = {} -> null
      - tmpfs                                       = {} -> null
      - tty                                         = false -> null
      - wait                                        = false -> null
      - wait_timeout                                = 60 -> null
        # (7 unchanged attributes hidden)

      - ports {
          - external = 8000 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id           = "sha256:fffffc90d343cbcb01a5032edac86db5998c536cd0a366514121a45c6723765cnginx" -> null
      - image_id     = "sha256:fffffc90d343cbcb01a5032edac86db5998c536cd0a366514121a45c6723765c" -> null
      - keep_locally = false -> null
      - name         = "nginx" -> null
      - repo_digest  = "nginx@sha256:67682bda769fae1ccf5183192b8daf37b64cae99c6c3302650f6f8bf5f0f95df" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

docker_container.nginx: Destroying... [id=89ae46e20c71a44361400681e569fe767289faa63a5eff60ca8147b657546f20]
docker_container.nginx: Destruction complete after 0s
docker_image.nginx: Destroying... [id=sha256:fffffc90d343cbcb01a5032edac86db5998c536cd0a366514121a45c6723765cnginx]
docker_image.nginx: Destruction complete after 1s

Destroy complete! Resources: 2 destroyed.
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ 
```
</details><br />

The NGINX container is no longer running

```bash
docker ps
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
ubuntu@ip-XXX-XX-XX-XXX:~/learn-terraform-docker-container$ 
```
</details><br />

## References

- [Install Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
