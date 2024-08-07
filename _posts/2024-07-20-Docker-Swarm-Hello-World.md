---
title: Docker Swarm Hello World
date: 2024-07-20 18:30:00 -0700
categories: [DOCKER, SWARM]
tags: [docker]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

## Pre-requisites

- Have Docker installed on the Docker Manager Node and all Worker Nodes
  - Refer to [Install Docker Engine on Ubuntu]({% post_url 2024-07-16-Docker-installing-ce %})
- Have the following ports opened:
  - Port `2377 TCP` for communication with and between manager nodes
  - Port `7946 TCP/UDP` for overlay network node discovery
  - Port `4789 UDP` (configurable) for overlay network traffic

## EC2 instances with Security Groups

I am using AWS to run four EC2 instances:
  - 1x Manager Node
  - 3x Worker Nodes

For the purpose of this lab:
Only the **Manager Node** has a Public IP address so I can use it as a jump host to SSH into the Worker Nodes.
**Worked Nodes** are in a Private Subnet and reach the internet via a **NAT Gateway**.

![]({{ site.baseurl }}/images/2024/07-20-Docker-Swarm-Hello-World/01-EC2-Instances.png)

The Security Group shared by all four EC2 Instances has all ports specified in the Docker's documentation opened.

![]({{ site.baseurl }}/images/2024/07-20-Docker-Swarm-Hello-World/02-Security-Group.png)

## Manager Node

```bash
ubuntu@ip-172-31-82-124:~$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enX0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 12:b2:c8:54:ab:ad brd ff:ff:ff:ff:ff:ff
    inet 172.31.82.124/20 metric 100 brd 172.31.95.255 scope global dynamic enX0
       valid_lft 2243sec preferred_lft 2243sec
    inet6 fe80::10b2:c8ff:fe54:abad/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:2d:ce:12:ec brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
ubuntu@ip-172-31-82-124:~$
```

Create a Docker Swarm tocken with

```bash
docker swarm init --advertise-addr <local_ip_address>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-82-124:~$ sudo docker swarm init --advertise-addr 172.31.82.124
Swarm initialized: current node (j09tqijqldqg1idr2nzg7iop7) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0q19ibou18j3vddb1ft30zu3453811gfscbjm3fo4du44vtlzm-37w7unle4vfgyzx5g7r2lcd8t 172.31.82.124:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

ubuntu@ip-172-31-82-124:~$
```
</details><br />

## Worker Node

Use the command specified in the Manager Node to join it.

```bash
docker swarm join --token <token> HOST:PORT
```

<details markdown=1>
<summary markdown="span">worker_node1</summary>

```bash
ubuntu@ip-172-31-98-224:~$ sudo docker swarm join --token SWMTKN-1-0q19ibou18j3vddb1ft30zu3453811gfscbjm3fo4du44vtlzm-37w7unle4vfgyzx5g7r2lcd8t 172.31.82.124:2377
This node joined a swarm as a worker.
ubuntu@ip-172-31-98-224:~$
```
</details><br />

<details markdown=1>
<summary markdown="span">worker_node2</summary>

```bash
ubuntu@ip-172-31-99-253:~$ sudo docker swarm join --token SWMTKN-1-0q19ibou18j3vddb1ft30zu3453811gfscbjm3fo4du44vtlzm-37w7unle4vfgyzx5g7r2lcd8t 172.31.82.124:2377
This node joined a swarm as a worker.
ubuntu@ip-172-31-99-253:~$
```
</details><br />


<details markdown=1>
<summary markdown="span">worker_node3</summary>

```bash
ubuntu@ip-172-31-97-46:~$ sudo docker swarm join --token SWMTKN-1-0q19ibou18j3vddb1ft30zu3453811gfscbjm3fo4du44vtlzm-37w7unle4vfgyzx5g7r2lcd8t 172.31.82.124:2377
This node joined a swarm as a worker.
ubuntu@ip-172-31-97-46:~$
```
</details><br />

## List nodes

