---
title: Juniper Junos OS R&S commands
date: 2024-01-05 04:00:00 -0700
categories: [JUNIPER, JUNOS OS]
tags: [juniper]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/juniper_logo.png)

## Vlans

```
show vlans
```

```
edit vlans
set default vlan-id 1
set vlan10  vlan-id 10
set vlan20  vlan-id 20
set <VLAN_NAME> vlan-id <VLAN_ID>
```

## Layer 2 Access & Trunk ports

```
show ethernet-switching table
```

### Access Ports

```
delete interfaces xe-0/0/x unit 0 family inet
set    interfaces xe-0/0/x unit 0 family ethernet-switching
set    interfaces xe-0/0/x unit 0 family ethernet-switching interface-mode access vlan members default
```

or

```
edit itnerfaces xe-0/0/x.0
delete family inet

edit family ethernet-switching
set interface-mode access vlan members <VLAN_NAME>
```

### Trunk Ports

```
edit itnerfaces xe-0/0/x.0
delete family inet

edit family ethernet-switching
set interface-mode trunk vlan members [vlan10 vlan20]
```

