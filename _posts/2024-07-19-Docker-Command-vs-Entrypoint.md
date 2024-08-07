---
title: Docker CMD vs Entrypoint
date: 2024-07-19 21:30:00 -0700
categories: [DOCKER]
tags: [docker]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

Containers are meant to run a specific process and terminate. Once the task is completed, the containers exits. The container only lives as long as the process inside it is alive. If the service inside the container is stopped or crashes the containers exits.

**CMD** or **Entrypoint** instructions in the `Dockerfile` define what process will run within the container when it is started.

In the `Dockerfile`, the format for these instructions could be either:

```
CMD command param1          example: CMD sleep 5
```

or

```
CMD ["command", "param1"]   example: CMD ["sleep", "5"]
```

## CMD - used to overwrite

Let's say the `Dockerfile` has the following line with the command `sleep 5`.

```bash
CMD sleep 5
```

We can **overwrite** it at the moment of running the container as following:

```bash
docker run <image_name> [COMMAND]
```

Example:

```bash
docker run ubuntu sleep 100
```

Now when the containers starts, the command at startup is `sleep 100`. 

## Entrypoint - used to append

Let's say the `Dockerfile` has the following line with the command `sleep`.

```bash
ENTRYPOINT ["sleep"]
```

We can **append** it at the moment of running the container as following:

```bash
docker run <image_name> [COMMAND]
```

Example:

```bash
docker run ubuntu 100
```

Now when the containers starts, the command at startup is `sleep 100`. 

However, in the case we run `docker run <image_name>` with no `[COMMAND]`, the command at startup would be just `sleep` which would crash since it expect an integer number defining the number of seconds to sleep.

## Mix of CMD and Entrypoint

Let's say we add both in this way, just as a proof of concept.

```bash
ENTRYPOINT ["sleep"]
CMD ["5"]
```

In this case when running just:

```bash
docker run ubuntu
```

When the containers starts, the command at startup is `sleep 5`. 

However, when running:

```bash
docker run ubuntu 5
```

Now when the containers starts, the command at startup is `sleep 10`. 

## How to overwrite Entrypoint

Using the same example of:

```bash
ENTRYPOINT ["sleep"]
CMD ["5"]
```

To overwrite what was specified in the `entrypoint` configuration, start the container with the `--entrypoint` flag, for example:

```bash
docker run --entrypoint better_sleep ubuntu 5
```

Where `better_sleep` would be a different command than `sleep`.

## Docker inspect

The command `docker inspect` can be helpful to let us know what **CMD** or **Entrypoint** was used when the container was started.

```bash
ubuntu@ip-172-31-93-171:~$ sudo docker inspect a3e57aab0fb5
[
    {
        "Id": "a3e57aab0fb5dce0518761d434c0d52e9d0d1cb9c9b65919048d52d4ce89feac",
        "Created": "2024-07-20T04:41:20.193218174Z",
        "Path": "sleep",
        "Args": [
            "100"
        ],
        ...
        "Config": {
            ...
            "Cmd": [
                "sleep",
                "100"
            ],
            "Image": "ubuntu",
            ...
            "Entrypoint": null,
            ...
            }
        },
```

## References

- [https://github.com/mmumshad/simple-webapp-flask](https://github.com/mmumshad/simple-webapp-flask)