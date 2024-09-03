---
title: Juniper playing with Junos PyEZ
date: 2024-08-11 17:00:00 -0700
categories: [JUNIPER, Junos PyEZ]
tags: [juniper, Junos PyEZ]     # TAG names should always be lowercase
---

## Introduction

I reviewed a Python script which automates locating and end point by IP Address by following the path from a Layer 3 Network device (a Core Switch) all the way down Distribution and Access switches. I noticed this script makes use of [Junos PyEZ](https://www.juniper.net/documentation/us/en/software/junos-pyez/junos-pyez-developer/topics/concept/junos-pyez-overview.html) which is basically a library to execute remote procedure calls (RPC) available through the Junos XML API.

I then wanted to try it myself, but at home I don't have any Juniper device to test with, so I used GNS3 to virtualize a Juniper vMX device.

After some difficulties, here the vMX virtual Router. The architechture is divided into the vCP VM (the Control Plane) and the vFP (the Forwarding Plane)

Here an screenshot of how I have the vFP connected to the vCP in GNS3. They can ping each other just fine over `em1` interface

![]({{ site.baseurl }}/images/2024/08-11-Juniper-playing-with-Junos-PyEZ/01-vFP-to-vCP-in-GNS3.png)

## Verify vCP to vFP connectivity

The vCP and vFP communicated over their `em1` interface on the **128.0.0.0/2** network

```bash
root> ping 128.0.0.16 routing-instance __juniper_private1__
PING 128.0.0.16 (128.0.0.16): 56 data bytes
64 bytes from 128.0.0.16: icmp_seq=0 ttl=64 time=0.273 ms
64 bytes from 128.0.0.16: icmp_seq=1 ttl=64 time=0.606 ms
```

```bash
root@vCP> show route table __juniper_private1__

__juniper_private1__.inet.0: 5 destinations, 6 routes (5 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.0.0/8         *[Direct/0] 00:03:45
                    > via em1.0
10.0.0.4/32        *[Local/0] 00:03:45
                      Local via em1.0
128.0.0.0/2        *[Direct/0] 00:03:45
                    > via em1.0
                    [Direct/0] 00:03:45
                    > via em1.0
128.0.0.1/32       *[Local/0] 00:03:45
                      Local via em1.0
128.0.0.4/32       *[Local/0] 00:03:45
                      Local via em1.0

__juniper_private1__.inet6.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

fe80::ea7:f1ff:fe05:1/128
                   *[Local/0] 00:03:45
                      Local via em1.0
fec0::/64          *[Direct/0] 00:03:45
                    > via em1.0
fec0::a:0:0:4/128  *[Local/0] 00:03:45
                      Local via em1.0

root@vCP> 
```

## vFP interfaces not showing up in vCP

I am facing an issue where the vFP is not properly booting the `ge-0/0/x` ports. I don't see them in the `vCP`

```bash
root@vCP> show interfaces terse
Interface               Admin Link Proto    Local                 Remote
cbp0                    up    up
demux0                  up    up
dsc                     up    up
em1                     up    up
em1.0                   up    up   inet     10.0.0.4/8
                                            128.0.0.1/2
                                            128.0.0.4/2
                                   inet6    fe80::ea7:f1ff:fe05:1/64
                                            fec0::a:0:0:4/64
                                   tnp      0x4
esi                     up    up
fti0                    up    up
fti1                    up    up
fti2                    up    up
fti3                    up    up
fti4                    up    up
fti5                    up    up
fti6                    up    up
fti7                    up    up
fxp0                    up    up
fxp0.0                  up    up   inet     192.168.127.132/24
gre                     up    up
ipip                    up    up
irb                     up    up
jsrv                    up    up
jsrv.1                  up    up   inet     128.0.0.127/2
lo0                     up    up
lo0.16384               up    up   inet     127.0.0.1           --> 0/0
lo0.16385               up    up   inet
lsi                     up    up
mtun                    up    up
pimd                    up    up
pime                    up    up
pip0                    up    up
pp0                     up    up
rbeb                    up    up
tap                     up    up
vtep                    up    up

root@vCP>
```

```bash
root@vCP> show chassis fpc 0
                     Temp  CPU Utilization (%)   CPU Utilization (%)  Memory    Utilization (%)
Slot State            (C)  Total  Interrupt      1min   5min   15min  DRAM (MB) Heap     Buffer
  0  Present          Absent

root@vCP>
```

Still trying to figure out this one, I have already tried:

- Increasing the RAM memory assigned the vFP VM
- Increasing the RAM memory assigned the GNS3 VM (running this in nested virtualization)
- Added `set chassis fpc 0 lite-modeset chassis fpc 0 lite-mode` to the vCP

## Testing with Junos PyEZ

First, we need to install it with Python PIP

```bash
python3 -m pip junos-eznc
```


## References

- [GitHub - junos_showCountersCascade](https://github.com/marco-minervino/junos_showCountersCascade)
- [Python/PyEZ](https://alexwilkins.dev/index.php/python-pyez/)
- [Junos CLI Reference - backup-liveness-detection](https://www.juniper.net/documentation/us/en/software/junos/cli-reference/topics/ref/statement/backup-liveness-detection-edit-protocols-iccp-peer-qfx-series.html)
- [Getting Started with Junos OS - Enable Remote Access Services](https://www.juniper.net/documentation/us/en/software/junos/junos-getting-started/topics/task/remote-access.html)
- [User Access and Authentication Administration Guide for Junos OS - User Accounts](https://www.juniper.net/documentation/us/en/software/junos/user-access/topics/topic-map/junos-os-user-accounts.html)
- [User Access and Authentication Administration Guide for Junos OS - Configure SSH Service for Remote Access to the Router or Switch](https://www.juniper.net/documentation/us/en/software/junos/user-access/topics/topic-map/junos-software-remote-access-overview.html#id-configuring-ssh-service-for-remote-access-to-the-router-or-switch)
- [vMX VPF crashed during the boot up (Using 15.1F4.15 on Ubuntu 14.04)](https://community.juniper.net/discussion/vmx-vpf-crashed-during-the-boot-up-using-151f415-on-ubuntu-1404)
- [Troubleshooting VFP and VCP Connection Establishment](https://www.juniper.net/documentation/us/en/software/vmx/vmx-vmware/topics/task/vmx-vm-connection-troubleshooting-esxi.html)
- [Juniper vMX on VMWare ESXi](https://edkimura.blogspot.com/2018/01/juniper-vmx-on-vmware-esxi.html)
- [vFPC fails to launch - console still in Linux hypervisor](https://community.juniper.net/discussion/vfpc-fails-to-launch-console-still-in-linux-hypervisor)