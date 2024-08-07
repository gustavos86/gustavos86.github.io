---
title: Linux TTL and iptables
date: 2024-07-13 05:39:00 -0700
categories: [LINUX, IPTABLES]
tags: [linux, iptables, ttl]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

## Issue

There was an iptables configuration line added to a Linux based network device.
Once this iptables rule was added, remote access to the device was lost.

For simplicity, the two Linux based network devices directly connected to each other involved in this scenario are:

- hostname `alpha` with IP address 10.1.2.1
- hostname `bravo` with IP address 10.1.2.2

This is the iptables line, applied to `alpha`, which caused the issue:

```
user@alpha:~ $ sudo iptables -A INPUT --match ttl --ttl-gt 30 -j DROP
user@alpha:~ $ Connection to alpha closed by remote host.
Connection to alpha closed.
```

What this line is doing is blocking any packet which IP TTL is larger than value 30 from being processed by the device's CPU.

Pings (ICMP traffic) from `bravo`, which is the directly connected device, are not completing now.

```
user@bravo:~ $ ping 10.1.2.1
PING 10.1.2.1 (10.1.2.1) 56(84) bytes of data.
^C
--- 10.1.2.1 ping statistics ---
15 packets transmitted, 0 received, 100% packet loss, time 329ms

user@bravo:~ $
```

Attempts to SSH are also failing.

```
user@bravo:~ $ ssh 10.1.2.1
ssh: connect to host 10.1.2.1 port 22: Connection timed out
user@bravo:~ $
```

The next logical question is, what is the IP TTL that `bravo` is using when attempting to ping or SSH into `alpha`?

We can see the current IP TTL value by running `sysctl net.ipv4.ip_default_ttl`

```
user@bravo:~ $ sysctl net.ipv4.ip_default_ttl
net.ipv4.ip_default_ttl = 255
user@bravo:~ $
```

The IP TTL value set is **255**.

We can confirm ICMP packets are using this value with a **tcpdump** packet capture.

<details markdown=1>
<summary markdown="span">sudo tcpdump -i any icmp and host 10.1.2.1 -v</summary>

Notice `, ttl 255,`

```
user@bravo:~ $ sudo tcpdump -i any icmp and host 10.1.2.1 -v
tcpdump: listening on any, link-type LINUX_SLL (Linux cooked v1), capture size 262144 bytes
15:56:23.958804 IP (tos 0x0, ttl 255, id 38003, offset 0, flags [DF], proto ICMP (1), length 84)
    10.1.2.2 > 10.1.2.1: ICMP echo request, id 26065, seq 1, length 64
15:56:23.958815 IP (tos 0x0, ttl 255, id 38003, offset 0, flags [DF], proto ICMP (1), length 84)
    10.1.2.2 > 10.1.2.1: ICMP echo request, id 26065, seq 1, length 64
15:56:24.984817 IP (tos 0x0, ttl 255, id 38235, offset 0, flags [DF], proto ICMP (1), length 84)
    10.1.2.2 > 10.1.2.1: ICMP echo request, id 26065, seq 2, length 64
15:56:24.984837 IP (tos 0x0, ttl 255, id 38235, offset 0, flags [DF], proto ICMP (1), length 84)
    10.1.2.2 > 10.1.2.1: ICMP echo request, id 26065, seq 2, length 64
15:56:26.008887 IP (tos 0x0, ttl 255, id 39175, offset 0, flags [DF], proto ICMP (1), length 84)
    10.1.2.2 > 10.1.2.1: ICMP echo request, id 26065, seq 3, length 64
```
</details><br />

Similarly, a packet capture showing the SSH connection attempt displays the IP TTL with value 255.

<details markdown=1>
<summary markdown="span">sudo tcpdump -i any host 10.1.2.1 and dst port 22 -v</summary>

Notice `, ttl 255,`

```
user@bravo:~ $ sudo tcpdump -i any host 10.1.2.1 and dst port 22 -v
tcpdump: listening on any, link-type LINUX_SLL (Linux cooked v1), capture size 262144 bytes
16:00:59.477830 IP (tos 0x0, ttl 255, id 13018, offset 0, flags [DF], proto TCP (6), length 60)
    10.1.2.2.58182 > 10.1.2.1.ssh: Flags [S], cksum 0x0a2f (incorrect -> 0x2056), seq 1439507084, win 63420, options [mss 9060,sackOK,TS val 1207133330 ecr 0,nop,wscale 8], length 0
16:00:59.477846 IP (tos 0x0, ttl 255, id 13018, offset 0, flags [DF], proto TCP (6), length 60)
    10.1.2.2.58182 > 10.1.2.1.ssh: Flags [S], cksum 0x0a2f (incorrect -> 0x2056), seq 1439507084, win 63420, options [mss 9060,sackOK,TS val 1207133330 ecr 0,nop,wscale 8], length 0
16:01:00.504811 IP (tos 0x0, ttl 255, id 13019, offset 0, flags [DF], proto TCP (6), length 60)
    10.1.2.2.58182 > 10.1.2.1.ssh: Flags [S], cksum 0x0a2f (incorrect -> 0x1c53), seq 1439507084, win 63420, options [mss 9060,sackOK,TS val 1207134357 ecr 0,nop,wscale 8], length 0
```
</details>

