---
title: Juniper Advanced Junos Enterprise Switching (AJEX) notes
date: 2025-05-02 08:30:00 -0700
categories: [JUNIPER, JUNOS OS]
tags: [juniper]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/juniper_logo.png)

## Vlans

**By default all interfaces are Access Ports assigned to VLAN 1.**

Access Ports

- Connects to end-user devices
- Carry untagged traffic
- Associated with single vlan

```
set interfaces ge-0/0/8 unit 0 family ethernet-switching interface-mode access
set interfaces ge-0/0/8 unit 0 family ethernet-switching vlam members 10
```

```
set interfaces ge-0/0/9 unit 0 family ethernet-switching interface-mode access
set interfaces ge-0/0/9 unit 0 family ethernet-switching vlam members 20
```

Trunk Ports

- Connect to other switches
- Carry tagged traffic
- Associated with multiple vlans


```
set interfaces ge-0/0/12 unit 0 family ethernet-switching interface-mode trunk
set interfaces ge-0/0/12 unit 0 family ethernet-switching vlam members [ v10 v20 ]

or 

set interfaces ge-0/0/12 unit 0 family ethernet-switching vlam members all
```

CLI commands available

```
{master:0}[edit] user@Switch-1# set vlans ?
+ community-vlans List of VLAN id or name description Text description of VLANs
> forwarding-options Forwarding options configuration
> interface Interface name for this VLAN isolated-vlan VLAN id or name l3-interface L3 interface name for this vlans mcae-mac-flush Enable IRB MAC flush in a/s mode for this VLAN on MCAE link up mcae-mac-synchronize Enable IRB MAC synchronization in this VLAN
> multicast-snooping-options Multicast snooping option configuration no-irb-layer-2-copy Disable transmission of layer-2 copy of packets of IRB private-vlan Type of secondary vlan for private vlan service-id Service id required if VLAN is of type MC-AE, and vlan-id all or vlan-id none or vlan-tags is configured
> switch-options VLANs switch-options configuration vlan-id IEEE 802.1q VLAN identifier for VLAN (1..4094) 
+ vlan-id-list Create VLAN for each vlan-id specified in the vlan-id-list
> vxlan 
```

## P-VLAN

Private VLANs:
- Enables to split a broadcast domain into multiple isolated broadcast subdomains. Essentially a VLAN inside a VLAN.
- The Primary VLAN can have one or more Secondary VLANs nested inside.
- Can be configured on a single switch or to span multiple switches.
- The P-VLAN feature is not supported on all EX Series switches.
- Cannot enable both the voice VLAN and P-VLAN feature at the same time on the same interface

The primary VLAN is the central VLAN for a P-VLAN.
- The primary VLAN always includes an 802.1Q tag.
- Secondary VLANs can be **community** or **isolated** VLANs and are nested inside the **primary VLAN**.
  - **Community VLAN**: Transports frames among interfaces within the same community and forwards frames upstream to the primary VLAN.
  - **Isolated VLAN**: Receives packets only from the primary VLAN and forward frames to the primary VLAN. It can be used when a P-VLAN is configured on one switch or spans multiple switches in a P-VLAN domain.
  - **Inter-switch isolated VLAN**: Used to forward isolated VLAN traffic from one switch to another through `pvlan-trunk` ports. Used when the P-VLAN spans multiple switches.
  - **Promiscuous ports**: Trunk ports that are typically connected to a Router. These ports have Layer 2 connectivity to all other switch ports, including isolated ports.
- Secondary VLANs are  not required to include 802.1Q tag unless the P-VLAN spans multiple switches.

Summary:

- Promiscuous port. Upstream trunk port that connects to a Router, Firewall, etc. No secondary VLAN on this one. Can communicate to any Community and Isolated secondary VLAN.
- Community ports to form groups of users within the same VLAN. They communicate among themselves as long as they are on the same Secondary VLAN. Can communicate with Promiscuous port.
- Isolated port have Layer 2 connectivity only with Primiscuous ports and `pvlan-trunk` ports.
- PVLAN-Trunk port. Connects 2 Switches participating in the same P-VLAN domain. Communicates with all ports except isolated ports.

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/p-vlan_01.png)
![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/p-vlan_02.png)
![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/p-vlan_03.png)
![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/p-vlan_04.png)
![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/p-vlan_05.png)

Tshoot commands

```
show vlans
show vlans extensive <PRIMARY-VLAN-NAME>
show vlans extensive <SECONDARY-VLAN-NAME>
```

## Dynamic VLAN Registration

MVRP - Multiple VLAN Registration Protocol.
Dynamically manages VLAN registration on a LAN. Prunes VLAN information on Trunk ports when a Switch has no active access ports for a configured vlan.
It can also be used to dynamically create VLANs in Switching networks.
MVRP replaces GVRP.

MVRP can only be enabled on Trunk interfaces.
MVRP does not support all Spanning-Tree Protocols (STPs). Currently, MVRP does not support the VLAN Spanning Tree Protocol (VSTP).

**NOTE:** There is an MVRP "extra byte" incompatibility in old vs new versions of Junos OS. Check the documentation in case of doubt.

Configuration

1. Configure all VLANs in the Access Switch.
2. Trunk ports are configured as usual but do not allow any VLAN on it.
3. Enable MVRP and specify the trunk/uplink ports.

```
set protocols mvrp interface ge-0/0/x.0
set protocols mvrp interface ge-0/0/y.0
```

Monitoring commands:

```
show mvrp
show mvrp dynamic-vlan-memberships
show mvrp statistics
show vlans
```