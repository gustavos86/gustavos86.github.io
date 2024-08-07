---
title: Docker Storage
date: 2024-07-20 10:15:00 -0700
categories: [DOCKER, STORAGE]
tags: [docker]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

## Docker Filesystem

```bash
/var/lib/docker
 containers
 image
 volumes
 ...
```

<details markdown=1>
<summary markdown="span">ls -l /var/lib/docker</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo ls -l /var/lib/docker
total 44
drwx--x--x 4 root root 4096 Jul 20 17:40 buildkit
drwx--x--- 2 root root 4096 Jul 20 17:40 containers
-rw------- 1 root root   36 Jul 20 17:40 engine-id
drwx------ 3 root root 4096 Jul 20 17:40 image
drwxr-x--- 3 root root 4096 Jul 20 17:40 network
drwx--x--- 3 root root 4096 Jul 20 17:40 overlay2
drwx------ 4 root root 4096 Jul 20 17:40 plugins
drwx------ 2 root root 4096 Jul 20 17:40 runtimes
drwx------ 2 root root 4096 Jul 20 17:40 swarm
drwx------ 2 root root 4096 Jul 20 17:40 tmp
drwx-----x 2 root root 4096 Jul 20 17:40 volumes
ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

## docker info ##

Shows information about Docker installed in the system, including what storage drivers are in use, in this case the Storage driver is `overlay2`.

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

## Bind mount

Mounts a directory from any location in the Docker Host.

```bash
docker run -v </absoluth_path/volume_name>:<container_dictory_path>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ pwd
/home/cloud_user
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ mkdir mysql_volume
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ sudo docker run -e MYSQL_ROOT_PASSWORD=password -v /home/cloud_user/mysql_volume:/var/lib/mysql mysql
2024-07-20 21:07:28+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 9.0.0-1.el9 started.
2024-07-20 21:07:29+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2024-07-20 21:07:29+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 9.0.0-1.el9 started.
2024-07-20 21:07:29+00:00 [Note] [Entrypoint]: Initializing database files
2024-07-20T21:07:29.774325Z 0 [System] [MY-015017] [Server] MySQL Server Initialization - start.
2024-07-20T21:07:29.776789Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 9.0.0) initializing of server in progress as process 80
2024-07-20T21:07:29.800593Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2024-07-20T21:07:32.578377Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2024-07-20T21:07:40.032548Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
2024-07-20T21:07:47.869512Z 0 [System] [MY-015018] [Server] MySQL Server Initialization - end.
2024-07-20 21:07:47+00:00 [Note] [Entrypoint]: Database files initialized
2024-07-20 21:07:47+00:00 [Note] [Entrypoint]: Starting temporary server
2024-07-20T21:07:47.955602Z 0 [System] [MY-015015] [Server] MySQL Server - start.
2024-07-20T21:07:48.284168Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 9.0.0) starting as process 119
2024-07-20T21:07:48.320234Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2024-07-20T21:07:54.156494Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2024-07-20T21:07:58.363586Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2024-07-20T21:07:58.363657Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2024-07-20T21:07:58.385186Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2024-07-20T21:07:59.034222Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: /var/run/mysqld/mysqlx.sock
2024-07-20T21:07:59.034762Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '9.0.0'  socket: '/var/run/mysqld/mysqld.sock'  port: 0  MySQL Community Server - GPL.
2024-07-20 21:07:59+00:00 [Note] [Entrypoint]: Temporary server started.
'/var/lib/mysql/mysql.sock' -> '/var/run/mysqld/mysqld.sock'
Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/leap-seconds.list' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/leapseconds' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/tzdata.zi' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone1970.tab' as time zone. Skipping it.

2024-07-20 21:08:10+00:00 [Note] [Entrypoint]: Stopping temporary server
2024-07-20T21:08:10.654202Z 10 [System] [MY-013172] [Server] Received SHUTDOWN from user root. Shutting down mysqld (Version: 9.0.0).
2024-07-20T21:08:12.337981Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 9.0.0)  MySQL Community Server - GPL.
2024-07-20T21:08:12.338034Z 0 [System] [MY-015016] [Server] MySQL Server - end.
2024-07-20 21:08:12+00:00 [Note] [Entrypoint]: Temporary server stopped

2024-07-20 21:08:12+00:00 [Note] [Entrypoint]: MySQL init process done. Ready for start up.

2024-07-20T21:08:12.692265Z 0 [System] [MY-015015] [Server] MySQL Server - start.
2024-07-20T21:08:13.013808Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 9.0.0) starting as process 1
2024-07-20T21:08:13.024238Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2024-07-20T21:08:19.841892Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2024-07-20T21:08:23.915437Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2024-07-20T21:08:23.915981Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2024-07-20T21:08:23.939262Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2024-07-20T21:08:25.718165Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
2024-07-20T21:08:25.718373Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '9.0.0'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
^Ccontext canceled
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                 NAMES
8e718f923c27   mysql     "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   3306/tcp, 33060/tcp   vibrant_hawking
cloud_user@553b1e446c1c:~$
cloud_user@553b1e446c1c:~$ sudo docker inspect 8e718f923c27
[
    {
        ...
        "Driver": "overlay2",
        ...
        "HostConfig": {
            "Binds": [
                "/home/cloud_user/mysql_volume:/var/lib/mysql"
            ],
        ...
        },
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/home/cloud_user/mysql_volume",
                "Destination": "/var/lib/mysql",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
        "Config": {
            ...
            "Env": [
                "MYSQL_ROOT_PASSWORD=password",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "GOSU_VERSION=1.17",
                "MYSQL_MAJOR=innovation",
                "MYSQL_VERSION=9.0.0-1.el9",
                "MYSQL_SHELL_VERSION=9.0.0-1.el9"
            ],
            ...,
            "Volumes": {
                "/var/lib/mysql": {}
            },
            ...
        },
        ...
    }
]
cloud_user@553b1e446c1c:~$
```
</details><br />

## --mount

`--mount` will be the way forward as opposed to `-v`

```bash
docker run \
 --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo mkdir -p /data/mysql
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ sudo docker run \
 -d \
 -e MYSQL_ROOT_PASSWORD=password \
 --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
3e3b199226f57d2329f611d456b08352517b17e34fd4f5ab6c4655fa0b760b11
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ sudo docker inspect 3e3b199
[
    {
    ...
        "HostConfig": {
            "Binds": null,
            ...
            "Mounts": [
                {
                    "Type": "bind",
                    "Source": "/data/mysql",
                    "Target": "/var/lib/mysql"
                }
            ],
            ...
        },
        ...
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/data/mysql",
                "Destination": "/var/lib/mysql",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],

        "Config": {
            ...
            "Volumes": {
                "/var/lib/mysql": {}
            },
            ...
        },
        ...
    }
]
cloud_user@553b1e446c1c:~$
```
</details><br />

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

## Different Storage drivers for Docker

Examples:

- AUFS
- ZFS
- BTRFS
- Device Mapper
- Oerlay
- Overlay2

## References

- [Use the OverlayFS storage driver](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)
