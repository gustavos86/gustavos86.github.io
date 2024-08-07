---
title: Launching an AWS EKS Cluster
date: 2024-07-15 19:31:00 -0700
categories: [AWS, EKS]
tags: [aws, eks]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

## Create IAM User

![]({{ site.baseurl }}/images/services/iam.png)

1. You could name the user **k8-admin**
2. Attach the Policy **AdministratorAccess** to it
3. Follow [Setup user profile in AWS CLI](https://myjourneytocloud.net/posts/Setup-user-profile-in-AWS-CLI/) to generate the AWS CLI keys
4. Document the **Access key** and **Secret access key**

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/01-Attaching-Policy-to-IAM-User.png)

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/02-Creating-Access-Keys-for-IAM-User.png)

## Lanch an EC2 instance to be used as Admin Workstation

![]({{ site.baseurl }}/images/services/ec2.png)

1. **AMI:** Amazon Linux 2 AMI
2. **Instance type:** t2.micro
3. **Key pair (login):** create a new RSA pem key pair "my-key"
4. download the key pair (my-key.pem)
5. **Network settings:** enable "Auto-assign public IP"
6. Launch the instance

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/03-Launching_EC2_instance_main.png)

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/04-Launching_EC2_instance_instance_type.png)

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/05-Launching_EC2_instance_networking.png)

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/06-Launching_EC2_instance_storage.png)

## Upgrade to AWS CLI v2.x on the Admin Workstation

Log in to the EC2 Instance Admin Workstation.

AWS CLI is currently on version 1.x

```bash
aws --version
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ aws --version
aws-cli/1.18.147 Python/2.7.18 Linux/5.10.220-209.867.amzn2.x86_64 botocore/1.18.6
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 57.9M  100 57.9M    0     0   130M      0 --:--:-- --:--:-- --:--:--  131M
[ec2-user@ip-10-0-1-88 ~]$ 

```
</details><br />

```bash
unzip awscliv2.zip
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
...
  inflating: aws/dist/docutils/parsers/rst/include/isolat2.txt  
  inflating: aws/dist/docutils/parsers/rst/include/isolat1.txt  
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

```bash
which aws
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ which aws
/usr/bin/aws
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

```bash
sudo ./aws/install --bin-dir /usr/bin --install-dir /usr/bin/aws-cli --update
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ sudo ./aws/install --bin-dir /usr/bin --install-dir /usr/bin/aws-cli --update
You can now run: /usr/bin/aws --version
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

AWS CLI should now be on version 2.x

```bash
aws --version
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ aws --version
aws-cli/2.17.13 Python/3.11.9 Linux/5.10.220-209.867.amzn2.x86_64 exe/x86_64.amzn.2
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

```bash
aws configure
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ aws configure
AWS Access Key ID [None]: AKIAYUFKLA4UHA3NRH46
AWS Secret Access Key [None]: *****
Default region name [None]: us-east-1
Default output format [None]: json
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

## Install kubectl on the Admin Workstation

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.16.8/2020-04-16/bin/linux/amd64/kubectl
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.16.8/2020-04-16/bin/linux/amd64/kubectl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 55.7M  100 55.7M    0     0  18.1M      0  0:00:03  0:00:03 --:--:-- 18.1M
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />


```bash
chmod +x ./kubectl
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ chmod +x ./kubectl
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

```bash
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

```bash
kubectl version --short --client
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ kubectl version --short --client
Client Version: v1.16.8-eks-e16311
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

## Install eksctl on the Admin Workstation

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />


```bash
sudo mv /tmp/eksctl /usr/bin
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ sudo mv /tmp/eksctl /usr/bin
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

```bash
eksctl version
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ eksctl version
0.186.0
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

