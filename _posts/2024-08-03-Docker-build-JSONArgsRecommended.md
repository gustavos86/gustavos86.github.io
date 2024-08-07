---
title: Docker build Warning - JSONArgsRecommended
date: 2024-08-03 11:30:00 -0700
categories: [DOCKER, DOCKER ERRORS]
tags: [docker, docker JSONArgsRecommended]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

While building this `Dockerfile`

<details markdown=1>
<summary markdown="span">Dockerfilecat </summary>

```bash
FROM ubuntu:20.04
RUN apt-get update && apt-get -y upgrade
RUN apt-get install -y lsb-release net-tools wget git vim curl tcpdump iputils-ping tree jq telnet sudo nano gnupg2
RUN curl -s https://deb.frrouting.org/frr/keys.asc | sudo apt-key add -
ENV FRRVER="frr-stable"
RUN echo deb https://deb.frrouting.org/frr $(lsb_release -s -c) $FRRVER | sudo tee -a /etc/apt/sources.list.d/frr.list
RUN apt update && apt install -y frr frr-pythontools
COPY daemons /etc/frr/daemons
COPY start_frr.sh /etc/frr/start_frr.sh
COPY vtysh.conf vtysh.conf
CMD /bin/bash /etc/frr/start_frr.sh
```
</details><br />

I got the following error

```bash
...
 1 warning found (use --debug to expand):
 - JSONArgsRecommended: JSON arguments recommended for CMD to prevent unintended behavior related to OS signals (line 11)
...
```

Complete output

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/build_frr_another_attempt$ docker build . -t frr_try_4
[+] Building 2.0s (14/14) FINISHED                                                                                                                                                           docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                                   0.0s
 => => transferring dockerfile: 616B                                                                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/ubuntu:20.04                                                                                                                                        0.3s
 => [internal] load .dockerignore                                                                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                                                                        0.0s
 => [1/9] FROM docker.io/library/ubuntu:20.04@sha256:0b897358ff6624825fb50d20ffb605ab0eaea77ced0adb8c6a4b756513dec6fc                                                                                  0.0s
 => [internal] load build context                                                                                                                                                                      0.1s
 => => transferring context: 2.79kB                                                                                                                                                                    0.0s
 => CACHED [2/9] RUN apt-get update && apt-get -y upgrade                                                                                                                                              0.0s
 => CACHED [3/9] RUN apt-get install -y lsb-release net-tools wget git vim curl tcpdump iputils-ping tree jq telnet sudo nano gnupg2                                                                   0.0s
 => CACHED [4/9] RUN curl -s https://deb.frrouting.org/frr/keys.asc | sudo apt-key add -                                                                                                               0.0s
 => CACHED [5/9] RUN echo deb https://deb.frrouting.org/frr $(lsb_release -s -c) frr-stable | sudo tee -a /etc/apt/sources.list.d/frr.list                                                             0.0s
 => CACHED [6/9] RUN apt update && apt install -y frr frr-pythontools                                                                                                                                  0.0s
 => [7/9] COPY daemons /etc/frr/daemons                                                                                                                                                                0.2s
 => [8/9] COPY start_frr.sh /etc/frr/start_frr.sh                                                                                                                                                      0.2s
 => [9/9] COPY vtysh.conf vtysh.conf                                                                                                                                                                   0.2s
 => exporting to image                                                                                                                                                                                 0.5s
 => => exporting layers                                                                                                                                                                                0.3s
 => => writing image sha256:d8dfb10daa91598dbe7e42af819f426dd385e011a401ff55d121afcfc3efdbc4                                                                                                           0.0s
 => => naming to docker.io/library/frr_try_4                                                                                                                                                           0.0s

 1 warning found (use --debug to expand):
 - JSONArgsRecommended: JSON arguments recommended for CMD to prevent unintended behavior related to OS signals (line 11)
cloud_user@553b1e446c1c:~/build_frr_another_attempt$
```
</details><br />

I tried using `--debug` but it seems not taken at all by `docker build`

```bash
cloud_user@553b1e446c1c:~/build_frr_another_attempt$ docker build . -t frr_try_4 --debug
unknown flag: --debug
See 'docker buildx build --help'.
cloud_user@553b1e446c1c:~/build_frr_another_attempt$ docker build --debug . -t frr_try_4
unknown flag: --debug
See 'docker buildx build --help'.
cloud_user@553b1e446c1c:~/build_frr_another_attempt$ docker build . --debug -t frr_try_4
unknown flag: --debug
See 'docker buildx build --help'.
cloud_user@553b1e446c1c:~/build_frr_another_attempt$
```

It is not even part of the help

```bash
cloud_user@553b1e446c1c:~/build_frr_another_attempt$ docker build --help
Start a build

Usage:  docker buildx build [OPTIONS] PATH | URL | -

Start a build

Aliases:
  docker buildx build, docker buildx b

