---
title: Using Bash scripts to simulate events in FRR
date: 2024-08-03 15:00:00 -0700
categories: [FRR, ]
tags: [frr]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/frr_logo.svg)

## Bounce interfaces every certain time

The following `bash` script can be used to bounce one or more interfaces every certain time. The time is controlled by the constants **MINWAIT** and **MAXWAIT**

```bash
MINWAIT=10
MAXWAIT=20
for n in $(seq 0 1000);
do
    if [ $((n % 2)) -eq 0 ]; then
        sudo ip link set eth0 down
        echo "sudo ip link set eth0 down"
    else
        sudo ip link set eth0 up
        echo "sudo ip link set eth0 up"
    fi
    
    backoff_time=$(($RANDOM%($MAXWAIT-$MINWAIT+1)+$MINWAIT))
    echo "sleeping for" $backoff_time "seconds"
    sleep $backoff_time
done
```

Execute the command in the `bash` shell of an FRR Linux instance

```bash
cloud_user@553b1e446c1c:~$ docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED       STATUS         PORTS     NAMES
b34035faae0d   ksator/frr:1.0   "/bin/bash /etc/frr/…"   4 hours ago   Up 7 minutes             frr100
37ece5e00ac0   ksator/frr:1.0   "/bin/bash /etc/frr/…"   4 hours ago   Up 7 minutes             frr200
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ docker exec -it frr100 bash
root@b34035faae0d:/#

root@b34035faae0d:/# MINWAIT=10
root@b34035faae0d:/# MAXWAIT=20
root@b34035faae0d:/# for n in $(seq 0 1000);
> do
>     if [ $((n % 2)) -eq 0 ]; then
>         sudo ip link set eth0 down
>         echo "sudo ip link set eth0 down"
>     else
>         sudo ip link set eth0 up
>         echo "sudo ip link set eth0 up"
>     fi
>
>     backoff_time=$(($RANDOM%($MAXWAIT-$MINWAIT+1)+$MINWAIT))
>     echo "sleeping for" $backoff_time "seconds"
>     sleep $backoff_time
> done
sudo ip link set eth0 down
sleeping for 19 seconds
sudo ip link set eth0 up
sleeping for 17 seconds
sudo ip link set eth0 down
sleeping for 17 seconds
sudo ip link set eth0 up
sleeping for 20 seconds
sudo ip link set eth0 down
sleeping for 11 seconds
sudo ip link set eth0 up
sleeping for 14 seconds
sudo ip link set eth0 down
sleeping for 14 seconds
^C
root@b34035faae0d:/#
```

Results can be monitored using the `watch` command. Since the `bash` is occupied by the script, we can open a 2nd connection to the same FRR instance

```bash
watch -n0 -diff 'ip link show eth0'
```

Outputs:

```bash
root@b34035faae0d:/# watch -n0 -diff 'ip link show eth0'

Every 0.1s: ip link showeth0                                                                                                                                                                                 b34035faae0d: Sat Aug  3 22:29:26 2024

12: eth0@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:c0:a8:01:64 brd ff:ff:ff:ff:ff:ff link-netnsid 0

...

Every 0.1s: ip link show eth0                                                                                                                                                                                 b34035faae0d: Sat Aug  3 22:36:49 2024

12: eth0@if13: <BROADCAST,MULTICAST> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default
    link/ether 02:42:c0:a8:01:64 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

## Bounce BGP sessions every certain time

A similar `bash` script can be used to bounce BGP sessions for testing purposes.

```bash
MINWAIT=10
MAXWAIT=20
for n in $(seq 0 1000);
do
    if [ $((n % 2)) -eq 0 ]; then
        sudo vtysh -c 'configure terminal' -c 'router bgp 65100' -c 'neighbor 192.168.1.200 shutdown'
        echo "neighbor 192.168.1.200 admin down"
    else
        sudo vtysh -c 'configure terminal' -c 'router bgp 65100' -c 'no neighbor 192.168.1.200 shutdown'
        echo "neighbor 192.168.1.200 admin up"
    fi

    backoff_time=$(($RANDOM%($MAXWAIT-$MINWAIT+1)+$MINWAIT))
    echo "sleeping for" $backoff_time "seconds"
    sleep $backoff_time