```bash
eksctl help
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ eksctl help
The official CLI for Amazon EKS

Usage: eksctl [command] [flags]

Commands:
  eksctl anywhere                        EKS anywhere
  eksctl associate                       Associate resources with a cluster
  eksctl completion                      Generates shell completion scripts for bash, zsh or fish
  eksctl create                          Create resource(s)
  eksctl delete                          Delete resource(s)
  eksctl deregister                      Deregister a non-EKS cluster
  eksctl disassociate                    Disassociate resources from a cluster
  eksctl drain                           Drain resource(s)
  eksctl enable                          Enable features in a cluster
  eksctl get                             Get resource(s)
  eksctl help                            Help about any command
  eksctl info                            Output the version of eksctl, kubectl and OS info
  eksctl register                        Register a non-EKS cluster
  eksctl scale                           Scale resources(s)
  eksctl set                             Set values
  eksctl unset                           Unset values
  eksctl update                          Update resource(s)
  eksctl upgrade                         Upgrade resource(s)
  eksctl utils                           Various utils
  eksctl version                         Output the version of eksctl

Common flags:
  -C, --color string   toggle colorized logs (valid options: true, false, fabulous) (default "true")
  -d, --dumpLogs       dump logs to disk on failure if set to true
  -h, --help           help for this command
  -v, --verbose int    set log level, use 0 to silence, 4 for debugging and 5 for debugging with AWS debug logging (default 3)

Use 'eksctl [command] --help' for more information about a command.


For detailed docs go to https://eksctl.io/

[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

## Create EKS cluster using eksctl


```bash
eksctl create cluster \
  --name dev-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-1-88 ~]$ eksctl create cluster \
