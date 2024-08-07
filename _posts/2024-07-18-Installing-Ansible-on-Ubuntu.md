---
title: Installing Ansible on Ubuntu 22.04.4 LTS
date: 2024-07-18 16:15:00 -0700
categories: [ANSIBLE, INSTALL]
tags: [ansible]     # TAG names should always be lowercase
---

**NOTE:** To refer to module names in **Ansible 2.10** and later, a **Fully Qualified Collection Name (FQCN)** is used, like `ansible.builtin.command`.
In **Ansible 2.9** and earlier, only the last part of this name was used.

## Installing Ansible from Ubuntu repo

Here is the Ubuntu version where we are Installing Ansible

```bash
cloud_user@553b1e446c1c:~$ cat /etc/*release*
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=22.04
DISTRIB_CODENAME=jammy
DISTRIB_DESCRIPTION="Ubuntu 22.04.4 LTS"
PRETTY_NAME="Ubuntu 22.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.4 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
cloud_user@553b1e446c1c:~$
```

Actually, getting Ansible from the Ubuntu repositories worked just fine.

```bash
sudo apt install ansible
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo apt install ansible
[sudo] password for cloud_user:
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  ieee-data python3-argcomplete python3-dnspython python3-kerberos python3-libcloud python3-lockfile python3-netaddr python3-ntlm-auth python3-packaging python3-pycryptodome
  python3-requests-kerberos python3-requests-ntlm python3-requests-toolbelt python3-selinux python3-simplejson python3-winrm python3-xmltodict
Suggested packages:
  cowsay sshpass python3-sniffio python3-trio python-lockfile-doc ipython3 python-netaddr-docs
The following NEW packages will be installed:
  ansible ieee-data python3-argcomplete python3-dnspython python3-kerberos python3-libcloud python3-lockfile python3-netaddr python3-ntlm-auth python3-packaging python3-pycryptodome
  python3-requests-kerberos python3-requests-ntlm python3-requests-toolbelt python3-selinux python3-simplejson python3-winrm python3-xmltodict
0 upgraded, 18 newly installed, 0 to remove and 14 not upgraded.
Need to get 22.9 MB of archives.
After this operation, 243 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/main amd64 python3-packaging all 21.3-1 [30.7 kB]
Get:2 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy-updates/main amd64 python3-pycryptodome amd64 3.11.0+dfsg1-3ubuntu0.1 [1029 kB]
Get:3 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/main amd64 python3-dnspython all 2.1.0-1ubuntu1 [123 kB]
Get:4 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/main amd64 ieee-data all 20210605.1 [1887 kB]
Get:5 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/main amd64 python3-netaddr all 0.8.0-2 [309 kB]
Get:6 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 ansible all 2.10.7+merged+base+2.10.8+dfsg-1 [17.5 MB]
Get:7 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 python3-argcomplete all 1.8.1-1.5 [27.2 kB]
Get:8 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 python3-kerberos amd64 1.1.14-3.1build5 [23.0 kB]
Get:9 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/main amd64 python3-lockfile all 1:0.12.2-2.2 [14.6 kB]
Get:10 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/main amd64 python3-simplejson amd64 3.17.6-1build1 [54.7 kB]
Get:11 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 python3-libcloud all 3.2.0-2 [1554 kB]
Get:12 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 python3-ntlm-auth all 1.4.0-1 [20.4 kB]
Get:13 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 python3-requests-kerberos all 0.12.0-2 [11.9 kB]
Get:14 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 python3-requests-ntlm all 1.1.0-1.1 [6160 B]
Get:15 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/main amd64 python3-requests-toolbelt all 0.9.1-1 [38.0 kB]
Get:16 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 python3-selinux amd64 3.3-1build2 [159 kB]
Get:17 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 python3-xmltodict all 0.12.0-2 [12.6 kB]
Get:18 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 python3-winrm all 0.3.0-2 [21.7 kB]
Fetched 22.9 MB in 0s (57.6 MB/s)
Selecting previously unselected package python3-packaging.
(Reading database ... 136512 files and directories currently installed.)
Preparing to unpack .../00-python3-packaging_21.3-1_all.deb ...
Unpacking python3-packaging (21.3-1) ...
Selecting previously unselected package python3-pycryptodome.
Preparing to unpack .../01-python3-pycryptodome_3.11.0+dfsg1-3ubuntu0.1_amd64.deb ...
Unpacking python3-pycryptodome (3.11.0+dfsg1-3ubuntu0.1) ...
Selecting previously unselected package python3-dnspython.
Preparing to unpack .../02-python3-dnspython_2.1.0-1ubuntu1_all.deb ...
Unpacking python3-dnspython (2.1.0-1ubuntu1) ...
Selecting previously unselected package ieee-data.
Preparing to unpack .../03-ieee-data_20210605.1_all.deb ...
Unpacking ieee-data (20210605.1) ...
Selecting previously unselected package python3-netaddr.
Preparing to unpack .../04-python3-netaddr_0.8.0-2_all.deb ...
Unpacking python3-netaddr (0.8.0-2) ...
Selecting previously unselected package ansible.
Preparing to unpack .../05-ansible_2.10.7+merged+base+2.10.8+dfsg-1_all.deb ...
Unpacking ansible (2.10.7+merged+base+2.10.8+dfsg-1) ...
Selecting previously unselected package python3-argcomplete.
Preparing to unpack .../06-python3-argcomplete_1.8.1-1.5_all.deb ...
Unpacking python3-argcomplete (1.8.1-1.5) ...
Selecting previously unselected package python3-kerberos.
Preparing to unpack .../07-python3-kerberos_1.1.14-3.1build5_amd64.deb ...
Unpacking python3-kerberos (1.1.14-3.1build5) ...
Selecting previously unselected package python3-lockfile.
Preparing to unpack .../08-python3-lockfile_1%3a0.12.2-2.2_all.deb ...
Unpacking python3-lockfile (1:0.12.2-2.2) ...
Selecting previously unselected package python3-simplejson.
Preparing to unpack .../09-python3-simplejson_3.17.6-1build1_amd64.deb ...
Unpacking python3-simplejson (3.17.6-1build1) ...
Selecting previously unselected package python3-libcloud.
Preparing to unpack .../10-python3-libcloud_3.2.0-2_all.deb ...
Unpacking python3-libcloud (3.2.0-2) ...
Selecting previously unselected package python3-ntlm-auth.
Preparing to unpack .../11-python3-ntlm-auth_1.4.0-1_all.deb ...
Unpacking python3-ntlm-auth (1.4.0-1) ...
Selecting previously unselected package python3-requests-kerberos.
Preparing to unpack .../12-python3-requests-kerberos_0.12.0-2_all.deb ...
Unpacking python3-requests-kerberos (0.12.0-2) ...
Selecting previously unselected package python3-requests-ntlm.
Preparing to unpack .../13-python3-requests-ntlm_1.1.0-1.1_all.deb ...
Unpacking python3-requests-ntlm (1.1.0-1.1) ...
Selecting previously unselected package python3-requests-toolbelt.
Preparing to unpack .../14-python3-requests-toolbelt_0.9.1-1_all.deb ...
Unpacking python3-requests-toolbelt (0.9.1-1) ...
Selecting previously unselected package python3-selinux.
Preparing to unpack .../15-python3-selinux_3.3-1build2_amd64.deb ...
Unpacking python3-selinux (3.3-1build2) ...
Selecting previously unselected package python3-xmltodict.
Preparing to unpack .../16-python3-xmltodict_0.12.0-2_all.deb ...
Unpacking python3-xmltodict (0.12.0-2) ...
Selecting previously unselected package python3-winrm.
Preparing to unpack .../17-python3-winrm_0.3.0-2_all.deb ...
Unpacking python3-winrm (0.3.0-2) ...
Setting up python3-lockfile (1:0.12.2-2.2) ...
Setting up python3-requests-toolbelt (0.9.1-1) ...
Setting up python3-ntlm-auth (1.4.0-1) ...
Setting up python3-pycryptodome (3.11.0+dfsg1-3ubuntu0.1) ...
Setting up python3-kerberos (1.1.14-3.1build5) ...
Setting up python3-simplejson (3.17.6-1build1) ...
Setting up python3-xmltodict (0.12.0-2) ...
Setting up python3-packaging (21.3-1) ...
Setting up python3-requests-kerberos (0.12.0-2) ...
Setting up ieee-data (20210605.1) ...
Setting up python3-dnspython (2.1.0-1ubuntu1) ...
Setting up python3-selinux (3.3-1build2) ...
Setting up python3-argcomplete (1.8.1-1.5) ...
Setting up python3-requests-ntlm (1.1.0-1.1) ...
Setting up python3-libcloud (3.2.0-2) ...
Setting up python3-netaddr (0.8.0-2) ...
Setting up python3-winrm (0.3.0-2) ...
Setting up ansible (2.10.7+merged+base+2.10.8+dfsg-1) ...
Processing triggers for man-db (2.10.2-1) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
cloud_user@553b1e446c1c:~$
```
</details><br />

