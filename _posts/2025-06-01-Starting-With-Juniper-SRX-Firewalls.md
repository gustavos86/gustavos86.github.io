---
title: Starting with Juniper SRX Firewalls
date: 2025-06-01 15:00:00 -0700
categories: [JUNIPER, JUNOS OS, SRX]
tags: [juniper]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/juniper_logo.png)

## Packet Mode Processing

The SRX basically operates like a Router.

```
set security forwarding-options family mpls mode packet-based
```

- [[SRX] How to change forwarding mode for IPv4 from 'flow based' to 'packet based'](https://supportportal.juniper.net/s/article/SRX-How-to-change-forwarding-mode-for-IPv4-from-flow-based-to-packet-based?language=en_US)

## Logical Packet Flow

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/logical-packet-flow-diagram.png)

## Branch SRX Series Factory-Default configuration

- Interface `ge-0/0/0` is set for **untrust zone** and to get IP via DHCP
- Default IP address on `fxp0.0`: **192.168.1.1/24**
- Interface `irb.0` is the **trust zone** with address **192.168.2.1/24**
- All other ports are configured as Layer 2

## Security Zones

Interfaces can pass and accept traffic only if assigned to **non-null zone**.

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/zones-01.png)

Types of Zones

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/types-of-zones.png)

Creating a Zone

```
set security zones security-zone untrust interface ge-0/0/0.0
set security zones security-zone trust interface ge-0/0/1.0
set security zones security-zone dmz interface ge-0/0/2.0
set security zones security-zone Server interface ge-0/0/3.0
set security zones security-zone VPN interface st0.0

set security zones functional-zone management interface ge-0/0/3.0
```

Allow SSH and FTP inbound the Security Zone called "HR" and destined to the SRX

```
set security zones security-zone HR host-inbound-traffic system-services ssh
set security zones security-zone HR host-inbound-traffic system-services ftp
```

Show commands

```
show security zones <NAME>
```

Junos-Host Zone CLI Configuration

Allow specific hosts from **SRC_ZONE** to SSH to the SRX

```
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-management match source-address host1
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-management match destination-address any
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-management match application junos-ssh
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-management match dynamic-application none
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-management match url-category none
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-management then permit
```

Block all other SSH attempts from other hosts

```
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block match source-address any
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block match destination-address any
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block match application junos-ssh
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block match source-identity any
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block match dynamic-application none
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block match url-category none
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block then deny
```

## Screen Objects

Generate alarms without dropping packets.

```
set security screen ids-option TEST alarm-without-drop
```

Screen Objects are evaluated only on the Ingress Zone.

Screen Types

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/screen-types.png)

Screen Categories

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/screen-categories.png)

Configuring Screen Options

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/screen-config.png)