>   --name dev-cluster \
>   --region us-east-1 \
>   --nodegroup-name standard-workers \
>   --node-type t3.medium \
>   --nodes 3 \
>   --nodes-min 1 \
>   --nodes-max 4 \
>   --managed
2024-07-16 03:25:38 [ℹ]  eksctl version 0.186.0
2024-07-16 03:25:38 [ℹ]  using region us-east-1
Error: getting availability zones: error getting availability zones for region us-east-1: operation error EC2: DescribeAvailabilityZones, https response error StatusCode: 403, RequestID: d9d1ffa8-4d43-47a2-bdb2-3d3f4dcf13dd, api error UnauthorizedOperation: You are not authorized to perform this operation. User: arn:aws:iam::593062725416:user/k8-admin is not authorized to perform: ec2:DescribeAvailabilityZones because no identity-based policy allows the ec2:DescribeAvailabilityZones action
[ec2-user@ip-10-0-1-88 ~]$ 
```
</details><br />

I got an error: 

  Error: getting availability zones: error getting availability zones for region us-east-1: operation error EC2: DescribeAvailabilityZones, https response error StatusCode: 403, RequestID: d9d1ffa8-4d43-47a2-bdb2-3d3f4dcf13dd, api error UnauthorizedOperation: You are not authorized to perform this operation. User: arn:aws:iam::593062725416:user/k8-admin is not authorized to perform: ec2:DescribeAvailabilityZones because no identity-based policy allows the ec2:DescribeAvailabilityZones action

The error was because I actually did not add the **AdministratorAccess** IAM Policy to the **k8-admin** IAM User at the moment I created this user on the first steps of this guide.

<details markdown=1>
<summary markdown="span">Now attaching the IAM Policy to the User</summary>

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/07-Fixing-Attaching-Policy-to-IAM-User.png)

</details><br />


<details markdown=1>
<summary markdown="span">eksctl create cluster working OK now</summary>

```bash
[ec2-user@ip-10-0-0-225 ~]$ eksctl create cluster \
>   --name dev-cluster \
>   --region us-east-1 \
>   --nodegroup-name standard-workers \
>   --node-type t3.medium \
>   --nodes 3 \
>   --nodes-min 1 \
>   --nodes-max 4 \
>   --managed
2024-07-16 04:04:29 [ℹ]  eksctl version 0.186.0
2024-07-16 04:04:29 [ℹ]  using region us-east-1
2024-07-16 04:04:29 [ℹ]  setting availability zones to [us-east-1f us-east-1d]
2024-07-16 04:04:29 [ℹ]  subnets for us-east-1f - public:192.168.0.0/19 private:192.168.64.0/19
2024-07-16 04:04:29 [ℹ]  subnets for us-east-1d - public:192.168.32.0/19 private:192.168.96.0/19
2024-07-16 04:04:29 [ℹ]  nodegroup "standard-workers" will use "" [AmazonLinux2/1.30]
2024-07-16 04:04:29 [ℹ]  using Kubernetes version 1.30
2024-07-16 04:04:29 [ℹ]  creating EKS cluster "dev-cluster" in "us-east-1" region with managed nodes
2024-07-16 04:04:29 [ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial managed nodegroup
2024-07-16 04:04:29 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-east-1 --cluster=dev-cluster'
2024-07-16 04:04:29 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "dev-cluster" in "us-east-1"
2024-07-16 04:04:29 [ℹ]  CloudWatch logging will not be enabled for cluster "dev-cluster" in "us-east-1"
2024-07-16 04:04:29 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=us-east-1 --cluster=dev-cluster'
2024-07-16 04:04:29 [ℹ]  default addons vpc-cni, kube-proxy, coredns were not specified, will install them as EKS addons
2024-07-16 04:04:29 [ℹ]  
2 sequential tasks: { create cluster control plane "dev-cluster", 
    2 sequential sub-tasks: { 
        2 sequential sub-tasks: { 
            1 task: { create addons },
            wait for control plane to become ready,
        },
        create managed nodegroup "standard-workers",
    } 
}
2024-07-16 04:04:29 [ℹ]  building cluster stack "eksctl-dev-cluster-cluster"
2024-07-16 04:04:29 [ℹ]  deploying stack "eksctl-dev-cluster-cluster"
2024-07-16 04:04:59 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:05:29 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:06:29 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:07:29 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
```
</details><br />

## Wait on CloudFormation

![]({{ site.baseurl }}/images/services/cloudformation.png)

In the CloudFormation section, you can see the progress.

It may take around **15 minutes** to complete.

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/08-CloudFormation_in_progress.png)

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/09-CloudFormation_in_progress_details.png)

Once CloudFormation completes, this is the whole output of `eksctl create cluster` command.

<details markdown=1>
<summary markdown="span">eksctl create cluster complete output</summary>

```bash
[ec2-user@ip-10-0-0-225 ~]$ eksctl create cluster \
>   --name dev-cluster \
>   --region us-east-1 \
>   --nodegroup-name standard-workers \
>   --node-type t3.medium \
>   --nodes 3 \
>   --nodes-min 1 \
>   --nodes-max 4 \
>   --managed
2024-07-16 04:04:29 [ℹ]  eksctl version 0.186.0
2024-07-16 04:04:29 [ℹ]  using region us-east-1
2024-07-16 04:04:29 [ℹ]  setting availability zones to [us-east-1f us-east-1d]
2024-07-16 04:04:29 [ℹ]  subnets for us-east-1f - public:192.168.0.0/19 private:192.168.64.0/19
2024-07-16 04:04:29 [ℹ]  subnets for us-east-1d - public:192.168.32.0/19 private:192.168.96.0/19
2024-07-16 04:04:29 [ℹ]  nodegroup "standard-workers" will use "" [AmazonLinux2/1.30]
2024-07-16 04:04:29 [ℹ]  using Kubernetes version 1.30
2024-07-16 04:04:29 [ℹ]  creating EKS cluster "dev-cluster" in "us-east-1" region with managed nodes
2024-07-16 04:04:29 [ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial managed nodegroup
2024-07-16 04:04:29 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-east-1 --cluster=dev-cluster'
2024-07-16 04:04:29 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "dev-cluster" in "us-east-1"
2024-07-16 04:04:29 [ℹ]  CloudWatch logging will not be enabled for cluster "dev-cluster" in "us-east-1"
2024-07-16 04:04:29 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=us-east-1 --cluster=dev-cluster'
2024-07-16 04:04:29 [ℹ]  default addons vpc-cni, kube-proxy, coredns were not specified, will install them as EKS addons
2024-07-16 04:04:29 [ℹ]  
2 sequential tasks: { create cluster control plane "dev-cluster", 
    2 sequential sub-tasks: { 
        2 sequential sub-tasks: { 
            1 task: { create addons },
            wait for control plane to become ready,
        },
        create managed nodegroup "standard-workers",
    } 
}
2024-07-16 04:04:29 [ℹ]  building cluster stack "eksctl-dev-cluster-cluster"
2024-07-16 04:04:29 [ℹ]  deploying stack "eksctl-dev-cluster-cluster"
2024-07-16 04:04:59 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:05:29 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:06:29 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:07:29 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:08:29 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:09:30 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:10:30 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:11:30 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:12:30 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:13:30 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-cluster"
2024-07-16 04:13:31 [!]  recommended policies were found for "vpc-cni" addon, but since OIDC is disabled on the cluster, eksctl cannot configure the requested permissions; the recommended way to provide IAM permissions for "vpc-cni" addon is via pod identity associations; after addon creation is completed, add all recommended policies to the config file, under `addon.PodIdentityAssociations`, and run `eksctl update addon`
2024-07-16 04:13:31 [ℹ]  creating addon
2024-07-16 04:13:31 [ℹ]  successfully created addon
2024-07-16 04:13:31 [ℹ]  creating addon
2024-07-16 04:13:32 [ℹ]  successfully created addon
2024-07-16 04:13:32 [ℹ]  creating addon
2024-07-16 04:13:32 [ℹ]  successfully created addon
2024-07-16 04:15:33 [ℹ]  building managed nodegroup stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 04:15:33 [ℹ]  deploying stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 04:15:33 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 04:16:03 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 04:16:59 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 04:18:56 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 04:18:56 [ℹ]  waiting for the control plane to become ready
2024-07-16 04:18:57 [✔]  saved kubeconfig as "/home/ec2-user/.kube/config"
2024-07-16 04:18:57 [ℹ]  no tasks
2024-07-16 04:18:57 [✔]  all EKS cluster resources for "dev-cluster" have been created
2024-07-16 04:18:57 [✔]  created 0 nodegroup(s) in cluster "dev-cluster"
2024-07-16 04:18:57 [ℹ]  nodegroup "standard-workers" has 3 node(s)
2024-07-16 04:18:57 [ℹ]  node "ip-192-168-24-195.ec2.internal" is ready
2024-07-16 04:18:57 [ℹ]  node "ip-192-168-36-113.ec2.internal" is ready
2024-07-16 04:18:57 [ℹ]  node "ip-192-168-51-2.ec2.internal" is ready
2024-07-16 04:18:57 [ℹ]  waiting for at least 1 node(s) to become ready in "standard-workers"
2024-07-16 04:18:57 [ℹ]  nodegroup "standard-workers" has 3 node(s)
2024-07-16 04:18:57 [ℹ]  node "ip-192-168-24-195.ec2.internal" is ready
2024-07-16 04:18:57 [ℹ]  node "ip-192-168-36-113.ec2.internal" is ready
2024-07-16 04:18:57 [ℹ]  node "ip-192-168-51-2.ec2.internal" is ready
2024-07-16 04:18:57 [✔]  created 1 managed nodegroup(s) in cluster "dev-cluster"
2024-07-16 04:18:58 [ℹ]  kubectl command should work with "/home/ec2-user/.kube/config", try 'kubectl get nodes'
2024-07-16 04:18:58 [✔]  EKS cluster "dev-cluster" in "us-east-1" region is ready
[ec2-user@ip-10-0-0-225 ~]$ 
```
</details><br />

## Verify the EKS cluster

![]({{ site.baseurl }}/images/services/eks.png)

Once CloudFormation has completed successfully, go to the EKS service.
The EKS cluster should be there.

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/10-EKS-cluster-created.png)

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/11-EKS-cluster-overview.png)

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/12-EKS-cluster-networking.png)

![]({{ site.baseurl }}/images/services/ec2.png)

Verify the K8s worker nodes were created successfully.

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/13-EC2-workier-nodes-created.png)

```bash
eksctl get cluster
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 ~]$ eksctl get cluster
NAME            REGION          EKSCTL CREATED
dev-cluster     us-east-1       True
[ec2-user@ip-10-0-0-225 ~]$ 
```
</details><br />

Enable us to connect to our cluster by configuring `kubectl`

```bash
aws eks update-kubeconfig \
  --name dev-cluster \
  --region us-east-1
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 ~]$ aws eks update-kubeconfig \
>   --name dev-cluster \
>   --region us-east-1
Added new context arn:aws:eks:us-east-1:480231763066:cluster/dev-cluster to /home/ec2-user/.kube/config
[ec2-user@ip-10-0-0-225 ~]$
```
</details><br />

## Create a K8s Deployment

Install `git`

```bash
sudo yum install -y git
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 ~]$ sudo yum install -y git
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                                                                                                                                                                             | 3.6 kB  00:00:00     
Resolving Dependencies
--> Running transaction check
---> Package git.x86_64 0:2.40.1-1.amzn2.0.3 will be installed
--> Processing Dependency: git-core = 2.40.1-1.amzn2.0.3 for package: git-2.40.1-1.amzn2.0.3.x86_64
--> Processing Dependency: git-core-doc = 2.40.1-1.amzn2.0.3 for package: git-2.40.1-1.amzn2.0.3.x86_64
--> Processing Dependency: perl-Git = 2.40.1-1.amzn2.0.3 for package: git-2.40.1-1.amzn2.0.3.x86_64
--> Processing Dependency: perl(Git) for package: git-2.40.1-1.amzn2.0.3.x86_64
--> Processing Dependency: perl(Term::ReadKey) for package: git-2.40.1-1.amzn2.0.3.x86_64
--> Running transaction check
---> Package git-core.x86_64 0:2.40.1-1.amzn2.0.3 will be installed
---> Package git-core-doc.noarch 0:2.40.1-1.amzn2.0.3 will be installed
---> Package perl-Git.noarch 0:2.40.1-1.amzn2.0.3 will be installed
--> Processing Dependency: perl(Error) for package: perl-Git-2.40.1-1.amzn2.0.3.noarch
---> Package perl-TermReadKey.x86_64 0:2.30-20.amzn2.0.2 will be installed
--> Running transaction check
---> Package perl-Error.noarch 1:0.17020-2.amzn2 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================================================================================================================
 Package                                                  Arch                                           Version                                                     Repository                                          Size