## Solution

Temporarily, we modify the IP TTL on `bravo` so we can SSH into `alpha` and remove the iptables rule.

```
user@bravo:~ $ sudo sysctl -w net.ipv4.ip_default_ttl=29
net.ipv4.ip_default_ttl = 29
user@bravo:~ $
```

Now ping is working:

```
user@bravo:~ $ ping 10.1.2.1
PING 10.1.2.1 (10.1.2.1) 56(84) bytes of data.
64 bytes from 10.1.2.1: icmp_seq=1 ttl=255 time=0.657 ms
64 bytes from 10.1.2.1: icmp_seq=2 ttl=255 time=0.742 ms
64 bytes from 10.1.2.1: icmp_seq=3 ttl=255 time=0.580 ms
^C
--- 10.1.2.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 54ms
rtt min/avg/max/mdev = 0.580/0.659/0.742/0.072 ms
user@bravo:~ $
```

We see outgoing packets from `10.1.2.2` (`bravo`) with **IP TTL 29**.

<details markdown=1>
<summary markdown="span">sudo tcpdump -i any icmp and host 10.1.2.1 -v</summary>

Notice `, ttl 29,`

```
user@bravo:~ $ sudo tcpdump -i any icmp and host 10.1.2.1 -v
tcpdump: listening on any, link-type LINUX_SLL (Linux cooked v1), capture size 262144 bytes
16:06:23.892584 IP (tos 0x0, ttl 29, id 13601, offset 0, flags [DF], proto ICMP (1), length 84)
    10.1.2.2 > 10.1.2.1: ICMP echo request, id 28770, seq 1, length 64
16:06:23.892600 IP (tos 0x0, ttl 29, id 13601, offset 0, flags [DF], proto ICMP (1), length 84)
    10.1.2.2 > 10.1.2.1: ICMP echo request, id 28770, seq 1, length 64
16:06:23.893222 IP (tos 0x0, ttl 255, id 22280, offset 0, flags [none], proto ICMP (1), length 84)
    10.1.2.1 > 10.1.2.2: ICMP echo reply, id 28770, seq 1, length 64
16:06:23.893222 IP (tos 0x0, ttl 255, id 22280, offset 0, flags [none], proto ICMP (1), length 84)
    10.1.2.1 > 10.1.2.2: ICMP echo reply, id 28770, seq 1, length 64
16:06:24.920793 IP (tos 0x0, ttl 29, id 13737, offset 0, flags [DF], proto ICMP (1), length 84)
    10.1.2.2 > 10.1.2.1: ICMP echo request, id 28770, seq 2, length 64
16:06:24.920803 IP (tos 0x0, ttl 29, id 13737, offset 0, flags [DF], proto ICMP (1), length 84)
    10.1.2.2 > 10.1.2.1: ICMP echo request, id 28770, seq 2, length 64
16:06:24.921502 IP (tos 0x0, ttl 255, id 23261, offset 0, flags [none], proto ICMP (1), length 84)
    10.1.2.1 > 10.1.2.2: ICMP echo reply, id 28770, seq 2, length 64
16:06:24.921502 IP (tos 0x0, ttl 255, id 23261, offset 0, flags [none], proto ICMP (1), length 84)
    10.1.2.1 > 10.1.2.2: ICMP echo reply, id 28770, seq 2, length 64
```
</details><br />

SSH connectivity is recovered and we can remove the incorrectly set iptables rule

```
bravo:~$ ssh alpha
This is device alpha now...

alpha:~ $
alpha:~ $ sudo iptables -S
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-A INPUT -m ttl --ttl-gt 30 -j DROP
alpha:~ $
alpha:~ $ sudo iptables -D INPUT -m ttl --ttl-gt 30 -j DROP
alpha:~ $
alpha:~ $ sudo iptables -S
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
alpha:~ $
```

Once we are done, we set the IP TTL value back to its original configuration on `bravo`.

```
bravo:~ $ sudo sysctl -w net.ipv4.ip_default_ttl=255
net.ipv4.ip_default_ttl = 255
bravo:~ $
```

## References

- [https://gcore.com/learning/how-to-change-ttl-in-linux/](https://gcore.com/learning/how-to-change-ttl-in-linux/)
- [https://serverfault.com/questions/1109415/iptables-ttl-and-limit-bytes](https://serverfault.com/questions/1109415/iptables-ttl-and-limit-bytes)