done
```

Execute the command in the `bash` shell of an FRR Linux instance

```bash
cloud_user@553b1e446c1c:~$ docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED       STATUS          PORTS     NAMES
b34035faae0d   ksator/frr:1.0   "/bin/bash /etc/frr/…"   4 hours ago   Up 28 minutes             frr100
37ece5e00ac0   ksator/frr:1.0   "/bin/bash /etc/frr/…"   4 hours ago   Up 28 minutes             frr200
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ docker exec -it frr100 bash
root@b34035faae0d:/#

root@b34035faae0d:/# MINWAIT=10
root@b34035faae0d:/# MAXWAIT=20
root@b34035faae0d:/# for n in $(seq 0 1000);
> do
>     if [ $((n % 2)) -eq 0 ]; then
>         sudo vtysh -c 'configure terminal' -c 'router bgp 65100' -c 'neighbor 192.168.1.200 shutdown'
>         echo "neighbor 192.168.1.200 admin down"
>     else
>         sudo vtysh -c 'configure terminal' -c 'router bgp 65100' -c 'no neighbor 192.168.1.200 shutdown'
>         echo "neighbor 192.168.1.200 admin up"
>     fi
>
>     backoff_time=$(($RANDOM%($MAXWAIT-$MINWAIT+1)+$MINWAIT))
>     echo "sleeping for" $backoff_time "seconds"
>     sleep $backoff_time
> done
neighbor 192.168.1.200 admin down
sleeping for 11 seconds

neighbor 192.168.1.200 admin up
sleeping for 15 seconds
neighbor 192.168.1.200 admin down
sleeping for 15 seconds
neighbor 192.168.1.200 admin up
sleeping for 18 seconds
neighbor 192.168.1.200 admin down
sleeping for 13 seconds
neighbor 192.168.1.200 admin up
sleeping for 16 seconds
neighbor 192.168.1.200 admin down
sleeping for 11 seconds
^C
root@b34035faae0d:/#
```

Again, results can be monitored using the `watch` command by opening a 2nd connection to the same FRR instance

```bash
watch -n0 -diff "vtysh -c 'show bgp summary'"
```

Outputs:

```bash
root@b34035faae0d:/# watch -n0 -diff "vtysh -c 'show bgp summary'"

Every 0.1s: vtysh -c 'show bgp summary'                                                                                                                                                                       b34035faae0d: Sat Aug  3 22:34:07 2024


IPv4 Unicast Summary:
BGP router identifier 192.168.100.100, local AS number 65100 VRF default vrf-id 0
BGP table version 112
RIB entries 3, using 288 bytes of memory
Peers 1, using 20 KiB of memory

Neighbor        V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
192.168.1.200   4      65200       317       225        0    0    0 00:00:03 Idle (Admin)        0 N/A

Total number of neighbors 1

...

Every 0.1s: vtysh -c 'show bgp summary'                                                                                                                                                                       b34035faae0d: Sat Aug  3 22:35:29 2024


IPv4 Unicast Summary:
BGP router identifier 192.168.100.100, local AS number 65100 VRF default vrf-id 0
BGP table version 117
RIB entries 5, using 480 bytes of memory
Peers 1, using 20 KiB of memory

Neighbor        V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
192.168.1.200   4      65200       329       239      117    0    0 00:00:09            1        1 N/A

Total number of neighbors 1
```

## Gathering logs in FRR

1. Create a file where to store the logs, in this case I will use `/var/log/logs.txt`. Give it write privileges to all users.

```bash
touch /var/log/logs.txt
chmod o+w /var/log/logs.txt
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
root@b34035faae0d:~# touch /var/log/logs.txt
root@b34035faae0d:~#