==============================================================================================================================================================================================================================
Installing:
 git                                                      x86_64                                         2.40.1-1.amzn2.0.3                                          amzn2-core                                          54 k
Installing for dependencies:
 git-core                                                 x86_64                                         2.40.1-1.amzn2.0.3                                          amzn2-core                                          10 M
 git-core-doc                                             noarch                                         2.40.1-1.amzn2.0.3                                          amzn2-core                                         3.0 M
 perl-Error                                               noarch                                         1:0.17020-2.amzn2                                           amzn2-core                                          32 k
 perl-Git                                                 noarch                                         2.40.1-1.amzn2.0.3                                          amzn2-core                                          42 k
 perl-TermReadKey                                         x86_64                                         2.30-20.amzn2.0.2                                           amzn2-core                                          31 k

Transaction Summary
==============================================================================================================================================================================================================================
Install  1 Package (+5 Dependent packages)

Total download size: 13 M
Installed size: 44 M
Downloading packages:
(1/6): git-2.40.1-1.amzn2.0.3.x86_64.rpm                                                                                                                                                               |  54 kB  00:00:00     
(2/6): git-core-2.40.1-1.amzn2.0.3.x86_64.rpm                                                                                                                                                          |  10 MB  00:00:00     
(3/6): git-core-doc-2.40.1-1.amzn2.0.3.noarch.rpm                                                                                                                                                      | 3.0 MB  00:00:00     
(4/6): perl-Error-0.17020-2.amzn2.noarch.rpm                                                                                                                                                           |  32 kB  00:00:00     
(5/6): perl-Git-2.40.1-1.amzn2.0.3.noarch.rpm                                                                                                                                                          |  42 kB  00:00:00     
(6/6): perl-TermReadKey-2.30-20.amzn2.0.2.x86_64.rpm                                                                                                                                                   |  31 kB  00:00:00     
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                          43 MB/s |  13 MB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : git-core-2.40.1-1.amzn2.0.3.x86_64
  . . .[########################################################################################################################################################################  ] 6/  Installing : git-2.40.1-1.amzn2.0.3.x86_64                                                                                                                                                                              6/6 
  Verifying  : perl-TermReadKey-2.30-20.amzn2.0.2.x86_64                                                                                                                                                                  1/6 
  Verifying  : git-2.40.1-1.amzn2.0.3.x86_64                                                                                                                                                                              2/6 
  Verifying  : 1:perl-Error-0.17020-2.amzn2.noarch                                                                                                                                                                        3/6 
  Verifying  : git-core-2.40.1-1.amzn2.0.3.x86_64                                                                                                                                                                         4/6 
  Verifying  : git-core-doc-2.40.1-1.amzn2.0.3.noarch                                                                                                                                                                     5/6 
  Verifying  : perl-Git-2.40.1-1.amzn2.0.3.noarch                                                                                                                                                                         6/6 

Installed:
  git.x86_64 0:2.40.1-1.amzn2.0.3                                                                                                                                                                                             

Dependency Installed:
  git-core.x86_64 0:2.40.1-1.amzn2.0.3      git-core-doc.noarch 0:2.40.1-1.amzn2.0.3      perl-Error.noarch 1:0.17020-2.amzn2      perl-Git.noarch 0:2.40.1-1.amzn2.0.3      perl-TermReadKey.x86_64 0:2.30-20.amzn2.0.2     

Complete!
[ec2-user@ip-10-0-0-225 ~]$ 
```
</details><br />

```bash
git clone https://github.com/ACloudGuru-Resources/Course_EKS-Basics
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 ~]$ git clone https://github.com/ACloudGuru-Resources/Course_EKS-Basics
Cloning into 'Course_EKS-Basics'...
remote: Enumerating objects: 21, done.
remote: Counting objects: 100% (21/21), done.
remote: Compressing objects: 100% (20/20), done.
remote: Total 21 (delta 8), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (21/21), 5.37 KiB | 2.68 MiB/s, done.
Resolving deltas: 100% (8/8), done.
[ec2-user@ip-10-0-0-225 ~]$
```
</details><br />

```bash
cd Course_EKS-Basics
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 ~]$ cd Course_EKS-Basics
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

