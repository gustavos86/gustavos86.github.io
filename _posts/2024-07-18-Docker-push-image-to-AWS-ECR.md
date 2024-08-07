---
title: Docker push image to AWS ECR
date: 2024-07-18 06:00:00 -0700
categories: [DOCKER, AWS ECR]
tags: [docker, aws ecr]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

## ECR

Docker images can be pushed to public or private repositorioes in Amazon Elastic Container Registry (ECR).

## Create a repo in ECR

![]({{ site.baseurl }}/images/services/ecr.png)

In the **Amazon Management Console**, go to the ECR service and click on **Create a repository**

![]({{ site.baseurl }}/images/2024/07-18-Docker-push-image-to-AWS-ECR/01-Create-a-repository-main.png)

You will be presented with different options, including selection if this repository will be **private** or **public**. I went with **private** for this tutorial. You also need to specify a **repository name**

![]({{ site.baseurl }}/images/2024/07-18-Docker-push-image-to-AWS-ECR/02-Create-a-repository-settings.png)

The repo will be created, including an **URI**

![]({{ site.baseurl }}/images/2024/07-18-Docker-push-image-to-AWS-ECR/03-ECR-repo-created.png)

## Docker login

**NOTE:** a pre-requisite is having setup access to your AWS account using AWS CLI, You can refer to [Setup user profile in AWS CLI](https://myjourneytocloud.net/posts/Setup-user-profile-in-AWS-CLI/)

From the Docker Host, use the AWS CLI to login to ECR.

```bash
aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
```

Replace `aws_account_id` and `region` with your corresponding values.

**NOTE:** You may need to use **sudo** as in `sudo docker login` in this command depending on your Docker configuration.

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ aws ecr get-login-password | sudo docker login --username AWS --password-stdin 654654406075.dkr.ecr.us-east-1.amazonaws.com
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credential-stores

Login Succeeded
cloud_user@553b1e446c1c:~$ 
```
</details><br />

Next step is Tag properly the Docker Image, the usual format should me `<Registry>/<User or Account>/<Image or Repository>:tag`. When the `tag` is ommited Docker assigns `latest`. Sometimes `<User or Account>` is optional.

Once clicking on the newly create repository, on the upper-right side there is a button named simply **View push commands**

![]({{ site.baseurl }}/images/2024/07-18-Docker-push-image-to-AWS-ECR/04-View-push-commands-button.png)

This button displays a series of commands you can use in your Docker Host to tag your Docker Image properly in preparation for the push

![]({{ site.baseurl }}/images/2024/07-18-Docker-push-image-to-AWS-ECR/05-ECR-instructions.png)

I will tag properly the image named `cloud_user-vote` I already had built locally.

```bash
docker tag cloud_user-vote:latest 654654406075.dkr.ecr.us-east-1.amazonaws.com/cloud_user-vote:latest
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo docker images
REPOSITORY          TAG       IMAGE ID       CREATED        SIZE
cloud_user-vote     latest    3cd62f1d4c2c   10 hours ago   153MB
cloud_user@553b1e446c1c:~$ 

cloud_user@553b1e446c1c:~$ sudo docker tag cloud_user-vote:latest 654654406075.dkr.ecr.us-east-1.amazonaws.com/cloud_user-vote:latest
cloud_user@553b1e446c1c:~$ 

cloud_user@553b1e446c1c:~$ sudo docker images
REPOSITORY                                                     TAG       IMAGE ID       CREATED        SIZE
654654406075.dkr.ecr.us-east-1.amazonaws.com/cloud_user-vote   latest    3cd62f1d4c2c   10 hours ago   153MB
cloud_user-vote                                                latest    3cd62f1d4c2c   10 hours ago   153MB
cloud_user@553b1e446c1c:~$ 
```
</details><br />

And finally, the push to ECR

```bash
docker push 654654406075.dkr.ecr.us-east-1.amazonaws.com/cloud_user-vote:latest
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo docker push 654654406075.dkr.ecr.us-east-1.amazonaws.com/cloud_user-vote:latest
The push refers to repository [654654406075.dkr.ecr.us-east-1.amazonaws.com/cloud_user-vote]
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

The image with **tag** `latest` is now in the repository.

![]({{ site.baseurl }}/images/2024/07-18-Docker-push-image-to-AWS-ECR/06-ECR-push-successful.png)

## References

- [Private registry authentication in Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/registry_auth.html)
