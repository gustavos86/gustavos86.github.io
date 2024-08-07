---
title: Docker Compose
date: 2024-07-17 19:30:00 -0700
categories: [DOCKER, COMPOSE]
tags: [docker]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

## TLDR;

Just copy and paste this to have Docker Compose installed

```bash
sudo curl -SL https://github.com/docker/compose/releases/download/v2.29.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## Install Docker Compose

```bash
sudo curl -SL https://github.com/docker/compose/releases/download/v2.29.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo curl -SL https://github.com/docker/compose/releases/download/v2.29.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 60.2M  100 60.2M    0     0  65.1M      0 --:--:-- --:--:-- --:--:-- 65.1M
cloud_user@553b1e446c1c:~$
```
</details><br />

Apply executable permissions to the binary

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo chmod +x /usr/local/bin/docker-compose
cloud_user@553b1e446c1c:~$ 
```
</details><br />

Docker compose should be installed by now. Verify with the command:

```bash
docker-compose
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ docker-compose

Usage:  docker compose [OPTIONS] COMMAND

Define and run multi-container applications with Docker

Options:
      --all-resources              Include all resources, even those not used by services
      --ansi string                Control when to print ANSI control characters ("never"|"always"|"auto") (default "auto")
      --compatibility              Run compose in backward compatibility mode
      --dry-run                    Execute command in dry run mode
      --env-file stringArray       Specify an alternate environment file
  -f, --file stringArray           Compose configuration files
      --parallel int               Control max parallelism, -1 for unlimited (default -1)
      --profile stringArray        Specify a profile to enable
      --progress string            Set type of progress output (auto, tty, plain, json, quiet) (default "auto")
      --project-directory string   Specify an alternate working directory
                                   (default: the path of the, first specified, Compose file)
  -p, --project-name string        Project name

Commands:
  attach      Attach local standard input, output, and error streams to a service's running container
  build       Build or rebuild services
  config      Parse, resolve and render compose file in canonical format
  cp          Copy files/folders between a service container and the local filesystem
  create      Creates containers for a service
  down        Stop and remove containers, networks
  events      Receive real time events from containers
  exec        Execute a command in a running container
  images      List images used by the created containers
  kill        Force stop service containers
  logs        View output from containers
  ls          List running compose projects
  pause       Pause services
  port        Print the public port for a port binding
  ps          List containers
  pull        Pull service images
  push        Push service images
  restart     Restart service containers
  rm          Removes stopped service containers
  run         Run a one-off command on a service
  scale       Scale services 
  start       Start services
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop services
  top         Display the running processes
  unpause     Unpause services
  up          Create and start containers
  version     Show the Docker Compose version information
  wait        Block until the first service container stops
  watch       Watch build context for service and rebuild/refresh containers when files are updated

Run 'docker compose COMMAND --help' for more information on a command.
cloud_user@553b1e446c1c:~$ 
```
</details><br />

## More than one compose file in the directory