```bash
ls -l
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ ls -l
total 12
-rw-rw-r-- 1 ec2-user ec2-user 1069 Jul 16 04:32 LICENSE
-rw-rw-r-- 1 ec2-user ec2-user  328 Jul 16 04:32 nginx-deployment.yaml
-rw-rw-r-- 1 ec2-user ec2-user  154 Jul 16 04:32 nginx-svc.yaml
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

<details markdown=1>
<summary markdown="span">nginx-deployment.yaml</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    env: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      env: dev
  template:
    metadata:
      labels:
        env: dev
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
</details><br />

<details markdown=1>
<summary markdown="span">nginx-svc.yaml</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    env: dev
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    env: dev
```
</details><br />

**NOTE:** Services should be created before the Pods (Deployment)

```bash
kubectl apply -f ./nginx-svc.yaml
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl apply -f ./nginx-svc.yaml
service/nginx-svc created
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

```bash
kubectl get service
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get service
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP                                                             PORT(S)        AGE
kubernetes   ClusterIP      10.100.0.1       <none>                                                                  443/TCP        26m
nginx-svc    LoadBalancer   10.100.195.251   ab0312c96509e4e0db8df45e1c4a76f4-26133787.us-east-1.elb.amazonaws.com   80:30089/TCP   24s
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

