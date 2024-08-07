---
title: Running FRR in a Docker Container
date: 2024-08-03 06:30:00 -0700
categories: [FRR, INSTALL]
tags: [frr]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/frr_logo.svg)

![]({{ site.baseurl }}/images/services/docker_logo.png)

## Introduction

FRR is free software that implements and manages various IPv4 and IPv6 routing protocols. It runs on nearly all distributions of Linux and BSD and supports all modern CPU architectures.

### Interactive Shell

FRR offers an IOS-like interactive shell called `vtysh` where a user can run individual configuration or show commands. To get into this shell, issue the `vtysh` command from either a privilege user (root, or with sudo) or a user account that is part of the `frrvty` group.

```bash
root@ub18:~# vtysh

Hello, this is FRRouting (version 8.1-dev).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

ub18#
```

## Download the FRR image from GitHub

Used the `docker pull` command to download the FRR image from its DockerHub repository [frrouting/frr image in DockerHub](https://hub.docker.com/r/frrouting/frr)

```bash
docker pull frrouting/frr
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ sudo docker pull frrouting/frr
Using default tag: latest
latest: Pulling from frrouting/frr
213ec9aee27d: Pull complete
e20f854e72c8: Pull complete
2e60d6091b05: Pull complete
298220357c95: Pull complete
4f4fb700ef54: Pull complete
3d6bd20b68ba: Pull complete
64f8711cd905: Pull complete
Digest: sha256:990e83490108b686fd6df3b1cafa6bdbb2714acb00eedb9a89693946f46f45ce
Status: Downloaded newer image for frrouting/frr:latest
docker.io/frrouting/frr:latest
cloud_user@553b1e446c1c:~/installing_frr$
```
</details><br />

Run `docker images` to verify the FRR Docker image was indeed downloaded correctly

```bash
docker images
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ sudo docker images
REPOSITORY                                                     TAG       IMAGE ID       CREATED         SIZE
frrouting/frr                                                  latest    d19bacb84eae   21 months ago   151MB
cloud_user@553b1e446c1c:~/installing_frr$
```
</details><br />

Start the container based on the downloaded FRR image in background using the `-d` flag. 

```bash
docker run -d --name router1 frrouting/frr
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ sudo docker run -d --name router1 frrouting/frr
dbb607874b8ea33d234a5199a7ed3605c47aa3dcf9bd67b1710241f21b2e0c95
cloud_user@553b1e446c1c:~/installing_frr$

cloud_user@553b1e446c1c:~/installing_frr$ sudo docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED              STATUS              PORTS     NAMES
dbb607874b8e   frrouting/frr   "/sbin/tini -- /usr/…"   About a minute ago   Up About a minute             router1
cloud_user@553b1e446c1c:~/installing_frr$
```
</details><br />

**IMPORTANT:** If you try to spin up the container with `docker run` and log in with `bash` at the same time, like when using `docker run -it --name router1 frrouting/frr bash` then `vtysh` will NOT start properly.

Log in to the conteiner using either `bash` or `sh`

```bash
docker exec -it <CONTEINERS_NAME> bash
```

or

```bash
docker exec -it <CONTEINERS_NAME> sh
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ sudo docker exec -it router1 bash
bash-5.1# exit
exit
cloud_user@553b1e446c1c:~/installing_frr$
cloud_user@553b1e446c1c:~/installing_frr$ sudo docker exec -it router1 sh
/ #
```
</details><br />

Run `vtysh` to enter FRR's IOS-like interactive shell

```bash
vtysh
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ sudo docker exec -it router1 bash
bash-5.1# vtysh
% Can't open configuration file /etc/frr/vtysh.conf due to 'No such file or directory'.

Hello, this is FRRouting (version 8.4_git).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

dbb607874b8e# show running-config
Building configuration...

Current configuration:
!
frr version 8.4_git
frr defaults traditional
hostname dbb607874b8e
!
end
dbb607874b8e#
```
</details><br />

### The problem

Seems that `zebra` is not starting

```bash
bash-5.1# vtysh
show % Can't open configuration file /etc/frr/vtysh.conf due to 'No such file or directory'.

Hello, this is FRRouting (version 8.4_git).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

ab3fbb10ffc4# show ip route
zebra is not running
ab3fbb10ffc4#
```

and the `/etc/init.d/frr status` is not working for some reason

```bash
bash-5.1# /etc/init.d/frr status
bash-5.1#

bash-5.1# /etc/init.d/frr start
privs_init: initial cap_set_proc failed: Operation not permitted
Wanted caps: cap_net_bind_service,cap_net_admin,cap_net_raw,cap_sys_admin=p
Have   caps: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=p
privs_init: initial cap_set_proc failed: Operation not permitted
Wanted caps: cap_net_bind_service,cap_net_admin,cap_net_raw,cap_sys_admin=p
Have   caps: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=p
Exiting from the script
bash-5.1#

bash-5.1# /etc/init.d/frr
Usage: /etc/init.d/frr {start|stop|status|reload|restart|force-reload|<priority>} [daemon]
       E.g. '/etc/init.d/frr 5' would start all daemons with a prio 1-5.
       reload applies only modifications from the running config to all daemons.
       reload neither restarts starts any daemon nor starts any new ones.
       Read /usr/share/doc/frr/README.Debian for details.
bash-5.1#
```

### The solution...

When I clone this Git repo [ksator/frrouting_demo in GitHub](https://github.com/ksator/frrouting_demo/blob/master/Dockerfile) and use `docker compose` using its file `docker-compose-demo1.yml`, the FRR container created has the `zebra` process working fine.

I tried several things to make my own Image from the `Dockerfile` but I keep getting the problem with the `zebra` process refusing to run.

Here, a couple of containers created from `docker compose` based on `docker-compose-demo1.yml` working fine:

```bash
cloud_user@553b1e446c1c:~/ksator_frrouting_demo$ cd frrouting_demo/
cloud_user@553b1e446c1c:~/ksator_frrouting_demo/frrouting_demo$ ll
total 52
drwxrwxr-x 5 cloud_user cloud_user  4096 Aug  3 17:34 ./
drwxrwxr-x 3 cloud_user cloud_user  4096 Aug  3 17:34 ../
drwxrwxr-x 8 cloud_user cloud_user  4096 Aug  3 17:34 .git/
-rw-rw-r-- 1 cloud_user cloud_user    16 Aug  3 17:34 .gitignore
-rw-rw-r-- 1 cloud_user cloud_user   447 Aug  3 17:34 Dockerfile
-rw-rw-r-- 1 cloud_user cloud_user 15849 Aug  3 17:34 README.md
drwxrwxr-x 4 cloud_user cloud_user  4096 Aug  3 17:34 demo1/
drwxrwxr-x 4 cloud_user cloud_user  4096 Aug  3 17:34 demo2/
-rw-rw-r-- 1 cloud_user cloud_user  1383 Aug  3 17:34 docker-compose-demo1.yml
-rw-rw-r-- 1 cloud_user cloud_user  1383 Aug  3 17:34 docker-compose-demo2.yml
cloud_user@553b1e446c1c:~/ksator_frrouting_demo/frrouting_demo$

cloud_user@553b1e446c1c:~$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~/ksator_frrouting_demo/frrouting_demo$ docker compose -f docker-compose-demo1.yml up -d
WARN[0000] /home/cloud_user/ksator_frrouting_demo/frrouting_demo/docker-compose-demo1.yml: `version` is obsolete
[+] Running 2/2
 ✔ Container frr200  Started                                                                                                                                                                                                                   3.1s
 ✔ Container frr100  Started                                                                                                                                                                                                                   3.1s
cloud_user@553b1e446c1c:~/ksator_frrouting_demo/frrouting_demo$

cloud_user@553b1e446c1c:~/ksator_frrouting_demo/frrouting_demo$ docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED       STATUS         PORTS     NAMES
b34035faae0d   ksator/frr:1.0   "/bin/bash /etc/frr/…"   9 hours ago   Up 3 minutes             frr100
37ece5e00ac0   ksator/frr:1.0   "/bin/bash /etc/frr/…"   9 hours ago   Up 3 minutes             frr200
cloud_user@553b1e446c1c:~/ksator_frrouting_demo/frrouting_demo$

cloud_user@553b1e446c1c:~/ksator_frrouting_demo/frrouting_demo$ docker exec -it frr100 bash
root@b34035faae0d:/# /etc/init.d/frr status
 * Status of watchfrr: running
 * Status of zebra: running
 * Status of mgmtd: running
 * Status of bgpd: running
 * Status of staticd: running
root@b34035faae0d:/#
root@b34035faae0d:/# vtysh

Hello, this is FRRouting (version 10.0.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

2024/08/04 04:03:48 [YDG3W-JND95] FD Limit set: 1048576 is stupidly large.  Is this what you intended?  Consider using --limit-fds also limiting size to 100000
b34035faae0d# show ip route
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

K>* 0.0.0.0/0 [0/0] via 192.168.1.1, eth0, 00:03:56
C>* 192.168.1.0/24 is directly connected, eth0, 00:03:56
L>* 192.168.1.100/32 is directly connected, eth0, 00:03:56
C>* 192.168.100.0/24 is directly connected, eth1, 00:03:56
L>* 192.168.100.100/32 is directly connected, eth1, 00:03:56
B>* 192.168.200.0/24 [20/0] via 192.168.1.200, eth0, weight 1, 00:03:50
b34035faae0d#
```

## One failed attempt... for reference

Created a `Dockerfile` with the following. The content was obtained from [ksator/frrouting_demo in GitHub](https://github.com/ksator/frrouting_demo/blob/master/Dockerfile)

```bash
FROM ubuntu:20.04
RUN apt-get update && apt-get -y upgrade 
RUN apt-get install -y lsb-release net-tools wget git vim curl tcpdump iputils-ping tree jq telnet sudo nano gnupg2
RUN curl -s https://deb.frrouting.org/frr/keys.asc | sudo apt-key add -
ENV FRRVER="frr-stable"
RUN echo deb https://deb.frrouting.org/frr $(lsb_release -s -c) $FRRVER | sudo tee -a /etc/apt/sources.list.d/frr.list
RUN apt update && apt install -y frr frr-pythontools
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ vim Dockerfile
```
</details><br />

Built the image from `Dockerfile` with `docker build`

```bash
docker build . -t frr_local_build
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ sudo docker build . -t frr_local_build
[+] Building 98.5s (10/10) FINISHED                                                                                                      docker:default
 => [internal] load build definition from Dockerfile                                                                                               0.2s
 => => transferring dockerfile: 483B                                                                                                               0.0s
 => [internal] load metadata for docker.io/library/ubuntu:20.04                                                                                    0.7s
 => [internal] load .dockerignore                                                                                                                  0.1s
 => => transferring context: 2B                                                                                                                    0.0s
 => [1/6] FROM docker.io/library/ubuntu:20.04@sha256:0b897358ff6624825fb50d20ffb605ab0eaea77ced0adb8c6a4b756513dec6fc                              2.9s
 => => resolve docker.io/library/ubuntu:20.04@sha256:0b897358ff6624825fb50d20ffb605ab0eaea77ced0adb8c6a4b756513dec6fc                              0.1s
 => => sha256:0b897358ff6624825fb50d20ffb605ab0eaea77ced0adb8c6a4b756513dec6fc 1.13kB / 1.13kB                                                     0.0s
 => => sha256:d86db849e59626d94f768c679aba441163c996caf7a3426f44924d0239ffe03f 424B / 424B                                                         0.0s
 => => sha256:5f5250218d28ad6612bf653eced407165dd6475a4daf9210b299fed991e172e9 2.30kB / 2.30kB                                                     0.0s
 => => sha256:9ea8908f47652b59b8055316d9c0e16b365e2b5cee15d3efcb79e2957e3e7cad 27.51MB / 27.51MB                                                   0.4s
 => => extracting sha256:9ea8908f47652b59b8055316d9c0e16b365e2b5cee15d3efcb79e2957e3e7cad                                                          2.0s
 => [2/6] RUN apt-get update && apt-get -y upgrade                                                                                                 9.8s
 => [3/6] RUN apt-get install -y lsb-release net-tools wget git vim curl tcpdump iputils-ping tree jq telnet sudo nano gnupg2                     61.0s
 => [4/6] RUN curl -s https://deb.frrouting.org/frr/keys.asc | sudo apt-key add -                                                                  1.8s
 => [5/6] RUN echo deb https://deb.frrouting.org/frr $(lsb_release -s -c) frr-stable | sudo tee -a /etc/apt/sources.list.d/frr.list                0.8s
 => [6/6] RUN apt update && apt install -y frr frr-pythontools                                                                                    18.2s
 => exporting to image                                                                                                                             2.5s
 => => exporting layers                                                                                                                            2.3s
 => => writing image sha256:e4c9267854185e5e56ef6af734b25baae4115d959fb3eff1e0e5a45da0d3f9d3                                                       0.0s
 => => naming to docker.io/library/frr_local_build                                                                                                 0.0s
cloud_user@553b1e446c1c:~/installing_frr$
```
</details><br />

Verified the image was built properly in Docker

```bash
docker images
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ sudo docker images
REPOSITORY                                                     TAG       IMAGE ID       CREATED         SIZE
frr_local_build                                                latest    e4c926785418   2 minutes ago   346MB
```
</details><br />

However, the `zebra` and `mgmtd` did not start.

```bash
sudo docker run -it --name router1 frr_local_build bash
```

Notice I had to start the process manually

```bash
root@df61a7f9ae1d:/# /etc/init.d/frr status
 * Status of watchfrr: FAILED
 * Status of zebra: FAILED
 * Status of mgmtd: FAILED
 * Status of staticd: FAILED
root@df61a7f9ae1d:/#
root@df61a7f9ae1d:/# /etc/init.d/frr start
...
root@df61a7f9ae1d:/#
root@df61a7f9ae1d:/# /etc/init.d/frr status
 * Status of watchfrr: running
 * Status of zebra: FAILED
 * Status of mgmtd: FAILED
 * Status of staticd: running
root@df61a7f9ae1d:/#
```

With `zebra` down not much can be done in FRR's Interactive Shell

```bash
root@df61a7f9ae1d:/# vtysh
...
df61a7f9ae1d#
df61a7f9ae1d# show ip route
zebra is not running
df61a7f9ae1d#
```

Complete outputs

<details markdown=1>
<summary markdown="span">outputs</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ sudo docker run -it --name router1 frr_local_build bash
root@df61a7f9ae1d:/# vtysh
Exiting: failed to connect to any daemons.
root@df61a7f9ae1d:/#
root@df61a7f9ae1d:/# /etc/init.d/frr status
 * Status of watchfrr: FAILED
 * Status of zebra: FAILED
 * Status of mgmtd: FAILED
 * Status of staticd: FAILED
root@df61a7f9ae1d:/#
cloud_user@553b1e446c1c:~$ docker run -it --name router2_2 frr_local_build bash
root@de0adeff7f28:/# /etc/init.d/frr status
 * Status of watchfrr: FAILED
 * Status of zebra: FAILED
 * Status of mgmtd: FAILED
 * Status of staticd: FAILED
root@de0adeff7f28:/# /etc/init.d/frr start
 * Starting watchfrr with command: '  /usr/lib/frr/watchfrr  -d  -F traditional   zebra mgmtd staticd'
2024/08/03 17:07:49 WATCHFRR: [YDG3W-JND95] FD Limit set: 1048576 is stupidly large.  Is this what you intended?  Consider using --limit-fds also limiting size to 100000
watchfrr[43]: [T83RR-8SM5G] watchfrr 10.0.1 starting: vty@0
watchfrr[43]: [ZCJ3S-SPH5S] zebra state -> down : initial connection attempt failed
watchfrr[43]: [ZCJ3S-SPH5S] mgmtd state -> down : initial connection attempt failed
watchfrr[43]: [ZCJ3S-SPH5S] staticd state -> down : initial connection attempt failed
watchfrr[43]: [YFT0P-5Q5YX] Forked background command [pid 44]: /usr/lib/frr/watchfrr.sh restart all
watchfrr[43]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
watchfrr[43]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
watchfrr[43]: [K54RW-4APGS][EC 268435457] startup did not complete within timeout (1/3 daemons running)
 * Started watchfrr
root@de0adeff7f28:/# watchfrr[43]: [YFT0P-5Q5YX] Forked background command [pid 83]: /usr/lib/frr/watchfrr.sh restart all
watchfrr[43]: [HD38Q-0HBRT][EC 268435457] staticd state -> down : read returned EOF
watchfrr[43]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
watchfrr[43]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded

root@df61a7f9ae1d:/# /etc/init.d/frr status
 * Status of watchfrr: running
 * Status of zebra: FAILED
 * Status of mgmtd: FAILED
 * Status of staticd: running
root@df61a7f9ae1d:/#
root@df61a7f9ae1d:/# vtysh

Hello, this is FRRouting (version 10.0.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

2024/08/03 14:03:01 [YDG3W-JND95] FD Limit set: 1048576 is stupidly large.  Is this what you intended?  Consider using --limit-fds also limiting size to 100000
df61a7f9ae1d#
df61a7f9ae1d# show ip route
zebra is not running
df61a7f9ae1d#
```
</details><br />

## Another failed attempt ... also for reference

This time, I am using this other `Dockerfile`

```bash
FROM debian:buster
MAINTAINER Rob Gil (rob@rem5.com)

ENV DEBIAN_FRONTEND noninteractive
ENV APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=DontWarn

RUN apt-get update && \
    apt-get install -y libpcre3-dev apt-transport-https ca-certificates curl wget logrotate \
    libc-ares2 libjson-c3 vim procps libreadline7 gnupg2 lsb-release apt-utils \
    libprotobuf-c-dev protobuf-c-compiler tini && rm -rf /var/lib/apt/lists/*

RUN curl -s https://deb.frrouting.org/frr/keys.asc | apt-key add -
RUN echo deb https://deb.frrouting.org/frr $(lsb_release -s -c) frr-stable | tee -a /etc/apt/sources.list.d/frr.list

RUN apt-get update && \
    apt-get install -y frr frr-pythontools && \
    rm -rf /var/lib/apt/lists/*

# Own the config / PID files
RUN mkdir -p /var/run/frr
RUN chown -R frr:frr /etc/frr /var/run/frr

# Simple init manager for reaping processes and forwarding signals
ENTRYPOINT ["/usr/bin/tini", "--"]

# Default CMD starts watchfrr
COPY --chmod=0755 docker-start /usr/lib/frr/docker-start
CMD ["/usr/lib/frr/docker-start"]
```

Building the image with it

```bash
docker build . -t frr_local_build_2
```

<details markdown=1>
<summary markdown="span">outputs</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ sudo docker build . -t frr_local_build_2
[+] Building 1.4s (12/12) FINISHED                                                                                                        docker:default
 => [internal] load build definition from Dockerfile                                                                                                0.0s
 => => transferring dockerfile: 1.07kB                                                                                                              0.0s
 => WARN: MaintainerDeprecated: Maintainer instruction is deprecated in favor of using label (line 2)                                               0.0s
 => [internal] load metadata for docker.io/library/debian:buster                                                                                    0.7s
 => [internal] load .dockerignore                                                                                                                   0.0s
 => => transferring context: 2B                                                                                                                     0.0s
 => CANCELED [1/8] FROM docker.io/library/debian:buster@sha256:58ce6f1271ae1c8a2006ff7d3e54e9874d839f573d8009c20154ad0f2fb0a225                     0.3s
 => => resolve docker.io/library/debian:buster@sha256:58ce6f1271ae1c8a2006ff7d3e54e9874d839f573d8009c20154ad0f2fb0a225                              0.1s
 => => sha256:2a0c1b9175adf759420fe0fbd7f5b449038319171eb76554bb76cbe172b62b42 529B / 529B                                                          0.0s
 => => sha256:69530eaa9e7e18d0aad40c38b75a22b40c6ebdc374c059bd5f2eb07042caa50a 1.46kB / 1.46kB                                                      0.0s
 => => sha256:58ce6f1271ae1c8a2006ff7d3e54e9874d839f573d8009c20154ad0f2fb0a225 984B / 984B                                                          0.0s
 => [internal] load build context                                                                                                                   0.2s
 => => transferring context: 2B                                                                                                                     0.0s
 => CACHED [2/8] RUN apt-get update &&     apt-get install -y libpcre3-dev apt-transport-https ca-certificates curl wget logrotate     libc-ares2   0.0s
 => CACHED [3/8] RUN curl -s https://deb.frrouting.org/frr/keys.asc | apt-key add -                                                                 0.0s
 => CACHED [4/8] RUN echo deb https://deb.frrouting.org/frr $(lsb_release -s -c) frr-stable | tee -a /etc/apt/sources.list.d/frr.list               0.0s
 => CACHED [5/8] RUN apt-get update &&     apt-get install -y frr frr-pythontools &&     rm -rf /var/lib/apt/lists/*                                0.0s
 => CACHED [6/8] RUN mkdir -p /var/run/frr                                                                                                          0.0s
 => CACHED [7/8] RUN chown -R frr:frr /etc/frr /var/run/frr                                                                                         0.0s
 => ERROR [8/8] COPY --chmod=0755 docker-start /usr/lib/frr/docker-start                                                                            0.0s
------
 > [8/8] COPY --chmod=0755 docker-start /usr/lib/frr/docker-start:
------

 2 warnings found (use --debug to expand):
 - MaintainerDeprecated: Maintainer instruction is deprecated in favor of using label (line 2)
 - LegacyKeyValueFormat: "ENV key=value" should be used instead of legacy "ENV key value" format (line 4)
Dockerfile:27
--------------------
  25 |
  26 |     # Default CMD starts watchfrr
  27 | >>> COPY --chmod=0755 docker-start /usr/lib/frr/docker-start
  28 |     CMD ["/usr/lib/frr/docker-start"]
  29 |
--------------------
ERROR: failed to solve: failed to compute cache key: failed to calculate checksum of ref 0a006a0d-fd20-4990-a1d7-148b600c604d::3jj4wza88d7vg4a09ll9b8oge: "/docker-start": not found
cloud_user@553b1e446c1c:~/installing_frr$
```
</details><br />

I should have the `docker-start` file created in the same directory.

Created `docker-start` file

```bash
#!/bin/bash

source /usr/lib/frr/frrcommon.sh
/usr/lib/frr/watchfrr $(daemon_list)
```

<details markdown=1>
<summary markdown="span">outputs</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ ll
total 16
drwxrwxr-x  2 cloud_user cloud_user 4096 Aug  3 14:21 ./
drwxr-x--- 19 cloud_user cloud_user 4096 Aug  3 14:21 ../
-rw-rw-r--  1 cloud_user cloud_user 1031 Aug  3 14:17 Dockerfile
-rw-rw-r--  1 cloud_user cloud_user   83 Aug  3 14:21 docker-start
cloud_user@553b1e446c1c:~/installing_frr$ cat docker-start
#!/bin/bash

source /usr/lib/frr/frrcommon.sh
/usr/lib/frr/watchfrr $(daemon_list)
cloud_user@553b1e446c1c:~/installing_frr$
```
</details><br />

Now, the build completed successfully this time

```bash
sudo docker build . -t frr_local_build_2
```

<details markdown=1>
<summary markdown="span">outputs</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ sudo docker build . -t frr_local_build_2
[+] Building 100.1s (13/13) FINISHED                                                                                                      docker:default
 => [internal] load build definition from Dockerfile                                                                                                0.0s
 => => transferring dockerfile: 1.07kB                                                                                                              0.0s
 => WARN: MaintainerDeprecated: Maintainer instruction is deprecated in favor of using label (line 2)                                               0.0s
 => [internal] load metadata for docker.io/library/debian:buster                                                                                    0.2s
 => [internal] load .dockerignore                                                                                                                   0.0s
 => => transferring context: 2B                                                                                                                     0.0s
 => [1/8] FROM docker.io/library/debian:buster@sha256:58ce6f1271ae1c8a2006ff7d3e54e9874d839f573d8009c20154ad0f2fb0a225                              5.2s
 => => resolve docker.io/library/debian:buster@sha256:58ce6f1271ae1c8a2006ff7d3e54e9874d839f573d8009c20154ad0f2fb0a225                              0.1s
 => => sha256:58ce6f1271ae1c8a2006ff7d3e54e9874d839f573d8009c20154ad0f2fb0a225 984B / 984B                                                          0.0s
 => => sha256:2a0c1b9175adf759420fe0fbd7f5b449038319171eb76554bb76cbe172b62b42 529B / 529B                                                          0.0s
 => => sha256:69530eaa9e7e18d0aad40c38b75a22b40c6ebdc374c059bd5f2eb07042caa50a 1.46kB / 1.46kB                                                      0.0s
 => => sha256:3892befd2c3f36ceb247ba7d906de12601d69b806597e65c4c837cf3d93df119 50.66MB / 50.66MB                                                    0.7s
 => => extracting sha256:3892befd2c3f36ceb247ba7d906de12601d69b806597e65c4c837cf3d93df119                                                           4.2s
 => [internal] load build context                                                                                                                   0.0s
 => => transferring context: 122B                                                                                                                   0.0s
 => [2/8] RUN apt-get update &&     apt-get install -y libpcre3-dev apt-transport-https ca-certificates curl wget logrotate     libc-ares2 libjso  72.7s
 => [3/8] RUN curl -s https://deb.frrouting.org/frr/keys.asc | apt-key add -                                                                        3.4s
 => [4/8] RUN echo deb https://deb.frrouting.org/frr $(lsb_release -s -c) frr-stable | tee -a /etc/apt/sources.list.d/frr.list                      0.8s
 => [5/8] RUN apt-get update &&     apt-get install -y frr frr-pythontools &&     rm -rf /var/lib/apt/lists/*                                      12.1s
 => [6/8] RUN mkdir -p /var/run/frr                                                                                                                 0.8s
 => [7/8] RUN chown -R frr:frr /etc/frr /var/run/frr                                                                                                0.7s
 => [8/8] COPY --chmod=0755 docker-start /usr/lib/frr/docker-start                                                                                  0.4s
 => exporting to image                                                                                                                              3.3s
 => => exporting layers                                                                                                                             3.2s
 => => writing image sha256:17bfebdb57fc038b29cd1c73724be3be58a0e3b64ea0919d8e83cd19580c7c24                                                        0.0s
 => => naming to docker.io/library/frr_local_build_2                                                                                                0.0s

 2 warnings found (use --debug to expand):
 - MaintainerDeprecated: Maintainer instruction is deprecated in favor of using label (line 2)
 - LegacyKeyValueFormat: "ENV key=value" should be used instead of legacy "ENV key value" format (line 4)
cloud_user@553b1e446c1c:~/installing_frr$
```
</details><br />

Ran a local container with the new Image

```bash
docker run -it --name router1 frr_local_build_2 bash
```

<details markdown=1>
<summary markdown="span">outputs</summary>

```bash
cloud_user@553b1e446c1c:~/installing_frr$ sudo docker run -it --name router1 frr_local_build_2 bash
root@fd632b7befd5:/# /etc/init.d/frr status
[FAIL] Status of watchfrr: FAILED ... failed!
[FAIL] Status of zebra: FAILED ... failed!
[FAIL] Status of mgmtd: FAILED ... failed!
[FAIL] Status of staticd: FAILED ... failed!
root@fd632b7befd5:/#
root@fd632b7befd5:/# /etc/init.d/frr start
[ ok ] Starting watchfrr with command: ' /usr/lib/frr/watchfrr -d -F traditional zebra mgmtd staticd'.
2024/08/03 14:31:30 WATCHFRR: [YDG3W-JND95] FD Limit set: 1048576 is stupidly large.  Is this what you intended?  Consider using --limit-fds also limiting size to 100000
watchfrr[120]: [T83RR-8SM5G] watchfrr 10.0.1 starting: vty@0
watchfrr[120]: [ZCJ3S-SPH5S] zebra state -> down : initial connection attempt failed
watchfrr[120]: [ZCJ3S-SPH5S] mgmtd state -> down : initial connection attempt failed
watchfrr[120]: [ZCJ3S-SPH5S] staticd state -> down : initial connection attempt failed
watchfrr[120]: [YFT0P-5Q5YX] Forked background command [pid 121]: /usr/lib/frr/watchfrr.sh restart all
watchfrr[120]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
watchfrr[120]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
watchfrr[120]: [K54RW-4APGS][EC 268435457] startup did not complete within timeout (1/3 daemons running)
[ ok ] Started watchfrr.
root@fd632b7befd5:/#
root@fd632b7befd5:/# /etc/init.d/frr status
[ ok ] Status of watchfrr: running.
[FAIL] Status of zebra: FAILED ... failed!
[FAIL] Status of mgmtd: FAILED ... failed!
[ ok ] Status of staticd: running.
root@fd632b7befd5:/#
```
</details><br />

But the same problem with `zebra` & `mgmtd` process failing to run

```bash
root@fd632b7befd5:/# /etc/init.d/frr status
[ ok ] Status of watchfrr: running.
[FAIL] Status of zebra: FAILED ... failed!
[FAIL] Status of mgmtd: FAILED ... failed!
[ ok ] Status of staticd: running.
root@fd632b7befd5:/#
```

## References

- [frrouting/frr image in DockerHub](https://hub.docker.com/r/frrouting/frr)
- [ksator/frrouting_demo in GitHub](https://github.com/ksator/frrouting_demo/blob/master/Dockerfile)
- [FRRouting/frr in GitHub](https://github.com/FRRouting/frr/blob/master/docker/debian/Dockerfile)
- [YouTube video: GNS3: FRRouting Using Docker Platform](https://www.youtube.com/watch?v=D4nk5VSUelg)
