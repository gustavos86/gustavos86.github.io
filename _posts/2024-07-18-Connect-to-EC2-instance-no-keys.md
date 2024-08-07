---
title: Connect to an EC2 instance without SSH keys
date: 2024-07-18 17:00:00 -0700
categories: [AWS, EC2]
tags: [aws, ec2 ssh]     # TAG names should always be lowercase
---

# This is absolutelly not recommended for obvious security purposes. Only a good idea in controlled lab environments, etc. Proceed with caution.

This worked in an EC2 instance launched with `Ubuntu 24.04 LTS`

<details markdown=1>
<summary markdown="span">cat /etc/*release*</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ cat /etc/*release*
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=24.04
DISTRIB_CODENAME=noble
DISTRIB_DESCRIPTION="Ubuntu 24.04 LTS"
PRETTY_NAME="Ubuntu 24.04 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

## 1. Add an user

```bash
sudo adduser <user>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo adduser cloud_user
info: Adding user `cloud_user' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `cloud_user' (1001) ...
info: Adding new user `cloud_user' (1001) with group `cloud_user (1001)' ...
info: Creating home directory `/home/cloud_user' ...
info: Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for cloud_user
Enter the new value, or press ENTER for the default
        Full Name []: 
        Room Number []: 
        Work Phone []: 
        Home Phone []: 
        Other []: 
Is the information correct? [Y/n] 
info: Adding new user `cloud_user' to supplemental / extra groups `users' ...
info: Adding user `cloud_user' to group `users' ...
ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

## 2. Modify the file `60-cloudimg-settings.conf`

Edit file `/etc/ssh/sshd_config.d/60-cloudimg-settings.conf` and set `PasswordAuthentication yes`.
Use the text edit of your preference. Make sure you use `sudo` privileges.

```bash
sudo vim /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo vim /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
ubuntu@ip-XXX-XX-XX-XXX:~$ 
ubuntu@ip-XXX-XX-XX-XXX:~$ cat /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
PasswordAuthentication yes
ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

## 3. Restart the `ssh` process

```bash
sudo systemctl restart ssh
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/ssh.service.d
             └─ec2-instance-connect.conf
     Active: active (running) since Thu 2024-07-18 23:48:57 UTC; 9min ago
TriggeredBy: ● ssh.socket
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 977 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 979 (sshd)
      Tasks: 1 (limit: 1130)
     Memory: 4.7M (peak: 8.2M)
        CPU: 1.922s
     CGroup: /system.slice/ssh.service
             └─979 "sshd: /usr/sbin/sshd -D -o AuthorizedKeysCommand /usr/share/ec2-instance-connect/eic_run_authorized_keys %u %f -o AuthorizedKeysCommandUser ec2-instance-connect [listener] 0 of 10-100 startups"

Jul 18 23:48:57 ip-XXX-XX-XX-XXX systemd[1]: Starting ssh.service - OpenBSD Secure Shell server...
Jul 18 23:48:57 ip-XXX-XX-XX-XXX sshd[979]: Server listening on :: port 22.
Jul 18 23:48:57 ip-XXX-XX-XX-XXX systemd[1]: Started ssh.service - OpenBSD Secure Shell server.
Jul 18 23:48:58 ip-XXX-XX-XX-XXX ec2-instance-connect[1111]: Querying EC2 Instance Connect keys for matching fingerprint: SHA256:odqvnNYj/lVYhOZ5a1iWfPRbGglMJWMU8TGxdvp0q/8
Jul 18 23:48:58 ip-XXX-XX-XX-XXX ec2-instance-connect[1143]: Providing ssh key from EC2 Instance Connect with fingerprint: SHA256:odqvnNYj/lVYhOZ5a1iWfPRbGglMJWMU8TGxdvp0q/8, request-id: 1f3a3cfe-0820-47f5-b288-bda9b27488e0, for IAM principal: arn:aws:iam::211125786036:user/>
Jul 18 23:49:00 ip-XXX-XX-XX-XXX ec2-instance-connect[1282]: Querying EC2 Instance Connect keys for matching fingerprint: SHA256:odqvnNYj/lVYhOZ5a1iWfPRbGglMJWMU8TGxdvp0q/8
Jul 18 23:49:00 ip-XXX-XX-XX-XXX ec2-instance-connect[1314]: Providing ssh key from EC2 Instance Connect with fingerprint: SHA256:odqvnNYj/lVYhOZ5a1iWfPRbGglMJWMU8TGxdvp0q/8, request-id: 1f3a3cfe-0820-47f5-b288-bda9b27488e0, for IAM principal: arn:aws:iam::211125786036:user/>
Jul 18 23:49:00 ip-XXX-XX-XX-XXX sshd[980]: Accepted publickey for ubuntu from 18.206.107.29 port 14751 ssh2: ED25519 SHA256:odqvnNYj/lVYhOZ5a1iWfPRbGglMJWMU8TGxdvp0q/8
Jul 18 23:49:00 ip-XXX-XX-XX-XXX sshd[980]: pam_unix(sshd:session): session opened for user ubuntu(uid=1000) by ubuntu(uid=0)
ubuntu@ip-XXX-XX-XX-XXX:~$ 
ubuntu@ip-XXX-XX-XX-XXX:~$ 
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo systemctl restart ssh
ubuntu@ip-XXX-XX-XX-XXX:~$ 
ubuntu@ip-XXX-XX-XX-XXX:~$ sudo systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/ssh.service.d
             └─ec2-instance-connect.conf
     Active: active (running) since Thu 2024-07-18 23:58:45 UTC; 2s ago
TriggeredBy: ● ssh.socket
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 1575 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 1577 (sshd)
      Tasks: 1 (limit: 1130)
     Memory: 1.2M (peak: 1.3M)
        CPU: 23ms
     CGroup: /system.slice/ssh.service
             └─1577 "sshd: /usr/sbin/sshd -D -o AuthorizedKeysCommand /usr/share/ec2-instance-connect/eic_run_authorized_keys %u %f -o AuthorizedKeysCommandUser ec2-instance-connect [listener] 0 of 10-100 startups"

Jul 18 23:58:45 ip-XXX-XX-XX-XXX systemd[1]: Starting ssh.service - OpenBSD Secure Shell server...
Jul 18 23:58:45 ip-XXX-XX-XX-XXX sshd[1577]: Server listening on :: port 22.
Jul 18 23:58:45 ip-XXX-XX-XX-XXX systemd[1]: Started ssh.service - OpenBSD Secure Shell server.
ubuntu@ip-XXX-XX-XX-XXX:~$ 
```
</details><br />

## 4. Connect to the EC2 instance by using the password now

```bash
ssh <user>@<public_ip>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
$ ssh cloud_user@<public_ip>
cloud_user@<public_ip>'s password:
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.8.0-1009-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro
...
```
</details><br />

## Optional. Add user to sudo group

Use with caution! This will allow the user to perform `sudo` commands on the host.

```bash
sudo adduser <user> sudo
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
ubuntu@ip-172-31-95-109:~$ sudo adduser cloud_user sudo
info: Adding user `cloud_user' to group `sudo' ...
ubuntu@ip-172-31-95-109:~$ 
```
</details><br />

## References

- [How to Use Sudo and the Sudoers File](https://www.hostinger.com/tutorials/sudo-and-the-sudoers-file/)