We end up with `ansible 2.10.8` installed

```bash
cloud_user@553b1e446c1c:~$ ansible --version
ansible 2.10.8
  config file = None
  configured module search path = ['/home/cloud_user/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Nov 20 2023, 15:14:05) [GCC 11.4.0]
cloud_user@553b1e446c1c:~$
```

## Issue with Ansible 3.10 & target running Python 3.12

```bash
cloud_user@553b1e446c1c:~$ ansible -i inventory ubuntu1 \
  -m ansible.builtin.user \
  -a "name=ansible create_home=yes" \
  -u cloud_user \
  -b \
  -k \
  -K
SSH password:
BECOME password[defaults to SSH password]:
ubuntu1 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "module_stderr": "Shared connection to ec2-3-86-82-111.compute-1.amazonaws.com closed.\r\n",
    "module_stdout": "\r\nTraceback (most recent call last):\r\n  File \"/home/cloud_user/.ansible/tmp/ansible-tmp-1721363268.8297591-2182-19563260793955/AnsiballZ_user.py\", line 102, in <module>\r\n    _ansiballz_main()\r\n  File \"/home/cloud_user/.ansible/tmp/ansible-tmp-1721363268.8297591-2182-19563260793955/AnsiballZ_user.py\", line 94, in _ansiballz_main\r\n    invoke_module(zipped_mod, temp_path, ANSIBALLZ_PARAMS)\r\n  File \"/home/cloud_user/.ansible/tmp/ansible-tmp-1721363268.8297591-2182-19563260793955/AnsiballZ_user.py\", line 37, in invoke_module\r\n    from ansible.module_utils import basic\r\n  File \"/tmp/ansible_ansible.builtin.user_payload_8nlnjjnu/ansible_ansible.builtin.user_payload.zip/ansible/module_utils/basic.py\", line 176, in <module>\r\nModuleNotFoundError: No module named 'ansible.module_utils.six.moves'\r\n",
    "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error",
    "rc": 1
}
cloud_user@553b1e446c1c:~$
```

