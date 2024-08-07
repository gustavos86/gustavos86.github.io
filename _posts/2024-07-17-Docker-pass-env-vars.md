---
title: Docker pass environment variables to container when initialized
date: 2024-07-17 08:00:00 -0700
categories: [DOCKER, COMMANDS]
tags: [docker]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

## Pass environment variables to a container

Here an example of an container that accepts environment variables during container's initialization

```bash
docker pull mmumshad/simple-webapp-color
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-92-44:~$ sudo docker pull mmumshad/simple-webapp-color
Using default tag: latest
latest: Pulling from mmumshad/simple-webapp-color
55cbf04beb70: Pull complete 
1607093a898c: Pull complete 
9a8ea045c926: Pull complete 
d4eee24d4dac: Pull complete 
b59856e9f0ab: Pull complete 
b023afffd10b: Pull complete 
13e2e806d7c8: Pull complete 
e90bc178f045: Pull complete 
bd415728f75a: Pull complete 
06d08c7638af: Pull complete 
98b4690dc1c7: Pull complete 
b2567acc3f18: Pull complete 
3a4e7915e211: Pull complete 
Digest: sha256:637eff5492960b6620d2c86bb9a5355408ebfc69234172502049e34b6ee94057
Status: Downloaded newer image for mmumshad/simple-webapp-color:latest
docker.io/mmumshad/simple-webapp-color:latest
ubuntu@ip-172-31-92-44:~$ 
```
</details><br />

Use the `-e` parameter to pass environmental variables to the container upon initializing. Example:

Here, we are passing `APP_COLOR` is equal to a certain string, green, red and blue in that order. This will change how the app behaves.

```bash
docker run -e APP_COLOR=green -p 8080:8080 mmumshad/simple-webapp-color
```

<details markdown=1>
<summary markdown="span">output</summary>

![]({{ site.baseurl }}/images/2024/07-17-Docker-pass-env-vars/01-app_color-green.png)

```bash
ubuntu@ip-172-31-92-44:~$ sudo docker run -e APP_COLOR=green -p 8080:8080 mmumshad/simple-webapp-color
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
174.21.185.203 - - [17/Jul/2024 15:23:44] "GET / HTTP/1.1" 200 -

```
</details><br />

```bash
docker run -e APP_COLOR=red   -p 8080:8080 mmumshad/simple-webapp-color
```

<details markdown=1>
<summary markdown="span">output</summary>

![]({{ site.baseurl }}/images/2024/07-17-Docker-pass-env-vars/01-app_color-red.png)

```bash
ubuntu@ip-172-31-92-44:~$ sudo docker run -e APP_COLOR=red   -p 8080:8080 mmumshad/simple-webapp-color
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
174.21.185.203 - - [17/Jul/2024 15:24:59] "GET / HTTP/1.1" 200 -

```
</details><br />

```bash
docker run -e APP_COLOR=blue  -p 8080:8080 mmumshad/simple-webapp-color
```

<details markdown=1>
<summary markdown="span">output</summary>

![]({{ site.baseurl }}/images/2024/07-17-Docker-pass-env-vars/01-app_color-blue.png)

```bash
ubuntu@ip-172-31-92-44:~$ sudo docker run -e APP_COLOR=blue   -p 8080:8080 mmumshad/simple-webapp-color
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
174.21.185.203 - - [17/Jul/2024 15:26:28] "GET / HTTP/1.1" 200 -

```
</details>

## See environmental variables used to initiate a container

```bash
docker inspect <container_name> | <container_id>
```

<details markdown=1>
<summary markdown="span">output</summary>

![]({{ site.baseurl }}/images/2024/07-17-Docker-pass-env-vars/01-app_color-blue.png)

Notice lines:

```
"Env": [
    "APP_COLOR=blue",
```

**NOTE:** Not all output was included

```bash
ubuntu@ip-172-31-92-44:~$ sudo docker ps
CONTAINER ID   IMAGE                          COMMAND             CREATED         STATUS         PORTS                                       NAMES
69a63bda5b5c   mmumshad/simple-webapp-color   "python ./app.py"   7 minutes ago   Up 7 minutes   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   boring_hugle
ubuntu@ip-172-31-92-44:~$ 

ubuntu@ip-172-31-92-44:~$ sudo docker inspect 69a63b
[
    {
        "Config": {
            "Hostname": "69a63bda5b5c",
            ...
            "ExposedPorts": {
                "8080/tcp": {}
            },
            ...
            "Env": [
                "APP_COLOR=blue",
                "PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "GPG_KEY=0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D",
                "PYTHON_VERSION=3.7.0",
                "PYTHON_PIP_VERSION=18.0"
            ],
            "Cmd": [
                "python",
                "./app.py"
            ],
            ...
        },
    }
]
```
</details>

# References

- [https://github.com/mmumshad/simple-webapp-color/blob/master/app.py](https://github.com/mmumshad/simple-webapp-color/blob/master/app.py)