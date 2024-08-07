---
title: Docker Host vs Container processes
date: 2024-07-19 15:30:00 -0700
categories: [DOCKER]
tags: [docker]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

Linux Process ID in Container is **1** which is the main Process. This same PID in the Docker Host is **PID X**. This is a fundamental part for the isolation that containers provide.

Here a bit of the proof:

## Get PID 1 in the Docker Host

**PID 1** is assigned to `/sbin/init`

```bash
ps -eaf
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ ps -eaf
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 22:33 ?        00:00:04 /sbin/init
...
```
</details><br />

## Starting a Container

```bash
docker run -d --rm -p 8888:8080 tomcat:8.0
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker run -d --rm -p 8888:8080 tomcat:8.0
Unable to find image 'tomcat:8.0' locally
8.0: Pulling from library/tomcat
f189db1b88b3: Pull complete 
3d06cf2f1b5e: Pull complete 
edd0da9e3091: Pull complete 
eb7768aae14e: Pull complete 
e2780f585e0f: Pull complete 
e5ed720afeba: Pull complete 
d9e134700cfc: Pull complete 
e4804b33d02a: Pull complete 
b9df0c24315e: Pull complete 
49fdae8eaa20: Pull complete 
1aea3d9a32e6: Pull complete 
Digest: sha256:8ecb10948deb32c34aeadf7bf95d12a93fbd3527911fa629c1a3e7823b89ce6f
Status: Downloaded newer image for tomcat:8.0
6f513d1ae9990846c7516ccd437f86a86e4f48ca1b2126cabd3db0d8a015c30e
ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

## Get PID 1 in the Container

```bash
docker exec <container_id> ps -ef
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker ps
CONTAINER ID   IMAGE        COMMAND             CREATED         STATUS         PORTS                                       NAMES
6f513d1ae999   tomcat:8.0   "catalina.sh run"   2 minutes ago   Up 2 minutes   0.0.0.0:8888->8080/tcp, :::8888->8080/tcp   nostalgic_ganguly
ubuntu@ip-XXX-XX-XX-XXX:~$ 

ubuntu@ip-XXX-XX-XX-XXX:~$ sudo docker exec 6f513d1 ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  3 23:06 ?        00:00:03 /docker-java-home/jre/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dignore.endorsed.dirs= -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat -Djava.io.tmpdir=/usr/local/tomcat/temp org.apache.catalina.startup.Bootstrap start
root          51       0  0 23:08 ?        00:00:00 ps -ef
ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

We see from this output that **PID 1** in the Tomcat container is running the process `/docker-java-home/jre/bin/java`

## Locate this same process in the Docker Host

If we look for this `/docker-java-home/jre/bin/java` process in the Docker Host, we will find it with a **PID** othen than **1** of course.

```bash
ps -eaf | grep /docker-java-home/jre/bin/java
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ ps -eaf | grep /docker-java-home/jre/bin/java
root        2964    2945  0 23:06 ?        00:00:04 /docker-java-home/jre/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dignore.endorsed.dirs= -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat -Djava.io.tmpdir=/usr/local/tomcat/temp org.apache.catalina.startup.Bootstrap start
```
</details><br />

As a conclusion, in the Docker Host we see the process `/docker-java-home/jre/bin/java` as **PID 2964** while in the Containter it is the running as **PID 1**