On the target Server

```bash
cloud_user@ip-XXX-XX-XX-XXX:~$ python3 --version
Python 3.12.3
cloud_user@ip-XXX-XX-XX-XXX:~$
```

The issue is found in google when looking with `ModuleNotFoundError: No module named 'ansible.module_utils.six.moves`.

There are several hits but in conclusion, there is an incompatibility in Ansible vs Python versions.

See: [Ansible 2.9.13 not working with python 3.12? #81946](https://github.com/ansible/ansible/issues/81946)

It seems the best is to upgrade Ansible and move on...

## Installing the latest Ansible version

We need to add the `ppa:ansible/ansible` repo

```bash
sudo apt update
sudo apt install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo apt update
[sudo] password for cloud_user:
Hit:1 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]
Hit:3 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy-backports InRelease
Get:4 http://security.ubuntu.com/ubuntu jammy-security InRelease [129 kB]
Hit:5 https://download.docker.com/linux/ubuntu jammy InRelease
Get:6 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy-updates/universe amd64 Packages [1108 kB]
Fetched 1365 kB in 1s (1057 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
17 packages can be upgraded. Run 'apt list --upgradable' to see them.
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ sudo apt install software-properties-common
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
software-properties-common is already the newest version (0.99.22.9).
software-properties-common set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 17 not upgraded.
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ sudo apt-add-repository ppa:ansible/ansible
Repository: 'deb https://ppa.launchpadcontent.net/ansible/ansible/ubuntu/ jammy main'
Description:
Ansible is a radically simple IT automation platform that makes your applications and systems easier to deploy. Avoid writing scripts or custom code to deploy and update your applications— automate in a language that approaches plain English, using SSH, with no agents to install on remote systems.

http://ansible.com/

If you face any issues while installing Ansible PPA, file an issue here:
https://github.com/ansible-community/ppa/issues
More info: https://launchpad.net/~ansible/+archive/ubuntu/ansible
Adding repository.
Press [ENTER] to continue or Ctrl-c to cancel.
Adding deb entry to /etc/apt/sources.list.d/ansible-ubuntu-ansible-jammy.list
Adding disabled deb-src entry to /etc/apt/sources.list.d/ansible-ubuntu-ansible-jammy.list
Adding key to /etc/apt/trusted.gpg.d/ansible-ubuntu-ansible.gpg with fingerprint 6125E2A8C77F2818FB7BD15B93C4A3FD7BB9C367
Hit:1 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy InRelease
Hit:2 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy-updates InRelease
Hit:3 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy-backports InRelease
Hit:4 http://security.ubuntu.com/ubuntu jammy-security InRelease
Hit:5 https://download.docker.com/linux/ubuntu jammy InRelease
Get:6 https://ppa.launchpadcontent.net/ansible/ansible/ubuntu jammy InRelease [18.0 kB]
Get:7 https://ppa.launchpadcontent.net/ansible/ansible/ubuntu jammy/main amd64 Packages [1120 B]
Get:8 https://ppa.launchpadcontent.net/ansible/ansible/ubuntu jammy/main Translation-en [752 B]
Fetched 19.9 kB in 1s (14.1 kB/s)
Reading package lists... Done
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ sudo apt install ansible
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  ieee-data python3-argcomplete python3-dnspython python3-libcloud python3-lockfile python3-netaddr python3-pycryptodome python3-requests-toolbelt python3-selinux
  python3-simplejson
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  ansible-core python3-nacl python3-paramiko python3-resolvelib
Suggested packages:
  python-nacl-doc python3-gssapi python3-invoke
The following NEW packages will be installed:
  ansible-core python3-nacl python3-paramiko python3-resolvelib
The following packages will be upgraded:
  ansible
1 upgraded, 4 newly installed, 0 to remove and 17 not upgraded.
Need to get 18.2 MB of archives.
After this operation, 2266 kB disk space will be freed.
Do you want to continue? [Y/n] y
Get:1 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 python3-resolvelib all 0.8.1-1 [23.6 kB]
Get:2 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy/main amd64 python3-nacl amd64 1.5.0-2 [63.1 kB]
Get:3 http://us-east-2.ec2.archive.ubuntu.com/ubuntu jammy-updates/main amd64 python3-paramiko all 2.9.3-0ubuntu1.2 [134 kB]
Get:4 https://ppa.launchpadcontent.net/ansible/ansible/ubuntu jammy/main amd64 ansible-core all 2.16.9-1ppa~jammy [1032 kB]
Get:5 https://ppa.launchpadcontent.net/ansible/ansible/ubuntu jammy/main amd64 ansible all 9.8.0-1ppa~jammy [17.0 MB]
Fetched 18.2 MB in 2s (10.7 MB/s)
Selecting previously unselected package python3-resolvelib.
(Reading database ... 174958 files and directories currently installed.)
Preparing to unpack .../python3-resolvelib_0.8.1-1_all.deb ...
Unpacking python3-resolvelib (0.8.1-1) ...
Selecting previously unselected package ansible-core.
Preparing to unpack .../ansible-core_2.16.9-1ppa~jammy_all.deb ...
Unpacking ansible-core (2.16.9-1ppa~jammy) ...
dpkg: error processing archive /var/cache/apt/archives/ansible-core_2.16.9-1ppa~jammy_all.deb (--unpack):
 trying to overwrite '/usr/bin/ansible', which is also in package ansible 2.10.7+merged+base+2.10.8+dfsg-1
dpkg-deb: error: paste subprocess was killed by signal (Broken pipe)
Preparing to unpack .../ansible_9.8.0-1ppa~jammy_all.deb ...
Unpacking ansible (9.8.0-1ppa~jammy) over (2.10.7+merged+base+2.10.8+dfsg-1) ...
Selecting previously unselected package python3-nacl.
Preparing to unpack .../python3-nacl_1.5.0-2_amd64.deb ...
Unpacking python3-nacl (1.5.0-2) ...
Selecting previously unselected package python3-paramiko.
Preparing to unpack .../python3-paramiko_2.9.3-0ubuntu1.2_all.deb ...
Unpacking python3-paramiko (2.9.3-0ubuntu1.2) ...
Errors were encountered while processing:
 /var/cache/apt/archives/ansible-core_2.16.9-1ppa~jammy_all.deb
needrestart is being skipped since dpkg has failed
E: Sub-process /usr/bin/dpkg returned an error code (1)
cloud_user@553b1e446c1c:~$
```
</details><br />

I faced this error when running `sudo apt install ansible`. It seems related to the fact that I had already Ansible installed.

```bash
Errors were encountered while processing:
 /var/cache/apt/archives/ansible-core_2.16.9-1ppa~jammy_all.deb
needrestart is being skipped since dpkg has failed
E: Sub-process /usr/bin/dpkg returned an error code (1)
cloud_user@553b1e446c1c:~$
```

So, I tried again:

```bash
cloud_user@553b1e446c1c:~$ sudo apt install ansible -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ansible is already the newest version (9.8.0-1ppa~jammy).
You might want to run 'apt --fix-broken install' to correct these.
The following packages have unmet dependencies:
 ansible : Depends: ansible-core but it is not going to be installed
E: Unmet dependencies. Try 'apt --fix-broken install' with no packages (or specify a solution).
cloud_user@553b1e446c1c:~$
```

It did not work but it gave me a next step, which was try with `apt --fix-broken install`.

So, I did that.

```bash
sudo apt --fix-broken install
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo apt --fix-broken install
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Correcting dependencies... Done
The following packages were automatically installed and are no longer required:
  ieee-data python3-argcomplete python3-dnspython python3-libcloud python3-lockfile python3-netaddr python3-pycryptodome python3-requests-toolbelt python3-selinux
  python3-simplejson
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  ansible-core
The following NEW packages will be installed:
  ansible-core
0 upgraded, 1 newly installed, 0 to remove and 17 not upgraded.
4 not fully installed or removed.
Need to get 0 B/1032 kB of archives.
After this operation, 6340 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
(Reading database ... 162684 files and directories currently installed.)
Preparing to unpack .../ansible-core_2.16.9-1ppa~jammy_all.deb ...
Unpacking ansible-core (2.16.9-1ppa~jammy) ...
Setting up python3-resolvelib (0.8.1-1) ...
Setting up ansible-core (2.16.9-1ppa~jammy) ...
Setting up ansible (9.8.0-1ppa~jammy) ...
Setting up python3-nacl (1.5.0-2) ...
Setting up python3-paramiko (2.9.3-0ubuntu1.2) ...
Processing triggers for man-db (2.10.2-1) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
cloud_user@553b1e446c1c:~$
```
</details><br />

This time, it worked and I ended up with `ansible 2.16.9` installed

```bash
cloud_user@553b1e446c1c:~$ ansible --version
ansible [core 2.16.9]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/cloud_user/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/cloud_user/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Nov 20 2023, 15:14:05) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.0.3
  libyaml = True
cloud_user@553b1e446c1c:~$
```

This time my Ansible test worked just fine.

```bash
cloud_user@553b1e446c1c:~$ ansible -i inventory ubuntu1 \
  -m ansible.builtin.user \
  -a "name=ansible create_home=yes" \
  -u cloud_user \
  -b \
  -k \
  -K
SSH password:
BECOME password[defaults to SSH password]:
ubuntu1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "comment": "",
    "create_home": true,
    "group": 1002,
    "home": "/home/ansible",
    "name": "ansible",
    "shell": "/bin/sh",
    "state": "present",
    "system": false,
    "uid": 1002
}
cloud_user@553b1e446c1c:~$
```

## References

- [How to check Ansible version on Linux/Unix](https://www.cyberciti.biz/faq/command-to-see-ansible-version-check-on-linux-unix/)
- [Releases and maintenance](https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-core-support-matrix)
- [Error when running ansible-playbook from ubuntu20.04 ansible server to ubuntu24.04](https://ubuntuforums.org/showthread.php?t=2497608)