root@b34035faae0d:~# ls -l /var/log/logs.txt
-rw-r--r-- 1 root root 0 Aug  4 04:08 /var/log/logs.txt
root@b34035faae0d:~#

root@b34035faae0d:~# chmod o+w /var/log/logs.txt
root@b34035faae0d:~#

root@b34035faae0d:~# ls -l /var/log/logs.txt
-rw-r--rw- 1 root root 0 Aug  4 04:08 /var/log/logs.txt
root@b34035faae0d:~#
```
</details><br />

2. Configure FRR to redirect the logs to this file

```bash
root@b34035faae0d:~# vtysh

Hello, this is FRRouting (version 10.0.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

2024/08/04 04:11:52 [YDG3W-JND95] FD Limit set: 1048576 is stupidly large.  Is this what you intended?  Consider using --limit-fds also limiting size to 100000
b34035faae0d# configure terminal
b34035faae0d(config)# log file /var/log/logs.txt
b34035faae0d(config)# exit
b34035faae0d# exit
root@b34035faae0d:~#
```

3. Enable one or more debugs in FRR

```bash
vtysh
debug bgp keepalives
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
root@b34035faae0d:/# vtysh

Hello, this is FRRouting (version 10.0.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

2024/08/04 04:13:53 [YDG3W-JND95] FD Limit set: 1048576 is stupidly large.  Is this what you intended?  Consider using --limit-fds also limiting size to 100000
b34035faae0d#

b34035faae0d# debug bgp keepalives
BGP keepalives debugging is on
b34035faae0d#
```
</details><br />

4. Monitor the log files using, for instance `tail -f`

```bash
tail -f /var/log/logs.txt
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
root@b34035faae0d:~# tail -f /var/log/logs.txt
2024/08/04 04:14:59 BGP: [P8XN0-33WQ6] 192.168.1.200 [FSM] Timer (keepalive timer expire)
2024/08/04 04:14:59 BGP: [HRDT0-0DPQ7] 192.168.1.200 sending KEEPALIVE
2024/08/04 04:14:59 BGP: [X61A3-E95TJ] 192.168.1.200 KEEPALIVE rcvd
2024/08/04 04:15:59 BGP: [P8XN0-33WQ6] 192.168.1.200 [FSM] Timer (keepalive timer expire)
2024/08/04 04:15:59 BGP: [HRDT0-0DPQ7] 192.168.1.200 sending KEEPALIVE
2024/08/04 04:15:59 BGP: [X61A3-E95TJ] 192.168.1.200 KEEPALIVE rcvd
```
</details><br />

## References

- [Show OSPF Neighbors](https://docs.vmware.com/en/VMware-SD-WAN/6.0/vmware-sdwan-troubleshooting-guide/GUID-517EFC43-CF20-4F5A-BD50-4E608CD6F56C.html)
- [Logging in VYOS](https://forum.vyos.io/t/logging-in-vyos/2414)
- [How to clear the contents of a file from the command line?](https://superuser.com/questions/90008/how-to-clear-the-contents-of-a-file-from-the-command-line)
- [BGP not connecting, bgp_read_packet error #4438](https://github.com/FRRouting/frr/issues/4438)
- [FRRouting](https://docs.nvidia.com/networking-ethernet-software/cumulus-linux-59/Layer-3/FRRouting/)
- [Kill and Logout users in pts/* Linux](https://ivan.reallusiondesign.com/kill-and-logout-users-in-pts-linux/)
- [How do I write a 'for' loop in Bash?](https://stackoverflow.com/questions/49110/how-do-i-write-a-for-loop-in-bash)
- [Bash Scripting – If Statement](https://www.geeksforgeeks.org/bash-scripting-if-statement/)
- [Repeat command automatically in Linux](https://stackoverflow.com/questions/13593771/repeat-command-automatically-in-linux)
- [Modify File Permissions with chmod](https://www.linode.com/docs/guides/modify-file-permissions-with-chmod/)