---
title: Docker basic commands
date: 2024-07-16 19:01:00 -0700
categories: [DOCKER, COMMANDS]
tags: [docker]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

## See Docker version

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

## Download and Run a Docker Container

```bash
docker run <image_name>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-22-38:~$ sudo docker run centos
Unable to find image 'centos:latest' locally
latest: Pulling from library/centos
a1d0c7532777: Pull complete 
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
ubuntu@ip-172-31-22-38:~$ 

ubuntu@ip-172-31-22-38:~$ sudo docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                      PORTS     NAMES
f4cc9b9446a2   centos    "/bin/bash"   19 seconds ago   Exited (0) 18 seconds ago             hungry_chandrasekhar
ubuntu@ip-172-31-22-38:~$ 

ubuntu@ip-172-31-22-38:~$ sudo docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
centos       latest    5d0da3dc9764   2 years ago   231MB
ubuntu@ip-172-31-22-38:~$
```
</details><br />

## Download and Run a Docker Container (with specific tag)

```bash
docker run <image_name>:<tag>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker run ubuntu:17.10
Unable to find image 'ubuntu:17.10' locally
17.10: Pulling from library/ubuntu
4ccdce43d1e0: Pull complete 
c95f13c88d92: Pull complete 
82656eee95ad: Pull complete 
78ff727be57a: Pull complete 
448bb314afa5: Pull complete 
Digest: sha256:3b811ac794645dfaa47408f4333ac6e433858ff16908965c68f63d5d315acf94
Status: Downloaded newer image for ubuntu:17.10
ubuntu@ip-XXX-XX-XX-XXX:~$

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       latest    35a88802559d   5 weeks ago   78.1MB
ubuntu       17.10     e211a66937c6   6 years ago   100MB
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

Default tag is `latest`, so by ommiting the tag it is equivalent to running:

```bash
docker run <image_name>:latest
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker run ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
9c704ecd0c69: Pull complete 
Digest: sha256:2e863c44b718727c860746568e1d54afd13b2fa71b160f5cd9058fc436217b30
Status: Downloaded newer image for ubuntu:latest
ubuntu@ip-XXX-XX-XX-XXX:~$

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       latest    35a88802559d   5 weeks ago   78.1MB
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## Run and take output from stdin

```bash
docker run -it <image_name>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker run -it kodekloud/simple-prompt-docker
Welcome! Please enter your name: Hector

Hello and Welcome Hector!
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

With no `-it` there is no prompt that wait for your input.

<details markdown=1>
<summary markdown="span">Output showing the output whitout the -it parameter</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker run kodekloud/simple-prompt-docker
Welcome! Please enter your name: 
Hello and Welcome !
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## Only Download the image

Contrary to `docker run` this command does not start the container after having downloaded the image

```bash
docker pull <image_name>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker images
REPOSITORY                       TAG       IMAGE ID       CREATED         SIZE
busybox                          latest    65ad0d468eb1   14 months ago   4.26MB
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## List all containers

List all containers running

```bash
sudo docker ps
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

List all containers running and idle

```bash
sudo docker ps -a
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker ps -a
CONTAINER ID   IMAGE                            COMMAND            CREATED             STATUS                         PORTS     NAMES
9f849317c6d5   kodekloud/simple-prompt-docker   "sh /opt/app.sh"   About an hour ago   Exited (0) About an hour ago             intelligent_pascal
411cfe4ee127   kodekloud/simple-prompt-docker   "sh /opt/app.sh"   About an hour ago   Exited (0) About an hour ago             gracious_hypatia
70da336bfd7b   kodekloud/simple-prompt-docker   "sh /opt/app.sh"   About an hour ago   Exited (0) About an hour ago             silly_kepler
b3775e7e7270   ubuntu:17.10                     "/bin/bash"        About an hour ago   Exited (0) About an hour ago             gallant_cerf
844cd89094d2   ubuntu                           "/bin/bash"        About an hour ago   Exited (0) About an hour ago             hardcore_kilby
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## Stop a container

```bash
docker stop <container_name> | <container_id>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND        CREATED         STATUS         PORTS     NAMES
46432b03e2a6   centos    "sleep 2000"   4 seconds ago   Up 3 seconds             quizzical_kepler
ubuntu@ip-XXX-XX-XX-XXX:~$

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker stop quizzical_kepler
quizzical_kepler
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## Remove an idle container

```bash
docker rm <container_name> | <container_id>
```

or to remove more than one container from the list of idle containers

```bash
docker rm <container_id_1> <container_id_2> <container_id_3> ...
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
ubuntu@ip-XXX-XX-XX-XXX:~$

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker ps -a
CONTAINER ID   IMAGE                            COMMAND            CREATED              STATUS                          PORTS     NAMES
46432b03e2a6   centos                           "sleep 2000"       About a minute ago   Exited (137) 43 seconds ago               quizzical_kepler
dc54b315fb68   centos                           "/bin/bash"        About a minute ago   Exited (0) About a minute ago             lucid_zhukovsky
9f849317c6d5   kodekloud/simple-prompt-docker   "sh /opt/app.sh"   2 hours ago          Exited (0) 2 hours ago                    intelligent_pascal
411cfe4ee127   kodekloud/simple-prompt-docker   "sh /opt/app.sh"   2 hours ago          Exited (0) 2 hours ago                    gracious_hypatia
70da336bfd7b   kodekloud/simple-prompt-docker   "sh /opt/app.sh"   2 hours ago          Exited (0) 2 hours ago                    silly_kepler
b3775e7e7270   ubuntu:17.10                     "/bin/bash"        2 hours ago          Exited (0) 2 hours ago                    gallant_cerf
844cd89094d2   ubuntu                           "/bin/bash"        2 hours ago          Exited (0) 2 hours ago                    hardcore_kilby
ubuntu@ip-XXX-XX-XX-XXX:~$

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker rm 46432 dc54 9f84 411c 70da3 b377 844c
46432
dc54
9f84
411c
70da3
b377
844c
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## List the local images

