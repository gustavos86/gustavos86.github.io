---
title: Installing asdf, Node.js and AWS CDK on Ubuntu 24.04 LTS
date: 2024-12-08 11:00:00 -0700
categories: [AWS, AWS CDK]
tags: [aws, aws cdk]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/aws.png)

**NOTE**: All configurations were taken from a lab environment.

This guide is based on **Ubuntu 24.04 LTS**

<details markdown=1>
<summary markdown="span">System information</summary>
```
ubuntu@ip-172-31-25-204:~$  cat /etc/*release*
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=24.04
DISTRIB_CODENAME=noble
DISTRIB_DESCRIPTION="Ubuntu 24.04.1 LTS"
PRETTY_NAME="Ubuntu 24.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.1 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
ubuntu@ip-172-31-25-204:~$
```
</details><br />

This post includes the commands to install:

1. asdf - command line interface (CLI) tool that helps manage runtime versions for multiple programming languages.
2. Node.js - Node.js is an open-source, cross-platform JavaScript runtime environment that allows developers to execute JavaScript code outside of a web browser.
3. AWS CDK - AWS Cloud Development Kit (AWS CDK) is an open-source software development framework for defining cloud infrastructure in code and provisioning it through AWS CloudFormation.

## Install asdf

To install asdf, run the following commands. Requires Git.

```bash
# Clone the repo
git clone https://github.com/asdf-vm/asdf.git ~/.asdf
chmod +x ~/.asdf/asdf.sh ~/.asdf/completions/asdf.bash

# Set variables in bash
echo '' >> .bashrc
echo '. $HOME/.asdf/asdf.sh' >> .bashrc
echo '. $HOME/.asdf/completions/asdf.bash' >> .bashrc

# reload .bashrc configuration
source .bashrc
```

Verify asdf was successfully installed

```bash
asdf version
```

<details markdown=1>
<summary markdown="span">asdf installation outputs</summary>

```bash
ubuntu@ip-172-31-25-204:~$ git clone https://github.com/asdf-vm/asdf.git ~/.asdf
Cloning into '/home/ubuntu/.asdf'...
remote: Enumerating objects: 8802, done.
remote: Counting objects: 100% (1909/1909), done.
remote: Compressing objects: 100% (348/348), done.
remote: Total 8802 (delta 1660), reused 1642 (delta 1557), pack-reused 6893 (from 1)
Receiving objects: 100% (8802/8802), 3.25 MiB | 5.12 MiB/s, done.
Resolving deltas: 100% (5282/5282), done.
ubuntu@ip-172-31-25-204:~$

ubuntu@ip-172-31-25-204:~$ chmod +x ~/.asdf/asdf.sh ~/.asdf/completions/asdf.bash
ubuntu@ip-172-31-25-204:~$

ubuntu@ip-172-31-25-204:~$ echo '' >> .bashrc
ubuntu@ip-172-31-25-204:~$ echo '. $HOME/.asdf/asdf.sh' >> .bashrc
ubuntu@ip-172-31-25-204:~$ echo '. $HOME/.asdf/completions/asdf.bash' >> .bashrc

ubuntu@ip-172-31-25-204:~$ asdf version
v0.14.1-c5116dc
ubuntu@ip-172-31-25-204:~$
```
</details><br />

## Install Node.js with asdf

To install Node.js, run the following commands.

```bash
# Installs Node.js plugin in asdf
asdf plugin-add nodejs

# Installs Node.js using asdf
asdf install nodejs 20.16.0

# Activates Node.js using asdf
asdf global nodejs 20.16.0
```

Verify Node.js was successfully installed

```bash
node --version
```

Optional. List the available Node.js versions in asdf

```bash
asdf list all nodejs
```

<details markdown=1>
<summary markdown="span">Node.js installation outputs</summary>

```bash
ubuntu@ip-172-31-25-204:~$ asdf plugin-add nodejs
initializing plugin repository...Cloning into '/home/ubuntu/.asdf/repository'...
remote: Enumerating objects: 6174, done.
remote: Counting objects: 100% (1852/1852), done.
remote: Compressing objects: 100% (67/67), done.
remote: Total 6174 (delta 1824), reused 1795 (delta 1785), pack-reused 4322 (from 1)
Receiving objects: 100% (6174/6174), 1.42 MiB | 3.38 MiB/s, done.
Resolving deltas: 100% (3396/3396), done.
ubuntu@ip-172-31-25-204:~$

ubuntu@ip-172-31-25-204:~$ asdf install nodejs 20.16.0
Trying to update node-build... ok
To follow progress, use 'tail -f /tmp/node-build.20241208231010.1582.log' or pass --verbose
Downloading node-v20.16.0-linux-x64.tar.gz...
-> https://nodejs.org/dist/v20.16.0/node-v20.16.0-linux-x64.tar.gz

WARNING: node-v20.16.0-linux-x64 is in LTS Maintenance mode and nearing its end of life.
It only receives *critical* security updates, *critical* bug fixes and documentation updates.

Installing node-v20.16.0-linux-x64...
Installed node-v20.16.0-linux-x64 to /home/ubuntu/.asdf/installs/nodejs/20.16.0

ubuntu@ip-172-31-25-204:~$

ubuntu@ip-172-31-25-204:~$ asdf global nodejs 20.16.0
ubuntu@ip-172-31-25-204:~$

ubuntu@ip-172-31-25-204:~$ node --version
v20.16.0
ubuntu@ip-172-31-25-204:~$
```
</details><br />

## Install AWS CDK

To install the AWS CDK, run the following commands.
And better to have [Typescript](https://www.typescriptlang.org/) installed as well.

```bash
npm install -g aws-cdk
npm install -g typescript
```

Verify AWS-CDK was successfully installed

```bash
cdk --version
```

<details markdown=1>
<summary markdown="span">AWS-CDK installation outputs</summary>

```bash
ubuntu@ip-172-31-25-204:~$ npm install -g aws-cdk

added 1 package in 1s
npm notice
npm notice New minor version of npm available! 10.8.1 -> 10.9.2
npm notice Changelog: https://github.com/npm/cli/releases/tag/v10.9.2
npm notice To update run: npm install -g npm@10.9.2
npm notice
Reshimming asdf nodejs...
ubuntu@ip-172-31-25-204:~$

ubuntu@ip-172-31-25-204:~$ npm install -g typescript

added 1 package in 1s
Reshimming asdf nodejs...
ubuntu@ip-172-31-25-204:~$

ubuntu@ip-172-31-25-204:~$ cdk --version
2.172.0 (build 0f666c5)
ubuntu@ip-172-31-25-204:~$
```
</details><br />
