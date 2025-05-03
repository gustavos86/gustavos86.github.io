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

## Layer 2 Tunnel Traffic

Q-in-Q tunneling is defined in IEEE 802.1ad.
Stacked VLAN tags:
- Service VLAN (S-VLAN): controlled by the service provider
- Customer VLAN (C-VLAN): typically controlled by the customer

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/l2tunneling-01.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/l2tunneling-02.png)

### Configuration using Bundling Method

C-VLAN interface (towards end-customer)

```
set interfaces ge-0/0/7 flexible-vlan-tagging
set interfaces ge-0/0/7 native-vlan-id 10
set interfaces ge-0/0/7 encapsulation extended-vlan-bridge
set interfaces ge-0/0/7 unit 100 vlan-id-list 1-25
set interfaces ge-0/0/7 unit 100 input-vlan-map push
set interfaces ge-0/0/7 unit 100 output-vlan-map pop
```

S-VLAN interface (towards the provider)

```
set interfaces ge-0/0/1 flexible-vlan-tagging
set interfaces ge-0/0/1 encapsulation extended-vlan-bridge
set interfaces ge-0/0/1 unit 100 vlan-id 100
```

### Configuration using Many-to-Many method

C-VLAN interface (towards end-customer)

```
set interfaces ge-0/0/7 flexible-vlan-tagging
set interfaces ge-0/0/7 native-vlan-id 10
set interfaces ge-0/0/7 encapsulation extended-vlan-bridge
set interfaces ge-0/0/7 unit 100 vlan-id-list 1-25
set interfaces ge-0/0/7 unit 100 input-vlan-map push
set interfaces ge-0/0/7 unit 100 output-vlan-map pop
set interfaces ge-0/0/7 unit 200 vlan-id-list 1-25
set interfaces ge-0/0/7 unit 200 input-vlan-map push
set interfaces ge-0/0/7 unit 200 output-vlan-map pop
```

S-VLAN interface (towards the provider)

```
set interfaces ge-0/0/1 flexible-vlan-tagging
set interfaces ge-0/0/1 encapsulation extended-vlan-bridge
set interfaces ge-0/0/1 unit 100 vlan-id 100
set interfaces ge-0/0/1 unit 200 vlan-id 200
```

### Configuring specific C-VLAN to S-VLAN Mapping

C-VLAN interface (towards end-customer)

```
set interfaces ge-0/0/15 flexible-vlan-tagging
set interfaces ge-0/0/15 encapsulation extended-vlan-bridge
set interfaces ge-0/0/15 unit 300 vlan-id 125
set interfaces ge-0/0/15 unit 300 input-vlan-map swap
set interfaces ge-0/0/15 unit 300 output-vlan-map swap
```

Monitoring commands

```
show interfaces ge-0/0/7 extensive
show interfaces ge-0/0/7 detail
```

## L2PT

```
set layer2-control mac-rewrite interface ge-0/0/7 protocol ?
```

Considerations:

- If you enable L2PT for untagged OAM LFM packets, do not configure LFM on the corresponding access interface
- If you enable L2PT for untagged LACP packets, do not configure LACP on the corresponding access interface
- CPD, UDLD, and VTP cannot be configured on EX Series switches. L2PT does, however, tunnel CDP, UDLD, and VTP PDUs.

## Configuring an LACP interface

1. Global command, allows X number of aggregated Ethernet interfaces to be created

```
set chassis aggregated-devices ethernet device-count 1
```

2. Assign physical interface to the LAG interface

```
set interfaces ge-0/0/1 ether-options 802.3ad ae0
set interfaces ge-0/0/2 ether-options 802.3ad ae0
```

3. Configure the LAG interface

```
set interfaces ae0.0 family ethernet-switching interface-mode trunk
set interfaces ae0.0 family ethernet-switching vlan members [v11 v12]
```

## Configuring a P-VLAN

```
set vlans pvlan-50 description "Primary VLAN" vlan-id 50
set vlans pvlan-50 community-vlans [finance sales]

set vlans finance description "Community VLAN" vlan-id 41
set vlans finance private-vlan community

set vlans sales description "Community VLAN" vlan-id 42
set vlans sales private-vlan community
```

