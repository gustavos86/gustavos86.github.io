---
title: Docker Registry container
date: 2024-07-18 06:00:00 -0700
categories: [DOCKER, REGISTRY]
tags: [docker]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

## Docker Registry

There is a container simply called **registry** stored in Docker Hub that you can use to set-up your own private Repository.
Here, we are running it in `localhost`. However, it can of course be run in a remote server.

![]({{ site.baseurl }}/images/2024/07-18-Docker-Registry-container/01-Docker-registry-in-Docker-Hub.png)

For a quick start, run the **registry** image

```bash
sudo docker run -d -p 5000:5000 --name registry registry:2
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo docker run -d -p 5000:5000 --name registry registry:2
42c3da90c8503aeca23f2f850682c23fe53486dd6414540f7c9b49da44848b70
cloud_user@553b1e446c1c:~$ 
cloud_user@553b1e446c1c:~$ sudo docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                                       NAMES
42c3da90c850   registry:2   "/entrypoint.sh /etc…"   2 minutes ago   Up 2 minutes   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   registry
cloud_user@553b1e446c1c:~$ 
```
</details><br />

Next, tag the Docker image properly

```bash
sudo docker image tag cloud_user-vote localhost:5000/cloud_user-vote
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo docker images
REPOSITORY                                                     TAG       IMAGE ID       CREATED        SIZE
cloud_user-vote                                                latest    3cd62f1d4c2c   10 hours ago   153MB
cloud_user@553b1e446c1c:~$ 

cloud_user@553b1e446c1c:~$ sudo docker image tag cloud_user-vote localhost:5000/cloud_user-vote
cloud_user@553b1e446c1c:~$ 

cloud_user@553b1e446c1c:~$ sudo docker images
REPOSITORY                                                     TAG       IMAGE ID       CREATED        SIZE
cloud_user-vote                                                latest    3cd62f1d4c2c   10 hours ago   153MB
localhost:5000/cloud_user-vote                                 latest    3cd62f1d4c2c   10 hours ago   153MB
cloud_user@553b1e446c1c:~$ 
```
</details><br />

Finally, push the images to `registry`

```bash
docker push localhost:5000/cloud_user-vote
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo docker push localhost:5000/cloud_user-vote
Using default tag: latest
The push refers to repository [localhost:5000/cloud_user-vote]
4ef1a5d1719d: Pushed 
3c128b45678a: Pushed 
f7dab6e3ed7a: Pushed 
264d4062512d: Pushed 
8216ecb6ac16: Pushed 
5c792cb82821: Pushed 
d1281f9883d7: Pushed 
5756a972e734: Pushed 
67e13e951fda: Pushed 
32148f9f6c5a: Pushed 
latest: digest: sha256:d791968855a2df93696729a39c671e4318b98cc9a425aa086336335d6a47eee9 size: 2414
cloud_user@553b1e446c1c:~$ 
```
</details><br />

## References

- https://hub.docker.com/_/registry