```bash
kubectl apply -f ./nginx-deployment.yaml
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl apply -f ./nginx-deployment.yaml
deployment.apps/nginx-deployment created
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

```bash
kubectl get deployment
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           18s
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

```bash
kubectl get pod
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-67c44fbdf6-9khhz   1/1     Running   0          49s
nginx-deployment-67c44fbdf6-wl2vf   1/1     Running   0          50s
nginx-deployment-67c44fbdf6-zftn7   1/1     Running   0          49s
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

```bash
kubectl get rs
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-67c44fbdf6   3         3         3       69s
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

```bash
kubectl get node
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get node
NAME                             STATUS   ROLES    AGE   VERSION
ip-192-168-24-195.ec2.internal   Ready    <none>   21m   v1.30.0-eks-036c24b
ip-192-168-36-113.ec2.internal   Ready    <none>   21m   v1.30.0-eks-036c24b
ip-192-168-51-2.ec2.internal     Ready    <none>   21m   v1.30.0-eks-036c24b
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

## Access the application for testing

The AWS ELB exposes the application to the Internet.

```bash
curl <DNS name of the ELB>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ curl ab0312c96509e4e0db8df45e1c4a76f4-26133787.us-east-1.elb.amazonaws.com
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

In a web browser navigate to <DNS name of the ELB>

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/14-Testing-the-app.png)