Options:
      --add-host strings              Add a custom host-to-IP mapping (format: "host:ip")
      --allow strings                 Allow extra privileged entitlement (e.g., "network.host", "security.insecure")
      --annotation stringArray        Add annotation to the image
      --attest stringArray            Attestation parameters (format: "type=sbom,generator=image")
      --build-arg stringArray         Set build-time variables
      --build-context stringArray     Additional build contexts (e.g., name=path)
      --builder string                Override the configured builder instance (default "default")
      --cache-from stringArray        External cache sources (e.g., "user/app:cache", "type=local,src=path/to/dir")
      --cache-to stringArray          Cache export destinations (e.g., "user/app:cache", "type=local,dest=path/to/dir")
      --call string                   Set method for evaluating build ("check", "outline", "targets") (default "build")
      --cgroup-parent string          Set the parent cgroup for the "RUN" instructions during build
      --check                         Shorthand for "--call=check" (default )
  -f, --file string                   Name of the Dockerfile (default: "PATH/Dockerfile")
      --iidfile string                Write the image ID to a file
      --label stringArray             Set metadata for an image
      --load                          Shorthand for "--output=type=docker"
      --metadata-file string          Write build result metadata to a file
      --network string                Set the networking mode for the "RUN" instructions during build (default "default")
      --no-cache                      Do not use cache when building the image
      --no-cache-filter stringArray   Do not cache specified stages
  -o, --output stringArray            Output destination (format: "type=local,dest=path")
      --platform stringArray          Set target platform for build
      --progress string               Set type of progress output ("auto", "plain", "tty", "rawjson"). Use plain to show container output (default "auto")
      --provenance string             Shorthand for "--attest=type=provenance"
      --pull                          Always attempt to pull all referenced images
      --push                          Shorthand for "--output=type=registry"
  -q, --quiet                         Suppress the build output and print image ID on success
      --sbom string                   Shorthand for "--attest=type=sbom"
      --secret stringArray            Secret to expose to the build (format: "id=mysecret[,src=/local/secret]")
      --shm-size bytes                Shared memory size for build containers
      --ssh stringArray               SSH agent socket or keys to expose to the build (format: "default|<id>[=<socket>|<key>[,<key>]]")
  -t, --tag stringArray               Name and optionally a tag (format: "name:tag")
      --target string                 Set the target build stage to build
      --ulimit ulimit                 Ulimit options (default [])

Experimental commands and flags are hidden. Set BUILDX_EXPERIMENTAL=1 to show them.
cloud_user@553b1e446c1c:~/build_frr_another_attempt$
```

## The --check flag

However, the `--check` flag seems to be working. Taken from the help

```bash
      --check                         Shorthand for "--call=check" (default )
```

This gave a more verbose output

```bash
...
WARNING: JSONArgsRecommended - https://docs.docker.com/go/dockerfile/rule/json-args-recommended/
JSON arguments recommended for CMD to prevent unintended behavior related to OS signals
Dockerfile:11
--------------------
   9 |     COPY start_frr.sh /etc/frr/start_frr.sh
  10 |     COPY vtysh.conf vtysh.conf
  11 | >>> CMD /bin/bash /etc/frr/start_frr.sh
  12 |
--------------------
...
```

Complete output:

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~/build_frr_another_attempt$ docker build --check . -t frr_try_4
[+] Building 0.4s (3/3) FINISHED                                                                                                                                                             docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                                   0.0s
 => => transferring dockerfile: 616B                                                                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/ubuntu:20.04                                                                                                                                        0.2s
 => [internal] load .dockerignore                                                                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                                                                        0.0s

WARNING: JSONArgsRecommended - https://docs.docker.com/go/dockerfile/rule/json-args-recommended/
JSON arguments recommended for CMD to prevent unintended behavior related to OS signals
Dockerfile:11
--------------------
   9 |     COPY start_frr.sh /etc/frr/start_frr.sh
  10 |     COPY vtysh.conf vtysh.conf
  11 | >>> CMD /bin/bash /etc/frr/start_frr.sh
  12 |
--------------------
cloud_user@553b1e446c1c:~/build_frr_another_attempt$
```
</details><br />

Looking at [docker.docs JSONArgsRecommended](https://docs.docker.com/reference/build-checks/json-args-recommended/), there is a very nice and clear explanation.

*Description*

*ENTRYPOINT and CMD instructions both support two different syntaxes for arguments:*

    *Shell form: CMD my-cmd start*
    *Exec form: CMD ["my-cmd", "start"]*

*When you use shell form, the executable runs as a child process to a shell, which doesn't pass signals. This means that the program running in the container can't detect OS signals like SIGTERM and SIGKILL and respond to them correctly.*

So, in the `Dockerfile`, I replaced this line

```bash
CMD /bin/bash /etc/frr/start_frr.sh
```

with this other line

```bash
CMD ["/bin/bash", "/etc/frr/start_frr.sh"]
```

and that fixed the warning message

```bash
cloud_user@553b1e446c1c:~/build_frr_another_attempt$ docker build --check . -t frr_try_4
[+] Building 0.4s (3/3) FINISHED                                                                                                                                                             docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                                   0.0s
 => => transferring dockerfile: 623B                                                                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/ubuntu:20.04                                                                                                                                        0.2s
 => [internal] load .dockerignore                                                                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                                                                        0.0s
cloud_user@553b1e446c1c:~/build_frr_another_attempt$
```

**IMPORTANT:** Now build the image again (excluding the `--check`)

## References

- [docker.docs JSONArgsRecommended](https://docs.docker.com/reference/build-checks/json-args-recommended/)