---
title: Docker create an Image using Dockerfile
date: 2024-07-17 06:00:00 -0700
categories: [DOCKER, DOCKERFILE]
tags: [docker]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

## Dockerfile

Here is a **Dockerfile** which performs the following actions.

1. OS - ubuntu
2. Update apt repo
3. Install dependencias using apt
4. Install Python dependencias using pip
5. Copy source code to /opt folder
6. Run the web server using "flask" command

In the **Dockerfile**, the lines are in the following format:

`INSTRUCTION argument`

Dockerfile

```dockerfile
FROM ubuntu

ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt update
RUN apt install -y python3 python3-pip python3-flask

COPY app.py /opt/app.py

ENTRYPOINT FLASK_APP=/opt/app.py flask run --host=0.0.0.0
```

<details markdown=1>
<summary markdown="span">Dockerfile in the server</summary>

```bash
ubuntu@ip-172-31-92-44:~$ pwd
/home/ubuntu
ubuntu@ip-172-31-92-44:~$ cat Dockerfile
FROM ubuntu

ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt update
RUN apt install -y python3 python3-pip python3-flask

COPY app.py /opt/app.py

ENTRYPOINT FLASK_APP=/opt/app.py flask run --host=0.0.0.0
ubuntu@ip-172-31-92-44:~$ 
```
</details><br />

We will need to create the file **app.py** since the line `COPY app.py /opt/app.py` will copy it to the container during the build.

<details markdown=1>
<summary markdown="span">app.py in the server</summary>

```python
import os
from flask import Flask
app = Flask(__name__)

@app.route("/")
def main():
    return "Welcome!"

@app.route('/how are you')
def hello():
    return 'I am good, how about you?'

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

```bash
ubuntu@ip-172-31-92-44:~$ pwd
/home/ubuntu
ubuntu@ip-172-31-92-44:~$ cat app.py
import os
from flask import Flask
app = Flask(__name__)

@app.route("/")
def main():
    return "Welcome!"

@app.route('/how are you')
def hello():
    return 'I am good, how about you?'

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
ubuntu@ip-172-31-92-44:~$ 
```
</details><br />

These lines related to TimeZone are needed to avoid an issue where the build is stuck asking for the timezone to the user.

```bash
ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```

## Docker build

```bash
docker build . -t <tag>
```

**NOTE:** Make sure that `-t <tag>` is used, otherwise the image will just have the TAG `<none>` which is not useful at all.
**NOTE:** To specify a Dockerfile named differently, like `Dockerfile2` use the `-f` parameter. For example `docker build . -f Dockerfile2 -t <tag>`

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-92-44:~$ sudo docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
<none>       <none>    0e824991dd69   About a minute ago   596MB
ubuntu       latest    35a88802559d   5 weeks ago          78.1MB
ubuntu@ip-172-31-92-44:~$ 

ubuntu@ip-172-31-92-44:~$ sudo docker build . -t gustavos86/my-custom-app
[+] Building 0.0s (0/1)                                                                                                                                                                                                                                  docker:defaul[+] Building 0.1s (10/10) FINISHED                                                                                                                                                                                                                       docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                                                                                               0.0s
 => => transferring dockerfile: 311B                                                                                                                                                                                                                               0.0s
 => WARN: JSONArgsRecommended: JSON arguments recommended for ENTRYPOINT to prevent unintended behavior related to OS signals (line 11)                                                                                                                            0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                                                                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                                                                                                                                                  0.0s
 => => transferring context: 2B                                                                                                                                                                                                                                    0.0s
 => [1/5] FROM docker.io/library/ubuntu:latest                                                                                                                                                                                                                     0.0s
 => [internal] load build context                                                                                                                                                                                                                                  0.0s
 => => transferring context: 28B                                                                                                                                                                                                                                   0.0s
 => CACHED [2/5] RUN ln -snf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime && echo America/Los_Angeles > /etc/timezone                                                                                                                                    0.0s
 => CACHED [3/5] RUN apt update                                                                                                                                                                                                                                    0.0s
 => CACHED [4/5] RUN apt install -y python3 python3-pip python3-flask                                                                                                                                                                                              0.0s
 => CACHED [5/5] COPY app.py /opt/app.py                                                                                                                                                                                                                           0.0s
 => exporting to image                                                                                                                                                                                                                                             0.0s
 => => exporting layers                                                                                                                                                                                                                                            0.0s
 => => writing image sha256:0e824991dd69b625b14806cc4b1c937be1164c02f6b8edff663902402700c1fd                                                                                                                                                                       0.0s
 => => naming to docker.io/gustavos86/my-custom-app                                                                                                                                                                                                                 0.0s

 1 warning found (use --debug to expand):
 - JSONArgsRecommended: JSON arguments recommended for ENTRYPOINT to prevent unintended behavior related to OS signals (line 11)
ubuntu@ip-172-31-92-44:~$ 

ubuntu@ip-172-31-92-44:~$ sudo docker images
REPOSITORY                TAG       IMAGE ID       CREATED         SIZE
gustavos86/my-custom-app   latest    0e824991dd69   2 minutes ago   596MB
ubuntu                    latest    35a88802559d   5 weeks ago     78.1MB
ubuntu@ip-172-31-92-44:~$ 
```
</details><br />

The dot `.` in the previous command was just implicitly saying use the **Dockerfile** named file in the currect directory.
Other way to run the command would be as follows.

```bash
docker build Dockerfile -t <tag>
```

## Docker login

To login to https://hub.docker.com/

```bash
docker login
```

**IMPORTANT:** Make sure that you use `sudo` with the `docker login` command if necessary. Most importantly, make sure this command is successful, otherwilse `docker push` will give a permissions relate errors when trying to upload the image to Docker Hub.

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-92-44:~$ sudo docker login
Log in with your Docker ID or email address to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com/ to create one.
You can log in with your password or a Personal Access Token (PAT). Using a limited-scope PAT grants better security and is required for organizations using SSO. Learn more at https://docs.docker.com/go/access-tokens/

Username: gustavos86
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credential-stores

Login Succeeded
ubuntu@ip-172-31-92-44:~$ 
```
</details><br />

## Docker push

```bash
docker push <repo/image_name:tagname>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-92-44:~$ sudo docker push gustavos86/my-custom-app:latest
The push refers to repository [docker.io/gustavos86/my-custom-app]
269a6fe381ea: Pushed 
64439ab4555c: Pushed 
84e90d879356: Pushed 
4bb858399d4d: Pushed 
a30a5965a4f7: Mounted from library/ubuntu 
latest: digest: sha256:3668b117d7
```
</details><br />

Once pushed, a new repo will be created in Docker Hub.

## Docker history

```bash
docker history <image_name>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-92-44:~$ sudo docker history gustavos86/my-custom-app
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
ubuntu@ip-172-31-92-44:~$ 
```
</details><br />

Image will be pushed to DockerHub

![]({{ site.baseurl }}/images/2024/07-17-Docker-create-image/01-DockerHub-repo-created.png)

## References

- [https://github.com/mmumshad/simple-webapp-flask](https://github.com/mmumshad/simple-webapp-flask)
- [https://grigorkh.medium.com/fix-tzdata-hangs-docker-image-build-cdb52cc3360d](https://grigorkh.medium.com/fix-tzdata-hangs-docker-image-build-cdb52cc3360d)