## Verify the EKS cluster redundancy

![]({{ site.baseurl }}/images/services/ec2.png)

Go to EC2, select all 3 worker nodes and stop them.

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/15-Stopping-worker-node.png)

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/16-Stopping-worker-node.png)

You should see new worker nodes and pods being created.

```bash
kubectl get node
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get node
NAME                             STATUS                        ROLES    AGE   VERSION
ip-192-168-24-195.ec2.internal   NotReady                      <none>   29m   v1.30.0-eks-036c24b
ip-192-168-36-113.ec2.internal   NotReady,SchedulingDisabled   <none>   29m   v1.30.0-eks-036c24b
ip-192-168-51-2.ec2.internal     NotReady                      <none>   29m   v1.30.0-eks-036c24b
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

```bash
kubectl get pod
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get pod
NAME                                READY   STATUS        RESTARTS   AGE
nginx-deployment-67c44fbdf6-4cw8v   0/1     Pending       0          24s
nginx-deployment-67c44fbdf6-9khhz   1/1     Running       0          9m24s
nginx-deployment-67c44fbdf6-wl2vf   1/1     Running       0          9m25s
nginx-deployment-67c44fbdf6-zftn7   1/1     Terminating   0          9m24s
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

In some minutes, the EKS cluster comes back to its original state.

<details markdown=1>
<summary markdown="span">kubectl get node</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get node
NAME                             STATUS                        ROLES    AGE   VERSION
ip-192-168-11-243.ec2.internal   Ready                         <none>   61s   v1.30.0-eks-036c24b
ip-192-168-24-195.ec2.internal   NotReady,SchedulingDisabled   <none>   32m   v1.30.0-eks-036c24b
ip-192-168-36-113.ec2.internal   NotReady,SchedulingDisabled   <none>   32m   v1.30.0-eks-036c24b
ip-192-168-49-228.ec2.internal   Ready                         <none>   3m    v1.30.0-eks-036c24b
ip-192-168-51-2.ec2.internal     NotReady                      <none>   32m   v1.30.0-eks-036c24b
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$
```
</details><br />

<details markdown=1>
<summary markdown="span">kubectl get pod</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get pod
NAME                                READY   STATUS        RESTARTS   AGE
nginx-deployment-67c44fbdf6-4cw8v   1/1     Running       0          3m35s
nginx-deployment-67c44fbdf6-7b8p8   1/1     Running       0          96s
nginx-deployment-67c44fbdf6-9khhz   1/1     Terminating   0          12m
nginx-deployment-67c44fbdf6-wl2vf   1/1     Running       0          12m
nginx-deployment-67c44fbdf6-zftn7   1/1     Terminating   0          12m
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

<details markdown=1>
<summary markdown="span">kubectl get node</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get node
NAME                             STATUS                        ROLES    AGE     VERSION
ip-192-168-11-243.ec2.internal   Ready                         <none>   117s    v1.30.0-eks-036c24b
ip-192-168-24-195.ec2.internal   NotReady,SchedulingDisabled   <none>   33m     v1.30.0-eks-036c24b
ip-192-168-36-113.ec2.internal   NotReady,SchedulingDisabled   <none>   33m     v1.30.0-eks-036c24b
ip-192-168-49-228.ec2.internal   Ready                         <none>   3m56s   v1.30.0-eks-036c24b
ip-192-168-51-2.ec2.internal     NotReady,SchedulingDisabled   <none>   33m     v1.30.0-eks-036c24b
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

<details markdown=1>
<summary markdown="span">kubectl get pod</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get pod
NAME                                READY   STATUS        RESTARTS   AGE
nginx-deployment-67c44fbdf6-4cw8v   1/1     Running       0          4m28s
nginx-deployment-67c44fbdf6-4s9pg   1/1     Running       0          28s
nginx-deployment-67c44fbdf6-7b8p8   1/1     Running       0          2m29s
nginx-deployment-67c44fbdf6-9khhz   1/1     Terminating   0          13m
nginx-deployment-67c44fbdf6-wl2vf   1/1     Terminating   0          13m
nginx-deployment-67c44fbdf6-zftn7   1/1     Terminating   0          13m
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$
```
</details><br />