```bash
docker node ls
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-82-124:~$ sudo docker node ls
ID                            HOSTNAME           STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
j09tqijqldqg1idr2nzg7iop7 *   ip-172-31-82-124   Ready     Active         Leader           27.0.3
i1yjurovqbb3yumfrtk7q9htr     ip-172-31-97-46    Ready     Active                          27.0.3
vlyx9w5sfyau9zlq7v8sxnubr     ip-172-31-98-224   Ready     Active                          27.0.3
wv8117fw3w3z9g0d19zgetjns     ip-172-31-99-253   Ready     Active                          27.0.3
ubuntu@ip-172-31-82-124:~$
```
</details><br />

## Inspect an individual node

To view the details for an individual node

```bash
docker node inspect
```

<details markdown=1>
<summary markdown="span">manager_node</summary>

```bash
ubuntu@ip-172-31-82-124:~$ sudo docker node inspect self --pretty
ID:			j09tqijqldqg1idr2nzg7iop7
Hostname:              	ip-172-31-82-124
Joined at:             	2024-07-21 02:10:45.901033975 +0000 utc
Status:
 State:			Ready
 Availability:         	Active
 Address:		172.31.82.124
Manager Status:
 Address:		172.31.82.124:2377
 Raft Status:		Reachable
 Leader:		Yes
Platform:
 Operating System:	linux
 Architecture:		x86_64
Resources:
 CPUs:			1
 Memory:		957.4MiB
Plugins:
 Log:		awslogs, fluentd, gcplogs, gelf, journald, json-file, local, splunk, syslog
 Network:		bridge, host, ipvlan, macvlan, null, overlay
 Volume:		local
Engine Version:		27.0.3
TLS Info:
 TrustRoot:
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----

 Issuer Subject:	MBMxETAPBgNVBAMTCHN3YXJtLWNh
 Issuer Public Key:	MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEb+UZDq3lMLMpId+bC8sHVHlQcZHFfm9SbNAnAPfcCtTZIaq++f0Rub2GkYlwW0dn7Fs0SDJROZSFf3jsxpgPsg==
ubuntu@ip-172-31-82-124:~$
```
</details><br />

<details markdown=1>
<summary markdown="span">worker_node1</summary>

```bash
ubuntu@ip-172-31-82-124:~$ sudo docker node inspect ip-172-31-98-224 --pretty
ID:			vlyx9w5sfyau9zlq7v8sxnubr
Hostname:              	ip-172-31-98-224
Joined at:             	2024-07-21 02:27:18.771063436 +0000 utc
Status:
 State:			Ready
 Availability:         	Active
 Address:		172.31.98.224
Platform:
 Operating System:	linux
 Architecture:		x86_64
Resources:
 CPUs:			1
 Memory:		957.4MiB
Plugins:
 Log:		awslogs, fluentd, gcplogs, gelf, journald, json-file, local, splunk, syslog
 Network:		bridge, host, ipvlan, macvlan, null, overlay
 Volume:		local
Engine Version:		27.0.3
TLS Info:
 TrustRoot:
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----

 Issuer Subject:	MBMxETAPBgNVBAMTCHN3YXJtLWNh
 Issuer Public Key:	MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEb+UZDq3lMLMpId+bC8sHVHlQcZHFfm9SbNAnAPfcCtTZIaq++f0Rub2GkYlwW0dn7Fs0SDJROZSFf3jsxpgPsg==
ubuntu@ip-172-31-82-124:~$
```
</details><br />

<details markdown=1>
<summary markdown="span">worker_node2</summary>

```bash
ubuntu@ip-172-31-82-124:~$ sudo docker node inspect ip-172-31-99-253 --pretty
ID:			wv8117fw3w3z9g0d19zgetjns
Hostname:              	ip-172-31-99-253
Joined at:             	2024-07-21 02:27:24.510108939 +0000 utc
Status:
 State:			Ready
 Availability:         	Active
 Address:		172.31.99.253
Platform:
 Operating System:	linux
 Architecture:		x86_64
Resources:
 CPUs:			1
 Memory:		957.4MiB
Plugins:
 Log:		awslogs, fluentd, gcplogs, gelf, journald, json-file, local, splunk, syslog
 Network:		bridge, host, ipvlan, macvlan, null, overlay
 Volume:		local
Engine Version:		27.0.3
TLS Info:
 TrustRoot:
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----

 Issuer Subject:	MBMxETAPBgNVBAMTCHN3YXJtLWNh
 Issuer Public Key:	MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEb+UZDq3lMLMpId+bC8sHVHlQcZHFfm9SbNAnAPfcCtTZIaq++f0Rub2GkYlwW0dn7Fs0SDJROZSFf3jsxpgPsg==
ubuntu@ip-172-31-82-124:~$
```
</details><br />

