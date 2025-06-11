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

## Address Objects

- Zone Address Objects

Address objects that are tied to a specific zone
May only be used in security policies where the zone is referred

- Global Address Objects

Define objects in a global address book to avoid duplicate entries for multiple zones
Can be used by all security policies
All objects must be unique

### Creating Address Objects with the CLI

IP address

```
set security address-book PRIVATE address HOST1 192.168.1.35
```

Wildcard address

```
set security address-book PRIVATE address HOST1 wildcard-address 192.168.0.12/255.255.0.255
```

Domain name address

```
set security address-book PRIVATE address HOST1 dns-name www.host1.com
```

Range address

```
set security address-book PRIVATE address HOST1 range-address 192.168.1.100 to 192.168.1.150
```

To configure Global address books

```
set security address-book global
```

```
set security address-book TRUST_ADDRESSES description "Address objects for the trust zone" address HR-PRINTER-05 description "PRINTER IN HR BUILDING C ROOM 05" 192.168.50.45
```

Create and Address Set

```
set security address-book TRUST_ADDRESSES address-set HR-PRINTERS address HR-PRINTER-1
```

Global Address Book attached to a specific Zone

```
set security address-book TRUST_ADDRESSES attach zone TRUST
```

## Service Objects

Display pre-defined Service Security Objects

```
show configuration groups junos-defaults applications
```

Create Custom Applications

```
set applications application MyFTP description "FTP with smaller timer" application-protocol ftp protocol tcp desination-port 21 inactivity-timeout 300
```

Application Sets

```
set applications application-set access application junos-ping
set applications application-set access application junos-ssh
set applications application-set access application junos-https
```

## Configuring Security Zones (LAB)

**NOTE:** The `host-inbound-traffic interface` settings overrides the `host-inbound-traffic zone`.

```
set security zones security-zone UNTRUST interfaces ge-0/0/0.0

delete security zones security-zone TRUST host-inbound-traffic system-services all
set security zones security-zone TRUST host-inbound-traffic system-services ssh
set security zones security-zone TRUST host-inbound-traffic system-services https
set security zones security-zone TRUST interfaces ge-0/0/1.0
set security zones security-zone TRUST interfaces ge-0/0/1.0 host-inbound-traffic system-services telnet

set security zones security-zone DMZ interfaces ge-0/0/2.0
```

## Configuring Screens (LAB)

```
set security screen ids-option DMZ-SCREEN icmp large

set security zones security-zone DMZ screen DMZ-SCREEN
set security zones security-zone DMZ host-inbound-traffic system-services ping
```

```
show security screen statistics zone DMZ
```

## Configuring Global Addresses And Address Sets (LAB)

```
set security address-book global address INTERNET-HOST 172.31.15.1/32

set security address-book DMZ-BOOK address DMZ-NET 10.10.102.0/24
set security address-book DMZ-BOOK address WebServer01 10.10.102.11/32
set security address-book DMZ-BOOK address WebServer02 10.10.102.12/32
set security address-book DMZ-BOOK address WebServer03 10.10.102.13/32
set security address-book DMZ-BOOK attach zone DMZ

set security address-book DMZ-BOOK address-set WebServerSet address WebServer01
set security address-book DMZ-BOOK address-set WebServerSet address WebServer02
set security address-book DMZ-BOOK address-set WebServerSet address WebServer03
```

## Configuring Service Applications and Applications Sets (LAB)

```
set applications application MY-APP protocol tcp destination-port 2020 activity-timeout 300

set applications application-set WebServerAppSet description "Applications for the Web Server"
set applications application-set WebServerAppSet application MY-APP
set applications application-set WebServerAppSet application junos-http
set applications application-set WebServerAppSet application junos-https
```

```
run show configuration groups junos-defaults applications
```

## Security Policies

Security Policies are examined in the following order:

1. Zone policies
2. Global policies
3. Default policy

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/security-policies-and-transit-traffic.png)

Zone security policy example:

```
set security policies from-zone UNTRUST to-zone TRUST policy POLICY-2 match source-address any destination-address any application any
set security policies from-zone UNTRUST to-zone TRUST policy POLICY-2 then deny

set security policies from-zone UNTRUST to-zone TRUST policy RULE-2 match source-address any desination-addresa any application any
set security policies from-zone UNTRUST to-zone TRUST policy RULE-2 then deny
```

Global security policy example:

```
set security policies global policy GLOBAL-1 match source-address any destination-address any application any
set security policies global policy GLOBAL-1 then deny
```

Default security policy example:

Default is deny all traffic.

```
set default-policy permit-all
```

Troubleshoot

```
show security policies
```