<details markdown=1>
<summary markdown="span">kubectl get node</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get node
NAME                             STATUS                        ROLES    AGE     VERSION
ip-192-168-11-243.ec2.internal   Ready                         <none>   3m30s   v1.30.0-eks-036c24b
ip-192-168-24-195.ec2.internal   NotReady,SchedulingDisabled   <none>   35m     v1.30.0-eks-036c24b
ip-192-168-36-113.ec2.internal   NotReady,SchedulingDisabled   <none>   35m     v1.30.0-eks-036c24b
ip-192-168-47-23.ec2.internal    Ready                         <none>   78s     v1.30.0-eks-036c24b
ip-192-168-49-228.ec2.internal   Ready                         <none>   5m29s   v1.30.0-eks-036c24b
ip-192-168-51-2.ec2.internal     NotReady,SchedulingDisabled   <none>   35m     v1.30.0-eks-036c24b
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$
```
</details><br />

<details markdown=1>
<summary markdown="span">kubectl get pod</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ kubectl get pod
NAME                                READY   STATUS        RESTARTS   AGE
nginx-deployment-67c44fbdf6-4cw8v   1/1     Running       0          6m6s
nginx-deployment-67c44fbdf6-4s9pg   1/1     Running       0          2m6s
nginx-deployment-67c44fbdf6-7b8p8   1/1     Running       0          4m7s
nginx-deployment-67c44fbdf6-9khhz   1/1     Terminating   0          15m
nginx-deployment-67c44fbdf6-wl2vf   1/1     Terminating   0          15m
nginx-deployment-67c44fbdf6-zftn7   1/1     Terminating   0          15m
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$
```
</details><br />

```bash
curl <DNS name of the ELB>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ curl ab0312c96509e4e0db8df45e1c4a76f4-26133787.us-east-1.elb.amazonaws.com
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

## Delete the EKS cluster

```bash
eksctl delete cluster dev-cluster
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ eksctl delete cluster dev-cluster
2024-07-16 04:55:04 [ℹ]  deleting EKS cluster "dev-cluster"
2024-07-16 04:55:05 [ℹ]  will drain 0 unmanaged nodegroup(s) in cluster "dev-cluster"
2024-07-16 04:55:05 [ℹ]  starting parallel draining, max in-flight of 1
2024-07-16 04:55:05 [✖]  failed to acquire semaphore while waiting for all routines to finish: context canceled
2024-07-16 04:55:05 [ℹ]  deleted 0 Fargate profile(s)
2024-07-16 04:55:05 [✔]  kubeconfig has been updated
2024-07-16 04:55:05 [ℹ]  cleaning up AWS load balancers created by Kubernetes objects of Kind Service or Ingress
2024-07-16 04:55:30 [ℹ]  
2 sequential tasks: { delete nodegroup "standard-workers", delete cluster control plane "dev-cluster" [async] 
}
2024-07-16 04:55:30 [ℹ]  will delete stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 04:55:30 [ℹ]  waiting for stack "eksctl-dev-cluster-nodegroup-standard-workers" to get deleted
2024-07-16 04:55:30 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 04:56:00 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 04:57:00 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 04:58:44 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 05:00:38 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 05:01:11 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 05:02:03 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 05:02:48 [ℹ]  waiting for CloudFormation stack "eksctl-dev-cluster-nodegroup-standard-workers"
2024-07-16 05:02:48 [ℹ]  will delete stack "eksctl-dev-cluster-cluster"
2024-07-16 05:02:48 [✔]  all cluster resources were deleted
[ec2-user@ip-10-0-0-225 Course_EKS-Basics]$ 
```
</details><br />

Using this command CloudFormation will run again and will take a few minutes to delete all resources involved.

![]({{ site.baseurl }}/images/2024/07-15-Launching-an-AWS-EKS-Cluster/17-CloudFormation-delete.png)

**NOTE:** Also delete the IAM User **k8-admin**.

## References

- [installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- [get started with EKS](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)
- [GitHub Course_EKS-Basics](https://github.com/ACloudGuru-Resources/Course_EKS-Basics)
- [Kubernetes Cheat Sheet](https://www.pluralsight.com/resources/blog/cloud/kubernetes-cheat-sheet)
- [Docker Hub](https://docs.docker.com/docker-hub/official_images/)