There are cases when there is more than a single compose file, for example in this GitHub repo (ksator / frrouting_demo GitHub repo)[https://github.com/ksator/frrouting_demo/tree/master]. In this case we can choose which compose file to use with the `-f` flag. 

```bash
cloud_user@553b1e446c1c:~/ksator_frrouting_demo/frrouting_demo$ tree
.
├── Dockerfile
├── README.md
...
├── docker-compose-demo1.yml
└── docker-compose-demo2.yml
```

Example:

```bash
docker-compose -f docker-compose-demo1.yml up
```

or 

```bash
docker-compose -f docker-compose-demo2.yml up
```

## Run Compose in detached mode

Use the `-d` flag. Example:

```bash
docker-compose -f docker-compose-demo1.yml up -d
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/ksator_frrouting_demo/frrouting_demo$ docker-compose -f docker-compose-demo1.yml up -d
WARN[0000] /home/cloud_user/ksator_frrouting_demo/frrouting_demo/docker-compose-demo1.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
[+] Running 2/2
 ✔ Container frr100  Started                                                                                                                                                                           0.9s
 ✔ Container frr200  Started                                                                                                                                                                           0.8s
cloud_user@553b1e446c1c:~/ksator_frrouting_demo/frrouting_demo$
```
</details><br />


## Clone the example-voting-app using Git

We will be using the voting-app to test Docker Compose functionality.

The code is stored in GitHub and the architecture is as follows:

![]({{ site.baseurl }}/images/2024/07-17-Docker-Compose/architecture.excalidraw.png)

Next, clone the repo:

```bash
git clone https://github.com/dockersamples/example-voting-app
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ git clone https://github.com/dockersamples/example-voting-app
Cloning into 'example-voting-app'...
remote: Enumerating objects: 1140, done.
remote: Total 1140 (delta 0), reused 0 (delta 0), pack-reused 1140
Receiving objects: 100% (1140/1140), 1.19 MiB | 10.71 MiB/s, done.
Resolving deltas: 100% (438/438), done.
cloud_user@553b1e446c1c:~$ 
```
</details><br />

## Create Docker Compose YAML file

- `image` is used to pull the image from Docker Hub
- `build` is used to create the image locally using a Dockerfile. This is in the GitHub repo we cloned
- `environment` need to be passed to postgres, otherwise it shows an error

`docker-compose.yml`

```yaml
services:
  redis:
    image: redis
  
  db:
    image: postgres:9.4
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
  
  vote:
    build: ./example-voting-app/vote
    ports:
      - 5000:80
    links:
      - redis
  
  worker:
    build: ./example-voting-app/worker
    links:
      - db
      - redis
  
  result:
    build: ./example-voting-app/result
    ports:
      - 5001:80
    links:
      - db
```

**NOTE:** There is another docker compose YAML file in the repo with more advance options. We are using this one for simplicity.
**NOTE:** `links` may be removed from this YAML file

Once the YAML file has been created, run:

```bash
docker-compose up
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo docker-compose up
[+] Running 5/5
 ✔ Container cloud_user-redis-1   Created                                                                                                                                                                                                                                                                0.0s 
 ✔ Container cloud_user-db-1      Recreated                                                                                                                                                                                                                                                              0.4s 
 ✔ Container cloud_user-vote-1    Created                                                                                                                                                                                                                                                                0.0s 
 ✔ Container cloud_user-result-1  Recreated                                                                                                                                                                                                                                                              0.4s 
 ✔ Container cloud_user-worker-1  Recreated                                                                                                                                                                                                                                                              0.4s 
Attaching to db-1, redis-1, result-1, vote-1, worker-1
redis-1   | 1:C 18 Jul 2024 03:43:17.371 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
redis-1   | 1:C 18 Jul 2024 03:43:17.372 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis-1   | 1:C 18 Jul 2024 03:43:17.373 * Redis version=7.2.5, bits=64, commit=00000000, modified=0, pid=1, just started
redis-1   | 1:C 18 Jul 2024 03:43:17.373 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis-1   | 1:M 18 Jul 2024 03:43:17.375 * monotonic clock: POSIX clock_gettime
redis-1   | 1:M 18 Jul 2024 03:43:17.377 * Running mode=standalone, port=6379.
redis-1   | 1:M 18 Jul 2024 03:43:17.378 * Server initialized
redis-1   | 1:M 18 Jul 2024 03:43:17.379 * Loading RDB produced by version 7.2.5
redis-1   | 1:M 18 Jul 2024 03:43:17.379 * RDB age 99 seconds
redis-1   | 1:M 18 Jul 2024 03:43:17.379 * RDB memory usage when created 0.83 Mb
redis-1   | 1:M 18 Jul 2024 03:43:17.379 * Done loading RDB, keys loaded: 0, keys expired: 0.
redis-1   | 1:M 18 Jul 2024 03:43:17.379 * DB loaded from disk: 0.001 seconds
redis-1   | 1:M 18 Jul 2024 03:43:17.380 * Ready to accept connections tcp
vote-1    | [2024-07-18 03:43:18 +0000] [1] [INFO] Starting gunicorn 22.0.0
vote-1    | [2024-07-18 03:43:18 +0000] [1] [INFO] Listening at: http://0.0.0.0:80 (1)
vote-1    | [2024-07-18 03:43:18 +0000] [1] [INFO] Using worker: sync
vote-1    | [2024-07-18 03:43:18 +0000] [7] [INFO] Booting worker with pid: 7
vote-1    | [2024-07-18 03:43:18 +0000] [8] [INFO] Booting worker with pid: 8
vote-1    | [2024-07-18 03:43:18 +0000] [9] [INFO] Booting worker with pid: 9
result-1  | Thu, 18 Jul 2024 03:43:18 GMT body-parser deprecated undefined extended: provide extended option at server.js:67:17
result-1  | App running on port 80
result-1  | Waiting for db
vote-1    | [2024-07-18 03:43:18 +0000] [10] [INFO] Booting worker with pid: 10
worker-1  | Waiting for db
result-1  | Waiting for db
worker-1  | Waiting for db
db-1      | The files belonging to this database system will be owned by user "postgres".
db-1      | This user must also own the server process.
db-1      | 
db-1      | The database cluster will be initialized with locale "en_US.utf8".
db-1      | The default database encoding has accordingly been set to "UTF8".
db-1      | The default text search configuration will be set to "english".
db-1      | 
db-1      | Data page checksums are disabled.
db-1      | 
db-1      | fixing permissions on existing directory /var/lib/postgresql/data ... ok
db-1      | creating subdirectories ... ok
result-1  | Waiting for db
db-1      | selecting default max_connections ... 100
db-1      | selecting default shared_buffers ... 128MB
db-1      | selecting default timezone ... Etc/UTC
db-1      | selecting dynamic shared memory implementation ... posix
db-1      | creating configuration files ... ok
worker-1  | Waiting for db
result-1  | Waiting for db
db-1      | creating template1 database in /var/lib/postgresql/data/base/1 ... ok
db-1      | initializing pg_authid ... ok
db-1      | setting password ... ok
db-1      | initializing dependencies ... ok
worker-1  | Waiting for db
db-1      | creating system views ... ok
db-1      | loading system objects' descriptions ... ok
result-1  | Waiting for db
db-1      | creating collations ... ok
worker-1  | Waiting for db
db-1      | creating conversions ... ok
result-1  | Waiting for db
db-1      | creating dictionaries ... ok
db-1      | setting privileges on built-in objects ... ok
worker-1  | Waiting for db
result-1  | Waiting for db
db-1      | creating information schema ... ok
db-1      | loading PL/pgSQL server-side language ... ok
db-1      | vacuuming database template1 ... ok
db-1      | copying template1 to template0 ... ok
worker-1  | Waiting for db
db-1      | copying template1 to postgres ... ok
result-1  | Waiting for db
db-1      | syncing data to disk ... ok
db-1      | 
db-1      | Success. You can now start the database server using:
db-1      | 
db-1      |     postgres -D /var/lib/postgresql/data
db-1      | or
db-1      |     pg_ctl -D /var/lib/postgresql/data -l logfile start
db-1      | 
db-1      | 
db-1      | WARNING: enabling "trust" authentication for local connections
db-1      | You can change this by editing pg_hba.conf or using the option -A, or
db-1      | --auth-local and --auth-host, the next time you run initdb.
db-1      | waiting for server to start....LOG:  database system was shut down at 2024-07-18 03:43:25 UTC
db-1      | LOG:  MultiXact member wraparound protections are now enabled
db-1      | LOG:  autovacuum launcher started
db-1      | LOG:  database system is ready to accept connections
worker-1  | Waiting for db
result-1  | Waiting for db
db-1      |  done
db-1      | server started
db-1      | 
db-1      | /usr/local/bin/docker-entrypoint.sh: ignoring /docker-entrypoint-initdb.d/*
db-1      | 
db-1      | LOG:  received fast shutdown request
db-1      | waiting for server to shut down....LOG:  aborting any active transactions
db-1      | LOG:  autovacuum launcher shutting down
db-1      | LOG:  shutting down
db-1      | LOG:  database system is shut down
worker-1  | Waiting for db
result-1  | Waiting for db
db-1      |  done
db-1      | server stopped
db-1      | 
db-1      | PostgreSQL init process complete; ready for start up.
db-1      | 
db-1      | LOG:  database system was shut down at 2024-07-18 03:43:27 UTC
db-1      | LOG:  MultiXact member wraparound protections are now enabled
db-1      | LOG:  autovacuum launcher started
db-1      | LOG:  database system is ready to accept connections
result-1  | Connected to db
db-1      | ERROR:  relation "votes" does not exist at character 38
db-1      | STATEMENT:  SELECT vote, COUNT(id) AS count FROM votes GROUP BY vote
result-1  | Error performing query: error: relation "votes" does not exist
worker-1  | Connected to db
db-1      | ERROR:  relation "votes" does not exist at character 38
db-1      | STATEMENT:  SELECT vote, COUNT(id) AS count FROM votes GROUP BY vote
result-1  | Error performing query: error: relation "votes" does not exist
worker-1  | Connecting to redis
worker-1  | Found redis at 172.18.0.3

cloud_user@553b1e446c1c:~$ sudo docker ps
CONTAINER ID   IMAGE               COMMAND                  CREATED          STATUS          PORTS                                   NAMES
8266d3f1912b   cloud_user-result   "/usr/bin/tini -- no…"   20 minutes ago   Up 20 minutes   0.0.0.0:5001->80/tcp, :::5001->80/tcp   cloud_user-result-1
da6391d5d1ed   cloud_user-worker   "dotnet Worker.dll"      20 minutes ago   Up 20 minutes                                           cloud_user-worker-1
0f01c8a8af97   postgres:9.4        "docker-entrypoint.s…"   20 minutes ago   Up 20 minutes   5432/tcp                                cloud_user-db-1
b4dac42b91af   cloud_user-vote     "gunicorn app:app -b…"   26 minutes ago   Up 20 minutes   0.0.0.0:5000->80/tcp, :::5000->80/tcp   cloud_user-vote-1
5d13b4d68208   redis               "docker-entrypoint.s…"   26 minutes ago   Up 20 minutes   6379/tcp                                cloud_user-redis-1
cloud_user@553b1e446c1c:~$ 
```
</details><br />

## Verify

By visiting **http://ip_address:5000/**

![]({{ site.baseurl }}/images/2024/07-17-Docker-Compose/01-voting-app-port-5000.png)

By visiting **http://ip_address:5001/**

![]({{ site.baseurl }}/images/2024/07-17-Docker-Compose/02-voting-app-port-5001.png)

You can also verify using cURL

```bash
curl localhost:5000
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ curl localhost:5000
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Cats vs Dogs!</title>
    <base href="/index.html">
    <meta name = "viewport" content = "width=device-width, initial-scale = 1.0">
    <meta name="keywords" content="docker-compose, docker, stack">
    <meta name="author" content="Tutum dev team">
    <link rel='stylesheet' href="/static/stylesheets/style.css" />
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.4.0/css/font-awesome.min.css">
  </head>
  <body>
    <div id="content-container">
      <div id="content-container-center">
        <h3>Cats vs Dogs!</h3>
        <form id="choice" name='form' method="POST" action="/">
          <button id="a" type="submit" name="vote" class="a" value="a">Cats</button>
          <button id="b" type="submit" name="vote" class="b" value="b">Dogs</button>
        </form>
        <div id="tip">
          (Tip: you can change your vote)
        </div>
        <div id="hostname">
          Processed by container ID b4dac42b91af
        </div>
      </div>
    </div>
    <script src="http://code.jquery.com/jquery-latest.min.js" type="text/javascript"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery-cookie/1.4.1/jquery.cookie.js"></script>

    
  </body>
</html>
```

</details><br />

```bash
curl localhost:5001
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ curl localhost:5001
<!DOCTYPE html>
<html ng-app="catsvsdogs">
  <head>
    <meta charset="utf-8">
    <title>Cats vs Dogs -- Result</title>
    <base href="/index.html">
    <meta name = "viewport" content = "width=device-width, initial-scale = 1.0">
    <meta name="keywords" content="docker-compose, docker, stack">
    <meta name="author" content="Docker">
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body ng-controller="statsCtrl" >
     <div id="background-stats">
       <div id="background-stats-1">
       </div><!--
      --><div id="background-stats-2">
      </div>
    </div>
    <div id="content-container">
      <div id="content-container-center">
        <div id="choice">
          <div class="choice cats">
            <div class="label">Cats</div>
            <div class="stat">{{aPercent | number:1}}%</div>
          </div>
          <div class="divider"></div>
          <div class="choice dogs">
            <div class="label">Dogs</div>
            <div class="stat">{{bPercent | number:1}}%</div>
          </div>
        </div>
      </div>
    </div>
    <div id="result">
      <span ng-if="total == 0">No votes yet</span>
      <span ng-if="total == 1">{{total}} vote</span>
      <span ng-if="total >= 2">{{total}} votes</span>
    </div>
    <script src="socket.io.js"></script>
    <script src="angular.min.js"></script>
    <script src="app.js"></script>
  </body>
</html>
cloud_user@553b1e446c1c:~$
```
</details><br />

## Stop the containers

```bash
docker compose down
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo docker compose down
[+] Running 6/6
 ✔ Container cloud_user-result-1  Removed                                                                                                                                                                                                                                                                0.8s 
 ✔ Container cloud_user-vote-1    Removed                                                                                                                                                                                                                                                                1.1s 
 ✔ Container cloud_user-worker-1  Removed                                                                                                                                                                                                                                                                0.9s 
 ✔ Container cloud_user-db-1      Removed                                                                                                                                                                                                                                                                0.5s 
 ✔ Container cloud_user-redis-1   Removed                                                                                                                                                                                                                                                                0.9s 
 ✔ Network cloud_user_default     Removed                                                                                                                                                                                                                                                                0.2s 
cloud_user@553b1e446c1c:~$ 

cloud_user@553b1e446c1c:~$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
cloud_user@553b1e446c1c:~$ 

cloud_user@553b1e446c1c:~$ sudo docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
cloud_user@553b1e446c1c:~$ 
```
</details><br />

## References

- [https://docs.docker.com/compose/install/standalone/](https://docs.docker.com/compose/install/standalone/)
- [https://github.com/dockersamples/example-voting-app](https://github.com/dockersamples/example-voting-app)