<details markdown=1>
<summary markdown="span">worker_node3</summary>

```bash
ubuntu@ip-172-31-82-124:~$ sudo docker node inspect ip-172-31-97-46 --pretty
ID:			i1yjurovqbb3yumfrtk7q9htr
Hostname:              	ip-172-31-97-46
Joined at:             	2024-07-21 02:27:26.358931469 +0000 utc
Status:
 State:			Ready
 Availability:         	Active
 Address:		172.31.97.46
Platform:
 Operating System:	linux
 Architecture:		x86_64
Resources:
 CPUs:			1
 Memory:		957.4MiB
Plugins:
 Log:		awslogs, fluentd, gcplogs, gelf, journald, json-file, local, splunk, syslog
 Network:		bridge, host, ipvlan, macvlan, null, overlay
 Volume:		local
Engine Version:		27.0.3
TLS Info:
 TrustRoot:
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----

 Issuer Subject:	MBMxETAPBgNVBAMTCHN3YXJtLWNh
 Issuer Public Key:	MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEb+UZDq3lMLMpId+bC8sHVHlQcZHFfm9SbNAnAPfcCtTZIaq++f0Rub2GkYlwW0dn7Fs0SDJROZSFf3jsxpgPsg==
ubuntu@ip-172-31-82-124:~$
```
</details><br />

## Creating containers in the Swarm

This will create several replicast of the image in the Nodes.

```bash
docker service create \
  --replicas=X \
  <image_name>
```

**NOTE:** Getting error **No such image**. There is be a problem here. Maybe related to having the Worker Nodes behind the AWS NAT Gateway but it shouldn't. Will troubleshoot later.

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-82-124:~$ sudo docker service create   --replicas=9   hello_world
image hello_world:latest could not be accessed on a registry to record
its digest. Each node will access hello_world:latest independently,
possibly leading to different nodes running different
versions of the image.

rodod65q8u464ba15zu1yuphn
overall progress: 0 out of 9 tasks
1/9: No such image: hello_world:latest
2/9: No such image: hello_world:latest
3/9: No such image: hello_world:latest
4/9: No such image: hello_world:latest
5/9: No such image: hello_world:latest
6/9: No such image: hello_world:latest
7/9: No such image: hello_world:latest
8/9: No such image: hello_world:latest
9/9: No such image: hello_world:latest
^COperation continuing in background.
Use `docker service ps rodod65q8u464ba15zu1yuphn` to check progress.
ubuntu@ip-172-31-82-124:~$
```
</details><br />

## See the services running on the Swarm

On the Manager Node

```bash
docker service ls
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-82-124:~$ sudo docker service ls
ID             NAME               MODE         REPLICAS   IMAGE                PORTS
nyv2olropkkm   relaxed_rhodes     replicated   0/9        hello_world:latest
hhu75oqsxfsu   unruffled_hoover   replicated   0/9        hello_world:latest
ubuntu@ip-172-31-82-124:~$
```
</details><br />

## Stop services on the Swarm

On the Manager Node

```bash
docker service rm
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-82-124:~$ sudo docker service rm nyv2olropkkm hhu75oqsxfsu
nyv2olropkkm
hhu75oqsxfsu
ubuntu@ip-172-31-82-124:~$
```
</details><br />

## References

- [Getting started with Swarm mode > Open protocols and ports between the hosts](https://docs.docker.com/engine/swarm/swarm-tutorial/#open-protocols-and-ports-between-the-hosts)
- [Manage nodes in a swarm](https://docs.docker.com/engine/swarm/manage-nodes/)