```
set interfaces ge-0/0/1 family ethernet-switching interface-mode access
set interfaces ge-0/0/1 family ethernet-switching vlan members finance
set interfaces ge-0/0/2 family ethernet-switching interface-mode access
set interfaces ge-0/0/2 family ethernet-switching vlan members finance

set interfaces ge-0/0/3 family ethernet-switching interface-mode access
set interfaces ge-0/0/3 family ethernet-switching vlan members sales
set interfaces ge-0/0/4 family ethernet-switching interface-mode access
set interfaces ge-0/0/4 family ethernet-switching vlan members sales

set interfaces ae0.0 family ethernet-switching interface-mode trunk
set interfaces ae0.0 family ethernet-switching inter-switch-link
set interfaces ae0.0 family ethernet-switching vlan members pvlan-50
```

## Configuring MVRP

MVRP requires to use Spanning-Tree enabled on the Trunk interfaces that are configured for MVRP.
It requires Rapid Spanning Tree Protocol (RSTP) or Multiple Spanning Tree Protocol (MSTP) enabled on the interface

```
set protocols rstp interface ae0
set protocols mvrp interface ae0
```

Verification commands

```
show vlans
show mvrp dynamic-vlan-memberships
show mvrp statistics
```

## Configuring Q-in-Q tunneling

C-VLAN to use is 100
P-VLAN to use is 200

Port facing the end-customer

```
set interfaces ge-0/0/6 family ethernet-switching
set interfaces ge-0/0/6 flexible-vlan-tagging
set interfaces ge-0/0/6 encapsulation extended-vlan-bridge
set interfaces ge-0/0/6 unit 200 vlan-id-list 100
set interfaces ge-0/0/6 unit 200 input-vlan-map push
set interfaces ge-0/0/6 unit 200 output-vlan-map pop
```

Port facing the provider's network

```
set interfaces ae0 family ethernet-switching
set interfaces ae0 flexible-vlan-tagging
set interfaces ae0 encapsulation extended-vlan-bridge
set interfaces ae0 unit 200 vlan-id 200
set interfaces ae0 unit 11 vlan-id 11
set interfaces ae0 unit 12 vlan-id 12
```

```
set vlans v100 interface ge-0/0/6.200
set vlans v100 interface ae0.200

set vlans v11 interface ae0.11
set vlans v12 interface ae0.12
```

## MSTP - Multiple Spanning Tree Protocol

Switch with the least Bridge ID (Bridge Priority + MAC address) becomes the **root bridge**. Default Bridge Priority 32K by default.

The Root Bridge has all its port as Designated Ports (Forwarding State).
STP cost on 1Gbps interfaces `ge-0/0/x` interfaces is **20,000** by default.
Pord ID = Sender port priority + Sender interface number
Port Priority is **128** by default.

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/mstp-01.png)

MSTP provides extension to RSTP.
Enables you to create Multiple Spanning-Tree Instances (MSTIs) to balance traffic flows over all available links.

MSTP Regions are "clusters". MSTP Regions shares same Region name, revision level and VLAN-to-instance mapping.
You can configure a maximum of 64 MSTIs per MST region with one regional root bridge per instance.

Up to 64 MSTIs can be configured on each MST region.

### Configuration of MSTP

MSTI 0 is the **Common STP** instance

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/mstp-02.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/mstp-03.png)

```
show spanning-tree mstp configuration
show spanning-tree interface
show spanning-tree bridge
```

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/mstp-04.png)

## VSTP - VLAN Spanning Tree Protocol

VLAN Spanning Tree Protocol (VSTP) maintains a separate spanning-tree instance for each VLAN, enabling load balancing of Layer 2 traffic.

Proprietary protocol that is compatible with similar protocols from other vendors including:
- Per-VLAN Spanning Tree Plus (PVST+)
- Rapid-PVST+ (RPVST+)

Supports up to 253 different spanning-tree topologies

Rapid Spanning Tree Protocol (RSTP) can be enabled in addition to VSTP to account for any VLANs above and beyond 253.

Each STI (Spanning Tree Instance) will send their own BPDUs effectively making that each VLAN will send their own BPDUs.

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/vstp-01.png)

### Configuration of VSTP

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/vstp-02.png)

```
show spanning-tree interface
show spanning-tree bridge
```

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/vstp-03.png)
