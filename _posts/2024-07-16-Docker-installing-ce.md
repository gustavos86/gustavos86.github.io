---
title: Install Docker Engine on Ubuntu
date: 2024-07-16 08:00:00 -0700
categories: [DOCKER, INSTALL]
tags: [docker]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

## TLDR;

Just copy and paste this to have Docker installed

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```

## Linux version on the Docker Host

Here is the Linux version of the Host where I am installing Docker

```bash
cat /etc/*release*
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-22-38:~$ cat /etc/*release*
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
ubuntu@ip-172-31-22-38:~$
```
</details><br />

## Uninstall old versions (optional)

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-22-38:~$ for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package 'docker.io' is not installed, so not removed
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package docker-doc
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package docker-compose
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package docker-compose-v2
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package podman-docker
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package 'containerd' is not installed, so not removed
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package 'runc' is not installed, so not removed
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
ubuntu@ip-172-31-22-38:~$ 
```
</details><br />

## Install using the convenience script

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-22-38:~$ curl -fsSL https://get.docker.com -o get-docker.sh
ubuntu@ip-172-31-22-38:~$ 
```
</details><br />

This is an **optional** dry-run to see what commands will be executed.

```bash
sudo sh ./get-docker.sh --dry-run
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-22-38:~$ sudo sh ./get-docker.sh --dry-run
# Executing docker install script, commit: 6d9743e9656cc56f699a64800b098d5ea5a60020
apt-get update -qq >/dev/null
DEBIAN_FRONTEND=noninteractive apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
install -m 0755 -d /etc/apt/keyrings
curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" > /etc/apt/sources.list.d/docker.list
apt-get update -qq >/dev/null
DEBIAN_FRONTEND=noninteractive apt-get install -y -qq docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-ce-rootless-extras docker-buildx-plugin >/dev/null
ubuntu@ip-172-31-22-38:~$ 
```
</details><br />

This script will install Docker.

```bash
sudo sh ./get-docker.sh
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-22-38:~$ sudo sh ./get-docker.sh
# Executing docker install script, commit: 6d9743e9656cc56f699a64800b098d5ea5a60020
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
Scanning processes...                                                                                                                                                                                                                                                               
Scanning linux images...                                                                                                                                                                                                                                                            
+ sh -c install -m 0755 -d /etc/apt/keyrings
+ sh -c curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" -o /etc/apt/keyrings/docker.asc
+ sh -c chmod a+r /etc/apt/keyrings/docker.asc
+ sh -c echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-ce-rootless-extras docker-buildx-plugin >/dev/null
Scanning processes...                                                                                                                                                                                                                                                               
Scanning linux images...                                                                                                                                                                                                                                                            
+ sh -c docker version
Client: Docker Engine - Community
 Version:           27.0.3
 API version:       1.46
 Go version:        go1.21.11
 Git commit:        7d4bcd8
 Built:             Sat Jun 29 00:02:23 2024
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          27.0.3
  API version:      1.46 (minimum version 1.24)
  Go version:       go1.21.11
  Git commit:       662f78c
  Built:            Sat Jun 29 00:02:23 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.18
  GitCommit:        ae71819c4f5e67bb4d5ae76a6b735f29cc25774e
 runc:
  Version:          1.7.18
  GitCommit:        v1.1.13-0-g58aa920
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

================================================================================

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

    dockerd-rootless-setuptool.sh install

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.


To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/

================================================================================

ubuntu@ip-172-31-22-38:~$
```
</details><br />

## Verify Docker CE was installed

```bash
docker version
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-22-38:~$ sudo docker version
Client: Docker Engine - Community
 Version:           27.0.3
 API version:       1.46
 Go version:        go1.21.11
 Git commit:        7d4bcd8
 Built:             Sat Jun 29 00:02:23 2024
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          27.0.3
  API version:      1.46 (minimum version 1.24)
  Go version:       go1.21.11
  Git commit:       662f78c
  Built:            Sat Jun 29 00:02:23 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.18
  GitCommit:        ae71819c4f5e67bb4d5ae76a6b735f29cc25774e
 runc:
  Version:          1.7.18
  GitCommit:        v1.1.13-0-g58aa920
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
ubuntu@ip-172-31-22-38:~$
```
</details><br />

```bash
docker run hello-world
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-22-38:~$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete 
Digest: sha256:1408fec50309afee38f3535383f5b09419e6dc0925bc69891e79d84cc4cdcec6
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

ubuntu@ip-172-31-22-38:~$
```
</details><br />

## Manage Docker as a non-root user

It is highly possible the Linux group `docker` was added while installing Docker. To verify this, run this command

```bash
compgen -g | egrep docker
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ compgen -g | egrep docker
docker
cloud_user@553b1e446c1c:~$
```
</details><br />

Now just add the user you want to have privileges on `docker` commands to this group

```bash
sudo usermod -aG docker <USERNAME>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo usermod -aG docker cloud_user
cloud_user@553b1e446c1c:~$
```
</details><br />

**IMPORTANT:** If you are currently logged on as that user, you need to **log out and log back in**

Once done, that user will have privileges running `docker` commands without the need of prepending `sudo`

```bash
cloud_user@553b1e446c1c:~$ docker ps -a
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS                      PORTS     NAMES
06a46594a8a7   frrouting/frr     "/sbin/tini -- /usr/…"   14 minutes ago   Up 14 minutes                         router1
45f7f336ab64   frr_local_build   "/bin/bash"              15 minutes ago   Exited (0) 15 minutes ago             router2
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ docker images
REPOSITORY                                                     TAG       IMAGE ID       CREATED         SIZE
frr_local_build_2                                              latest    17bfebdb57fc   3 hours ago     372MB
frr_local_build                                                latest    e4c926785418   3 hours ago     346MB
alpine                                                         latest    324bc02ae123   11 days ago     7.8MB
cloud_user-worker                                              latest    7230f713aad7   2 weeks ago     194MB
cloud_user-result                                              latest    6dbbfcb39ad0   2 weeks ago     219MB
654654406075.dkr.ecr.us-east-1.amazonaws.com/cloud_user-vote   latest    3cd62f1d4c2c   2 weeks ago     153MB
cloud_user-vote                                                latest    3cd62f1d4c2c   2 weeks ago     153MB
localhost:5000/cloud_user-vote                                 latest    3cd62f1d4c2c   2 weeks ago     153MB
hashicorp/terraform                                            latest    af62ce8c3aed   3 weeks ago     115MB
mysql                                                          latest    5cde95de907d   4 weeks ago     586MB
ubuntu                                                         latest    35a88802559d   8 weeks ago     78.1MB
redis                                                          latest    9c893be668ac   2 months ago    116MB
registry                                                       2         6a3edb1d5eb6   10 months ago   25.4MB
frrouting/frr                                                  latest    d19bacb84eae   21 months ago   151MB
postgres                                                       9.4       ed5a45034282   4 years ago     251MB
cloud_user@553b1e446c1c:~$
```

## References
- [docker.docs / Get Docker](https://docs.docker.com/get-docker/)
  - [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

![]({{ site.baseurl }}/images/2024/07-16-Docker-installing-ce/01-Docker-install-doc-menu.png)

- [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)