```bash
docker images
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker images
REPOSITORY                       TAG       IMAGE ID       CREATED         SIZE
ubuntu                           latest    35a88802559d   5 weeks ago     78.1MB
busybox                          latest    65ad0d468eb1   14 months ago   4.26MB
centos                           latest    5d0da3dc9764   2 years ago     231MB
kodekloud/simple-prompt-docker   latest    c0848fde57d1   4 years ago     4.41MB
ubuntu                           17.10     e211a66937c6   6 years ago     100MB
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## Remove an image from the list of local images

```bash
docker rmi <image_name>
```

**NOTE:** Before removing the image, all containers must be stopped, otherwise Docker gives you an error.

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker images
REPOSITORY                       TAG       IMAGE ID       CREATED         SIZE
ubuntu                           latest    35a88802559d   5 weeks ago     78.1MB
busybox                          latest    65ad0d468eb1   14 months ago   4.26MB
centos                           latest    5d0da3dc9764   2 years ago     231MB
kodekloud/simple-prompt-docker   latest    c0848fde57d1   4 years ago     4.41MB
ubuntu                           17.10     e211a66937c6   6 years ago     100MB
ubuntu@ip-XXX-XX-XX-XXX:~$

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker rmi centos
Untagged: centos:latest
Untagged: centos@sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Deleted: sha256:5d0da3dc976460b72c77d94c8a1ad043720b0416bfc16c52c45d4847e53fadb6
Deleted: sha256:74ddd0ec08fa43d09f32636ba91a0a3053b02cb4627c35051aff89f853606b59
ubuntu@ip-XXX-XX-XX-XXX:~$

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker images
REPOSITORY                       TAG       IMAGE ID       CREATED         SIZE
ubuntu                           latest    35a88802559d   5 weeks ago     78.1MB
busybox                          latest    65ad0d468eb1   14 months ago   4.26MB
kodekloud/simple-prompt-docker   latest    c0848fde57d1   4 years ago     4.41MB
ubuntu                           17.10     e211a66937c6   6 years ago     100MB
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## Execute a command when starting a container

```bash
docker run <image_name> <command_to_run>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker run ubuntu cat /etc/*release*
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
</details><br />

## Execute a command on a running Container

```bash
docker exec <container_name> | <container_id> <command_to_run>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND        CREATED              STATUS              PORTS     NAMES
eb0d9d60adc0   ubuntu    "sleep 2000"   About a minute ago   Up About a minute             beautiful_grothendieck
ubuntu@ip-XXX-XX-XX-XXX:~$

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker exec eb0d9d60adc0 cat /etc/*release*
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
</details><br />

## Run a container in detached mode

By default, running a docker container runs the linux job in foreground occupying the CLI in the shell.

Using the this command will make the container run in detached mode which would be it running in the brackground.

```bash
docker run -d <image_name>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker run -d ubuntu sleep 2000
f9aad5738ee63f8ad0261dedeec8379160f78ec9ed56dbb479df204c59195fe2
ubuntu@ip-XXX-XX-XX-XXX:~$

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND        CREATED         STATUS         PORTS     NAMES
f9aad5738ee6   ubuntu    "sleep 2000"   3 seconds ago   Up 3 seconds             jovial_grothendieck
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## Run a container in attached mode

To get shell access to a container once you run it, use:

```bash
docker run -it <image> bash
```

**NOTE:** The container will stop after one exits the bash.

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$docker run -it ubuntu bash
docker: permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Head "http://%2Fvar%2Frun%2Fdocker.sock/_ping": dial unix /var/run/docker.sock: connect: permission denied.
See 'docker run --help'.
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker run -it ubuntu bash
root@3ce833cdaa07:/# 
root@3ce833cdaa07:/# cat /etc/*release*
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
root@3ce833cdaa07:/# 
root@3ce833cdaa07:/# exit
exit
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## Log in to a container already running

```bash
docker exec -it <container_name> | <container_id> bash
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker exec -it 5133de86677e bash
root@5133de86677e:/# 
root@5133de86677e:/# 
```
</details><br />

## Network port mapping

```bash
docker run -p <host_port>:<container_port> <image_name>
```

<details markdown=1>
<summary markdown="span">output</summary>

`host:80 -> container:5000`

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker run -p 80:5000 kodekloud/webapp
Unable to find image 'kodekloud/webapp:latest' locally
latest: Pulling from kodekloud/webapp
60730f960363: Pull complete 
f7d512d82502: Pull complete 
a7cad26d0357: Pull complete 
25bb6f291ceb: Pull complete 
630ceed02486: Pull complete 
dd2f6ec7cee4: Pull complete 
c24bc06e3fe8: Pull complete 
dd9871015947: Pull complete 
Digest: sha256:f4ad0ca6b27f698a369011c092b1951abf4e4a51cb345d73ab5f7137a9c9dbcc
Status: Downloaded newer image for kodekloud/webapp:latest
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
72.21.198.65 - - [16/Jul/2024 23:56:47] "GET / HTTP/1.1" 200 -
72.21.198.65 - - [16/Jul/2024 23:56:47] "GET / HTTP/1.1" 200 -
72.21.198.65 - - [16/Jul/2024 23:56:47] "GET /favicon.ico HTTP/1.1" 404 -

```
</details><br />

<details markdown=1>
<summary markdown="span">output jenkins container</summary>

`host:8080  -> container:8080`
`host:50000 -> container:50000`

```bash
ubuntu@ip-172-31-93-193:~$ sudo docker run -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
Unable to find image 'jenkins/jenkins:lts' locally
lts: Pulling from jenkins/jenkins
e9aef93137af: Pull complete 
941a647ff357: Pull complete 
f0f251032519: Pull complete 
d4c632816288: Pull complete 
07b0d8ca8ab7: Pull complete 
995009f196ec: Pull complete 
25f6587a42fb: Pull complete 
a559a5c3d549: Pull complete 
aa636159e94b: Pull complete 
862717e1a01c: Pull complete 
6778d605d2d1: Pull complete 
86a41c428de5: Pull complete 
Digest: sha256:50b22a852fa690a195453231f649ee171ee6b15a57c3b49bab4130dd97612c34
Status: Downloaded newer image for jenkins/jenkins:lts
Running from: /usr/share/jenkins/jenkins.war
webroot: /var/jenkins_home/war
2024-07-17 02:47:17.836+0000 [id=1]     INFO    winstone.Logger#logInternal: Beginning extraction from war file
2024-07-17 02:47:19.759+0000 [id=1]     WARNING o.e.j.s.handler.ContextHandler#setContextPath: Empty contextPath
2024-07-17 02:47:19.971+0000 [id=1]     INFO    org.eclipse.jetty.server.Server#doStart: jetty-10.0.20; built: 2024-01-29T20:46:45.278Z; git: 3a745c71c23682146f262b99f4ddc4c1bc41630c; jvm 17.0.11+9
2024-07-17 02:47:20.694+0000 [id=1]     INFO    o.e.j.w.StandardDescriptorProcessor#visitServlet: NO JSP Support for /, did not find org.eclipse.jetty.jsp.JettyJspServlet
2024-07-17 02:47:20.831+0000 [id=1]     INFO    o.e.j.s.s.DefaultSessionIdManager#doStart: Session workerName=node0
2024-07-17 02:47:22.196+0000 [id=1]     INFO    hudson.WebAppMain#contextInitialized: Jenkins home directory: /var/jenkins_home found at: EnvVars.masterEnvVars.get("JENKINS_HOME")
2024-07-17 02:47:22.610+0000 [id=1]     INFO    o.e.j.s.handler.ContextHandler#doStart: Started w.@161f6623{Jenkins v2.452.3,/,file:///var/jenkins_home/war/,AVAILABLE}{/var/jenkins_home/war}
2024-07-17 02:47:22.641+0000 [id=1]     INFO    o.e.j.server.AbstractConnector#doStart: Started ServerConnector@3a1dd365{HTTP/1.1, (http/1.1)}{0.0.0.0:8080}
2024-07-17 02:47:22.673+0000 [id=1]     INFO    org.eclipse.jetty.server.Server#doStart: Started Server@f001896{STARTING}[10.0.20,sto=0] @6162ms
2024-07-17 02:47:22.679+0000 [id=25]    INFO    winstone.Logger#logInternal: Winstone Servlet Engine running: controlPort=disabled
2024-07-17 02:47:23.325+0000 [id=32]    INFO    jenkins.InitReactorRunner$1#onAttained: Started initialization
2024-07-17 02:47:23.387+0000 [id=32]    INFO    jenkins.InitReactorRunner$1#onAttained: Listed all plugins
2024-07-17 02:47:25.511+0000 [id=32]    INFO    jenkins.InitReactorRunner$1#onAttained: Prepared all plugins
2024-07-17 02:47:25.521+0000 [id=32]    INFO    jenkins.InitReactorRunner$1#onAttained: Started all plugins
2024-07-17 02:47:25.540+0000 [id=32]    INFO    jenkins.InitReactorRunner$1#onAttained: Augmented all extensions
2024-07-17 02:47:26.052+0000 [id=31]    INFO    jenkins.InitReactorRunner$1#onAttained: System config loaded
2024-07-17 02:47:26.054+0000 [id=31]    INFO    jenkins.InitReactorRunner$1#onAttained: System config adapted
2024-07-17 02:47:26.057+0000 [id=31]    INFO    jenkins.InitReactorRunner$1#onAttained: Loaded all jobs
2024-07-17 02:47:26.059+0000 [id=32]    INFO    jenkins.InitReactorRunner$1#onAttained: Configuration for all jobs updated
2024-07-17 02:47:26.207+0000 [id=45]    INFO    hudson.util.Retrier#start: Attempt #1 to do the action check updates server
2024-07-17 02:47:27.785+0000 [id=32]    INFO    jenkins.install.SetupWizard#init: 

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

5ffb4495e95445a299a96c1e2dd4fea6

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

2024-07-17 02:48:00.764+0000 [id=32]    INFO    jenkins.InitReactorRunner$1#onAttained: Completed initialization
2024-07-17 02:48:00.799+0000 [id=24]    INFO    hudson.lifecycle.Lifecycle#onReady: Jenkins is fully up and running
2024-07-17 02:48:00.892+0000 [id=45]    INFO    h.m.DownloadService$Downloadable#load: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
2024-07-17 02:48:00.893+0000 [id=45]    INFO    hudson.util.Retrier#start: Performed the action check updates server successfully at the attempt #1
```
</details><br />


## Volumen mapping

```bash
docker run -v <host_directory>:<container_directory> <image>
```

<details markdown=1>
<summary markdown="span">output</summary>

`/opt/datadir` is a directory in the Docker Host
mapped to
`/var/lib/mysql` is a directory in the Docker Container

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker run -v /opt/datadir:/var/lib/mysql mysql
Unable to find image 'mysql:latest' locally
latest: Pulling from library/mysql
d9a40b27c30f: Pull complete 
d948328c7651: Pull complete 
c1e267313ede: Pull complete 
7478f013875a: Pull complete 
9221a2250289: Pull complete 
d1f57baa52d5: Pull complete 
35c3d30e8624: Pull complete 
3d4d1dd0cca6: Pull complete 
08bfe3f7d1c5: Pull complete 
537e7daeeea3: Pull complete 
Digest: sha256:72a37ddc9f839cfd84f1f6815fb31ba26f37f4c200b90e49607797480e3be446
Status: Downloaded newer image for mysql:latest
2024-07-17 00:15:19+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 9.0.0-1.el9 started.
2024-07-17 00:15:20+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2024-07-17 00:15:20+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 9.0.0-1.el9 started.
2024-07-17 00:15:21+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
    You need to specify one of the following as an environment variable:
    - MYSQL_ROOT_PASSWORD
    - MYSQL_ALLOW_EMPTY_PASSWORD
    - MYSQL_RANDOM_ROOT_PASSWORD
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## See container Logs

```bash
docker logs <container_name> | <container_id>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                                                                      NAMES
b11fcc49c11a   jenkins/jenkins:lts   "/usr/bin/tini -- /u…"   12 minutes ago   Up 12 minutes   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:50000->50000/tcp, :::50000->50000/tcp   crazy_chebyshev
5133de86677e   kodekloud/webapp      "/bin/sh -c 'FLASK_A…"   21 minutes ago   Up 21 minutes   0.0.0.0:80->5000/tcp, :::80->5000/tcp                                                      compassionate_feistel
ubuntu@ip-XXX-XX-XX-XXX:~$
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker logs b11fcc49c11a
Running from: /usr/share/jenkins/jenkins.war
webroot: /var/jenkins_home/war
2024-07-17 00:04:18.854+0000 [id=1]     INFO    winstone.Logger#logInternal: Beginning extraction from war file
2024-07-17 00:04:20.620+0000 [id=1]     WARNING o.e.j.s.handler.ContextHandler#setContextPath: Empty contextPath
2024-07-17 00:04:20.781+0000 [id=1]     INFO    org.eclipse.jetty.server.Server#doStart: jetty-10.0.20; built: 2024-01-29T20:46:45.278Z; git: 3a745c71c23682146f262b99f4ddc4c1bc41630c; jvm 17.0.11+9
2024-07-17 00:04:21.387+0000 [id=1]     INFO    o.e.j.w.StandardDescriptorProcessor#visitServlet: NO JSP Support for /, did not find org.eclipse.jetty.jsp.JettyJspServlet
2024-07-17 00:04:21.498+0000 [id=1]     INFO    o.e.j.s.s.DefaultSessionIdManager#doStart: Session workerName=node0
2024-07-17 00:04:22.691+0000 [id=1]     INFO    hudson.WebAppMain#contextInitialized: Jenkins home directory: /var/jenkins_home found at: EnvVars.masterEnvVars.get("JENKINS_HOME")
2024-07-17 00:04:22.971+0000 [id=1]     INFO    o.e.j.s.handler.ContextHandler#doStart: Started w.@161f6623{Jenkins v2.452.3,/,file:///var/jenkins_home/war/,AVAILABLE}{/var/jenkins_home/war}
2024-07-17 00:04:23.010+0000 [id=1]     INFO    o.e.j.server.AbstractConnector#doStart: Started ServerConnector@3a1dd365{HTTP/1.1, (http/1.1)}{0.0.0.0:8080}
2024-07-17 00:04:23.043+0000 [id=1]     INFO    org.eclipse.jetty.server.Server#doStart: Started Server@f001896{STARTING}[10.0.20,sto=0] @5425ms
2024-07-17 00:04:23.048+0000 [id=25]    INFO    winstone.Logger#logInternal: Winstone Servlet Engine running: controlPort=disabled
2024-07-17 00:04:23.543+0000 [id=32]    INFO    jenkins.InitReactorRunner$1#onAttained: Started initialization
2024-07-17 00:04:23.593+0000 [id=31]    INFO    jenkins.InitReactorRunner$1#onAttained: Listed all plugins
2024-07-17 00:04:25.668+0000 [id=31]    INFO    jenkins.InitReactorRunner$1#onAttained: Prepared all plugins
2024-07-17 00:04:25.679+0000 [id=31]    INFO    jenkins.InitReactorRunner$1#onAttained: Started all plugins
2024-07-17 00:04:25.697+0000 [id=31]    INFO    jenkins.InitReactorRunner$1#onAttained: Augmented all extensions
2024-07-17 00:04:26.216+0000 [id=32]    INFO    jenkins.InitReactorRunner$1#onAttained: System config loaded
2024-07-17 00:04:26.218+0000 [id=32]    INFO    jenkins.InitReactorRunner$1#onAttained: System config adapted
2024-07-17 00:04:26.220+0000 [id=32]    INFO    jenkins.InitReactorRunner$1#onAttained: Loaded all jobs
2024-07-17 00:04:26.224+0000 [id=32]    INFO    jenkins.InitReactorRunner$1#onAttained: Configuration for all jobs updated
2024-07-17 00:04:26.378+0000 [id=45]    INFO    hudson.util.Retrier#start: Attempt #1 to do the action check updates server
2024-07-17 00:04:27.912+0000 [id=31]    INFO    jenkins.install.SetupWizard#init: 

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

0c0abdf5852749af9d354108c586f8dc

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

2024-07-17 00:05:02.764+0000 [id=31]    INFO    jenkins.InitReactorRunner$1#onAttained: Completed initialization
2024-07-17 00:05:02.802+0000 [id=24]    INFO    hudson.lifecycle.Lifecycle#onReady: Jenkins is fully up and running
2024-07-17 00:05:02.873+0000 [id=45]    INFO    h.m.DownloadService$Downloadable#load: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
2024-07-17 00:05:02.875+0000 [id=45]    INFO    hudson.util.Retrier#start: Performed the action check updates server successfully at the attempt #1
ubuntu@ip-XXX-XX-XX-XXX:~$

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker logs 5133
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
72.21.198.65 - - [16/Jul/2024 23:56:47] "GET / HTTP/1.1" 200 -
72.21.198.65 - - [16/Jul/2024 23:56:47] "GET / HTTP/1.1" 200 -
72.21.198.65 - - [16/Jul/2024 23:56:47] "GET /favicon.ico HTTP/1.1" 404 -
72.21.198.65 - - [17/Jul/2024 00:03:20] "GET / HTTP/1.1" 200 -
72.21.198.65 - - [17/Jul/2024 00:03:20] "GET / HTTP/1.1" 200 -
72.21.198.65 - - [17/Jul/2024 00:03:26] "GET / HTTP/1.1" 200 -
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## Additional details of the container

```bash
docker inspect <container_name> | <container_id>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                                                                      NAMES
b11fcc49c11a   jenkins/jenkins:lts   "/usr/bin/tini -- /u…"   13 minutes ago   Up 13 minutes   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:50000->50000/tcp, :::50000->50000/tcp   crazy_chebyshev
5133de86677e   kodekloud/webapp      "/bin/sh -c 'FLASK_A…"   22 minutes ago   Up 22 minutes   0.0.0.0:80->5000/tcp, :::80->5000/tcp                                                      compassionate_feistel
ubuntu@ip-XXX-XX-XX-XXX:~$
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker inspect b11f
[
    {
        "Id": "b11fcc49c11a5940dd9c2134e4a8dd9488a2fd52095c96361ae2c00bad891d55",
        "Created": "2024-07-17T00:04:17.034345761Z",
        "Path": "/usr/bin/tini",
        "Args": [
            "--",
            "/usr/local/bin/jenkins.sh"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 5830,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2024-07-17T00:04:17.26024117Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:7fef0d6ded53ed2c0bb64b8e07763090d7ae8cb509b12c3eb5d4bef89a02a704",
        "ResolvConfPath": "/var/lib/docker/containers/b11fcc49c11a5940dd9c2134e4a8dd9488a2fd52095c96361ae2c00bad891d55/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/b11fcc49c11a5940dd9c2134e4a8dd9488a2fd52095c96361ae2c00bad891d55/hostname",
        "HostsPath": "/var/lib/docker/containers/b11fcc49c11a5940dd9c2134e4a8dd9488a2fd52095c96361ae2c00bad891d55/hosts",
        "LogPath": "/var/lib/docker/containers/b11fcc49c11a5940dd9c2134e4a8dd9488a2fd52095c96361ae2c00bad891d55/b11fcc49c11a5940dd9c2134e4a8dd9488a2fd52095c96361ae2c00bad891d55-json.log",
        "Name": "/crazy_chebyshev",
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
                "50000/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "50000"
                    }
                ],
                "8080/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8080"
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
                58,
                262
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
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
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": [],
            "BlkioDeviceWriteBps": [],
            "BlkioDeviceReadIOps": [],
            "BlkioDeviceWriteIOps": [],
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": [],
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
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/5baf9a9e67a705a4f72bce1ae6fc387e5b265274391cacdf0ab71c0bf76f0d09-init/diff:/var/lib/docker/overlay2/00b55c029a19a614e5b849ac55b33365ac7ed7177d793d2df5cee58f2803a2f7/diff:/var/lib/docker/overlay2/e1f17fc0d5639333b700beac1e1a7b2c9b00583a77f0707835117503a3fe65f8/diff:/var/lib/docker/overlay2/d55e9f557380dde3d11bb313aa8c30f9ea92a95da363fabd171c4e65b6ce5ee7/diff:/var/lib/docker/overlay2/e521d40e0c11920e275cd9fac21b9377996123e65f65332f9ee32f335ef05db0/diff:/var/lib/docker/overlay2/6d2430ee19f9f7ee83e2a7e7dd4178ac3ccdaa3e3f928501a5e1874f94e0460c/diff:/var/lib/docker/overlay2/a7a26ec7d59242bcbb11475dfe0dc4df89f2e4e9e42b3d0dd2afbb3158b1da83/diff:/var/lib/docker/overlay2/e76071d2a53d5b157008ab28b7e7624fba3c7adab6fd395d19312d11a8d367eb/diff:/var/lib/docker/overlay2/b2bb2efd39983c0109c7f3ab292b907043fdeeeef73337bca74dbe0e62281cec/diff:/var/lib/docker/overlay2/37bbe3384005df134f38b246367ec9d6898a6d0b41ab8743f34909c20cdecac2/diff:/var/lib/docker/overlay2/0bb51dd441b80b604ba4e747012137687d23a93d7c724518814c3e46ac6df0de/diff:/var/lib/docker/overlay2/7b961795a2ea1d29732e892f7a7a29c252becac17c185ec981cd42bddfb0f7ef/diff:/var/lib/docker/overlay2/3a774487ba5d27a5b1211d8cfdf7f906ef554b985284606cd486dd3f9893aca8/diff",
                "MergedDir": "/var/lib/docker/overlay2/5baf9a9e67a705a4f72bce1ae6fc387e5b265274391cacdf0ab71c0bf76f0d09/merged",
                "UpperDir": "/var/lib/docker/overlay2/5baf9a9e67a705a4f72bce1ae6fc387e5b265274391cacdf0ab71c0bf76f0d09/diff",
                "WorkDir": "/var/lib/docker/overlay2/5baf9a9e67a705a4f72bce1ae6fc387e5b265274391cacdf0ab71c0bf76f0d09/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "911c8855ba48075ad04c1d560c81b943a41d66afe73aa7d84348c131b4be5619",
                "Source": "/var/lib/docker/volumes/911c8855ba48075ad04c1d560c81b943a41d66afe73aa7d84348c131b4be5619/_data",
                "Destination": "/var/jenkins_home",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "b11fcc49c11a",
            "Domainname": "",
            "User": "jenkins",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "50000/tcp": {},
                "8080/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "JENKINS_HOME=/var/jenkins_home",
                "JENKINS_SLAVE_AGENT_PORT=50000",
                "REF=/usr/share/jenkins/ref",
                "JENKINS_VERSION=2.452.3",
                "JENKINS_UC=https://updates.jenkins.io",
                "JENKINS_UC_EXPERIMENTAL=https://updates.jenkins.io/experimental",
                "JENKINS_INCREMENTALS_REPO_MIRROR=https://repo.jenkins-ci.org/incrementals",
                "COPY_REFERENCE_FILE_LOG=/var/jenkins_home/copy_reference_file.log",
                "JAVA_HOME=/opt/java/openjdk"
            ],
            "Cmd": null,
            "Image": "jenkins/jenkins:lts",
            "Volumes": {
                "/var/jenkins_home": {}
            },
            "WorkingDir": "",
            "Entrypoint": [
                "/usr/bin/tini",
                "--",
                "/usr/local/bin/jenkins.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "org.opencontainers.image.description": "The Jenkins Continuous Integration and Delivery server",
                "org.opencontainers.image.licenses": "MIT",
                "org.opencontainers.image.revision": "9fba54713675c711138675b4aaaff6268c14d31a",
                "org.opencontainers.image.source": "https://github.com/jenkinsci/docker",
                "org.opencontainers.image.title": "Official Jenkins Docker image",
                "org.opencontainers.image.url": "https://www.jenkins.io/",
                "org.opencontainers.image.vendor": "Jenkins project",
                "org.opencontainers.image.version": "2.452.3"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "ac5f38eb91306e6f896e51443b42baada0e46c7a8b816b7a87e2c7898b223c19",
            "SandboxKey": "/var/run/docker/netns/ac5f38eb9130",
            "Ports": {
                "50000/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "50000"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "50000"
                    }
                ],
                "8080/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8080"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "8080"
                    }
                ]
            },
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "2b3a2f1c547f769dbcb0cdfa321d00e3c02dd8c8cca6f6e38a11362ee9addce1",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.3",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:03",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "02:42:ac:11:00:03",
                    "DriverOpts": null,
                    "NetworkID": "5bda5debf60a9c1f2feddd3485fa8024f22eb1be2475babced64321b671487be",
                    "EndpointID": "2b3a2f1c547f769dbcb0cdfa321d00e3c02dd8c8cca6f6e38a11362ee9addce1",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.3",
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
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## docker info ##

Shows information about the docker installed in the system, including what storage drivers are in use.

```bash
docker info
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker info
Client: Docker Engine - Community
 Version:    27.0.3
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.15.1
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v2.28.1
    Path:     /usr/libexec/docker/cli-plugins/docker-compose

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 27.0.3
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 2bf793ef6dc9a18e00cb12efb64355c2c9d5eb41
 runc version: v1.1.13-0-g58aa920
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 6.8.0-1009-aws
 Operating System: Ubuntu 24.04 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 957.4MiB
 Name: ip-XXX-XX-XX-XXX
 ID: af525743-ebd7-41b9-9a15-82a166676245
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

## docker history

```bash
docker history <image_name>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XX:~$ sudo docker history gustavos86/my-custom-app
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
0e824991dd69   27 minutes ago   ENTRYPOINT ["/bin/sh" "-c" "FLASK_APP=/opt/a…   0B        buildkit.dockerfile.v0
<missing>      27 minutes ago   COPY app.py /opt/app.py # buildkit              254B      buildkit.dockerfile.v0
<missing>      27 minutes ago   RUN /bin/sh -c apt install -y python3 python…   479MB     buildkit.dockerfile.v0
<missing>      35 minutes ago   RUN /bin/sh -c apt update # buildkit            38.5MB    buildkit.dockerfile.v0
<missing>      35 minutes ago   RUN /bin/sh -c ln -snf /usr/share/zoneinfo/$…   20B       buildkit.dockerfile.v0
<missing>      35 minutes ago   ENV TZ=America/Los_Angeles                      0B        buildkit.dockerfile.v0
<missing>      5 weeks ago      /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      5 weeks ago      /bin/sh -c #(nop) ADD file:5601f441718b0d192…   78.1MB    
<missing>      5 weeks ago      /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B        
<missing>      5 weeks ago      /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B        
<missing>      5 weeks ago      /bin/sh -c #(nop)  ARG LAUNCHPAD_BUILD_ARCH     0B        
<missing>      5 weeks ago      /bin/sh -c #(nop)  ARG RELEASE                  0B        
ubuntu@ip-XXX-XX-XX-XX:~$ 
```
</details><br />

## Volume mounting

Create a new Volume.

```bash
docker volume create <volume_name>
```

The volume created is stored in `/var/lib/docker/volumes` directory.

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker volume create data_volume
data_volume
ubuntu@ip-XXX-XX-XX-XXX:~$

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo ls -l /var/lib/docker/volumes
total 32
brw------- 1 root root 202, 1 Jul 20 17:40 backingFsBlockDev
drwx-----x 3 root root   4096 Jul 20 20:50 data_volume
-rw------- 1 root root  32768 Jul 20 20:50 metadata.db
ubuntu@ip-XXX-XX-XX-XXX:~$
```
</details><br />

You can mount the volume using the `-v` argument.

```bash
docker run -v <volume_name>:<container_dictory_path> <container_name>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
buntu@ip-XXX-XX-XX-XXX:~$ sudo docker run -e MYSQL_ROOT_PASSWORD=password -v data_volume:/var/lib/mysql mysql
2024-07-20 20:52:57+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 9.0.0-1.el9 started.
2024-07-20 20:52:57+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2024-07-20 20:52:57+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 9.0.0-1.el9 started.
2024-07-20 20:52:57+00:00 [Note] [Entrypoint]: Initializing database files
2024-07-20T20:52:57.806683Z 0 [System] [MY-015017] [Server] MySQL Server Initialization - start.
2024-07-20T20:52:57.809749Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 9.0.0) initializing of server in progress as process 78
2024-07-20T20:52:57.835482Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2024-07-20T20:52:59.061288Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2024-07-20T20:53:01.589397Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
2024-07-20T20:53:05.096094Z 0 [System] [MY-015018] [Server] MySQL Server Initialization - end.
2024-07-20 20:53:05+00:00 [Note] [Entrypoint]: Database files initialized
2024-07-20 20:53:05+00:00 [Note] [Entrypoint]: Starting temporary server
2024-07-20T20:53:05.165449Z 0 [System] [MY-015015] [Server] MySQL Server - start.
2024-07-20T20:53:05.495929Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 9.0.0) starting as process 115
2024-07-20T20:53:05.519564Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2024-07-20T20:53:06.590105Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2024-07-20T20:53:07.157793Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2024-07-20T20:53:07.157916Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2024-07-20T20:53:07.167293Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2024-07-20T20:53:07.227716Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: /var/run/mysqld/mysqlx.sock
2024-07-20T20:53:07.228880Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '9.0.0'  socket: '/var/run/mysqld/mysqld.sock'  port: 0  MySQL Community Server - GPL.
2024-07-20 20:53:07+00:00 [Note] [Entrypoint]: Temporary server started.
'/var/lib/mysql/mysql.sock' -> '/var/run/mysqld/mysqld.sock'
Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/leap-seconds.list' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/leapseconds' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/tzdata.zi' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone1970.tab' as time zone. Skipping it.

2024-07-20 20:53:10+00:00 [Note] [Entrypoint]: Stopping temporary server
2024-07-20T20:53:10.488278Z 10 [System] [MY-013172] [Server] Received SHUTDOWN from user root. Shutting down mysqld (Version: 9.0.0).
2024-07-20T20:53:11.423500Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 9.0.0)  MySQL Community Server - GPL.
2024-07-20T20:53:11.423521Z 0 [System] [MY-015016] [Server] MySQL Server - end.
2024-07-20 20:53:11+00:00 [Note] [Entrypoint]: Temporary server stopped

2024-07-20 20:53:11+00:00 [Note] [Entrypoint]: MySQL init process done. Ready for start up.

2024-07-20T20:53:11.596401Z 0 [System] [MY-015015] [Server] MySQL Server - start.
2024-07-20T20:53:11.918606Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 9.0.0) starting as process 1
2024-07-20T20:53:11.944009Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2024-07-20T20:53:12.871894Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2024-07-20T20:53:13.250910Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2024-07-20T20:53:13.251315Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2024-07-20T20:53:13.258938Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2024-07-20T20:53:13.324030Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '9.0.0'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
2024-07-20T20:53:13.324806Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                 NAMES
d9a9c8b252f6   mysql     "docker-entrypoint.s…"   35 seconds ago   Up 34 seconds   3306/tcp, 33060/tcp   competent_dewdney
ubuntu@ip-XXX-XX-XX-XXX:~$

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker inspect d9a9c8b252f6
[
    {
        "HostConfig": {
            "Binds": [
                "data_volume:/var/lib/mysql"
            ],
        ...
        }
        ...
        "Mounts": [
            {
                "Type": "volume",
                "Name": "data_volume",
                "Source": "/var/lib/docker/volumes/data_volume/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
    }
]
```
</details><br />

**NOTE:** If the volume was not previously created when running `docker run -v <volume_name>:<container_dictory_path>`, the volume will be created at that moment.

## docker system

Show docker disk usage

```bash
docker system df
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          7         2         1.395GB   783.7MB (56%)
Containers      4         1         6B        0B (0%)
Local Volumes   6         1         188.7MB   131.5MB (69%)
Build Cache     45        0         61.3MB    61.3MB
cloud_user@553b1e446c1c:~$
```
</details><br />

Show docker disk usage verbose

```bash
docker system df -v
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo docker system df -v
Images space usage:

REPOSITORY                                                     TAG       IMAGE ID       CREATED        SIZE      SHARED SIZE   UNIQUE SIZE   CONTAINERS
cloud_user-worker                                              latest    7230f713aad7   2 days ago     194MB     0B            193.7MB       0
cloud_user-result                                              latest    6dbbfcb39ad0   2 days ago     219MB     74.78MB       144.3MB       0
654654406075.dkr.ecr.us-east-1.amazonaws.com/cloud_user-vote   latest    3cd62f1d4c2c   2 days ago     153MB     74.78MB       78.46MB       0
mysql                                                          latest    5cde95de907d   2 weeks ago    586MB     0B            585.7MB       3
redis                                                          latest    9c893be668ac   8 weeks ago    116MB     74.78MB       41.66MB       0
registry                                                       2         6a3edb1d5eb6   9 months ago   25.4MB    0B            25.43MB       1
postgres                                                       9.4       ed5a45034282   4 years ago    251MB     0B            250.7MB       0

Containers space usage:

CONTAINER ID   IMAGE        COMMAND                  LOCAL VOLUMES   SIZE      CREATED          STATUS                     NAMES
3e3b199226f5   mysql        "docker-entrypoint.s…"   0               6B        6 minutes ago    Up 6 minutes               relaxed_cori
ca9f4707ad90   mysql        "docker-entrypoint.s…"   0               0B        6 minutes ago    Exited (1) 6 minutes ago   condescending_blackburn
8e718f923c27   mysql        "docker-entrypoint.s…"   0               0B        15 minutes ago   Exited (0) 8 minutes ago   vibrant_hawking
42c3da90c850   registry:2   "/entrypoint.sh /etc…"   1               0B        2 days ago       Exited (2) 2 days ago      registry

Local Volumes space usage:

VOLUME NAME                                                        LINKS     SIZE
19ef997c9bfa9e6d25c0a4c8b9b9e113d402fbd2a6f8a0d5e87d478a2d7f1777   0         37.14MB
44c74df1fdc85fabf2e2769f9e8964bde2cc15b7ea4b0340b17c3f0ac081cbfe   0         37.16MB
63d275abe9e987d5e835983026007d451ba152531493767088a0214f559abc64   0         88B
73bf75f4c6f4f20a783fd1b679b475a63c707ecb3f8fbf28bf851906f73d5140   0         57.2MB
8c8d8553683b4d52e3da2267b82f6620b0689c09f5b2ad1a02e39434a7f84a47   0         88B
d09faeb59985b4d2b7cb535fd7650ebcecae8b4bca9989a750985187d8871978   1         57.2MB

Build cache usage: 61.3MB

CACHE ID       CACHE TYPE     SIZE      CREATED      LAST USED    USAGE     SHARED
thqz57xq5bxq   regular        0B        2 days ago   2 days ago   1         true
w761q5qm8kr4   regular        0B        2 days ago   2 days ago   1         true
0hs8qb99t5c3   regular        0B        2 days ago   2 days ago   1         true
ez1vqmymswak   regular        0B        2 days ago   2 days ago   2         true
avsurygii952   regular        0B        2 days ago   2 days ago   1         true
spmxe5y9ujas   regular        0B        2 days ago   2 days ago   1         true
5qd69e71ltgq   regular        0B        2 days ago   2 days ago   1         true
to7pjca34n8u   regular        0B        2 days ago   2 days ago   2         true
8f2pp7ie5iq8   regular        0B        2 days ago   2 days ago   2         true
oo4megwsl2gf   regular        0B        2 days ago   2 days ago   2         true
kgptdic7kqrc   regular        0B        2 days ago   2 days ago   1         false
6q3pm8mq2vfd   regular        0B        2 days ago   2 days ago   1         false
t6si2nu3d8o0   regular        0B        2 days ago   2 days ago   1         false
761nvcn9jtey   regular        4.47MB    2 days ago   2 days ago   1         true
sp37ext1tuwu   source.local   0B        2 days ago   2 days ago   1         false
owxc0acwc4q3   regular        0B        2 days ago   2 days ago   1         true
9fmaqlfkf0b5   regular        18.8MB    2 days ago   2 days ago   1         true
q7a6wupg5avj   source.local   1.05kB    2 days ago   2 days ago   1         false
z5js1vnbcd8v   regular        0B        2 days ago   2 days ago   1         true
lr58dxdr6cci   regular        21B       2 days ago   2 days ago   1         true
6epssjj29ae2   regular        6.13kB    2 days ago   2 days ago   1         true
8iooyruf5wc4   source.local   6.13kB    2 days ago   2 days ago   1         false
t1ubh3xj2syi   regular        9.27MB    2 days ago   2 days ago   1         true
ghcy6uc2rh9g   regular        0B        2 days ago   2 days ago   1         true
hsqlo8rmipb6   regular        278kB     2 days ago   2 days ago   1         true
tgx5p2mxfjia   regular        0B        2 days ago   2 days ago   1         true
xrsyzir29yas   source.local   486B      2 days ago   2 days ago   1         false
8ve64gq61kn4   regular        4.67MB    2 days ago   2 days ago   1         true
ol1i2rrzvzr1   source.local   278kB     2 days ago   2 days ago   1         false
sm61yjnzzqay   regular        68.9kB    2 days ago   2 days ago   1         true
emf6t5tol6rk   regular        12.9MB    2 days ago   2 days ago   1         true
olr6i2gj8oeg   source.local   14B       2 days ago   2 days ago   1         false
4vlmrdnbr6qm   source.local   0B        2 days ago   2 days ago   1         false
li6aaejipjr7   regular        3.55MB    2 days ago   2 days ago   1         true
nr29r8tq5q48   regular        6.94kB    2 days ago   2 days ago   1         false
ok914f47jcg0   regular        0B        2 days ago   2 days ago   1         true
pqizb27e1jek   source.local   997B      2 days ago   2 days ago   1         false
wq5toj5nvj7o   regular        0B        2 days ago   2 days ago   1         false
wuxml20cnwuu   regular        0B        2 days ago   2 days ago   1         false
yjae00175gd9   regular        16.1MB    2 days ago   2 days ago   1         false
dbpn1isljbm7   regular        0B        2 days ago   2 days ago   1         false
l2turhvez38p   regular        389B      2 days ago   2 days ago   1         false
mc3auj7akax0   regular        0B        2 days ago   2 days ago   1         true
iu2cs9zayptf   source.local   6.94kB    2 days ago   2 days ago   1         false
5ph2ulyd73dq   regular        44.9MB    2 days ago   2 days ago   1         false
cloud_user@553b1e446c1c:~$
```
</details><br />

## docker build

The `docker build`  command is used to create a new Docker Image based on a `Dockerfile`.

For more information refer to this post [Docker create an Image using Dockerfile]({% post_url 2024-07-17-Docker-create-image %})

## Resources

- [https://hub.docker.com/](https://hub.docker.com/)