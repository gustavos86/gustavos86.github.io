---
title: Juniper Junos OS R&S commands
date: 2025-01-05 04:00:00 -0700
categories: [JUNIPER, JUNOS OS]
tags: [juniper]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/juniper_logo.png)

## Vlans

```
show vlans
show vlans <VLAN_NAME> detail
```

```
edit vlans
set default vlan-id 1
set vlan10  vlan-id 10
set vlan20  vlan-id 20
set <VLAN_NAME> vlan-id <VLAN_ID>
```

Optional way to assign an interface to a VLAN

```
edit vlans <VLAN_NAME>

set interface <INTERFACE>
```

## Interface Monitoring commands

|          Juniper command           |         Cisco command        |
|------------------------------------|------------------------------|
| show interfaces descriptions       | show interfaces descriptions |
| show interfaces terse              | show ip interface brief      |
| show ethernet-switching interfaces | show interfaces switchport   |
| show ethernet-switching interfaces | show interfaces trunk        |
| show vlans                         | show vlans                   |
| show ethernet-switching table      | show mac address table       |

NOTE that on `show interfaces descriptions` - only interfaces with descriptions are displayed

```
show interfaces ge-0/0/x brief
show interfaces ge-0/0/x
show interfaces ge-0/0/x detail
show interfaces ge-0/0/x extensive
```

Log of recent changes in the MAC address table:

```
show ethernet-switching mac-learning-log | except 00:00:00:00:00:00
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
set native-vlan-id default      (optional)
```

## Spanning-Tree

802.1d STP
802.1w RSTP

```
show spanning-tree bridge
show spanning-tree interface
show ethernet-switching interfaces
```

Activate RSTP on all interfaces

```
edit protocols rstp

set interface all
```

Activate the `bpdu-block-on-edge` feature which Shuts down ports configured as `edge` when receiving a STP BPDU.

```
edit protocols rstp
set bpdu-block-on-edge

set interface xe-0/0/x edge
```

To automatically re-enable the port that was shutdown by the `bpdu-block-on-edge` feature and having received a STP BPDU.

```
edit protocols
edit layer2-control bpdu-block

set interface xe-0/0/x
set disable-timeout 180
```

To manually do the same

```
clear error bpdu interface xe-0/0/x
```

To use legacy STP as opposed to RSTP

```
edit protocols rstp

set force-version stp
```

To disable RSTP on a single interface

```
edit protocols rstp
edit interface xe-0/0/x

set disable
```

Modify RSTP Bridge-Priority

```
edit protocols rstp

set bridge-priority 20k
or
set bridge-priority 16k
or
set bridge-priority 8k
```

Modify RSTP cost

```
edit protocols rstp
edit interface xe-0/0/x

set cost <COST>
```

Example:

```
edit protocols rstp

set bridge-priority 20k
set interface ge-0/0/x.0 disable
set interface ge-0/0/y.0 cost 1000
set interface ge-0/0/z.0 edge

set interface ge-0/0/y.0 mode point-to-point
```

### MSTP

MSTP provides the same benefits as RSTP, but also provides standardized support to multiple VLANs with different topologies.

Group VLANs

```
set protocols mstp configuration-name example

set protocols mstp msti 1 vlan [1 3 5]  # Assign VLANs 1, 3, and 5 to instance 1
set protocols mstp msti 2 vlan [2 4 6]  # Assign VLANs 2, 4, and 6 to instance 2
```

## Rib-groups

1. Define `rib-group` under the `routing-options` hierarchy level

```
edit routing-options rib-groups <RIB-GROUP-NAME>

set import-rib <ROUTING-TABLE-NAME1>
set import-rib <ROUTING-TABLE-NAME2>
```

2.  Apply the `rib-group` to routing protocols, interface routes, or both, as needed

```
edit protocols ospf

set rib-group <RIB-GROUP-NAME>
```

```
edit routing-options

set interface-routes rib-group <FAMILY> <RIB-GROUP-NAME>
```

```
edit routing-options

set static rib-group <RIB-GROUP-NAME>
```

3. Create a `routing-policy` (optional)

```
edit policy-options policy-statement <POLICY-NAME> term <TERM_NAME>

set to rib <ROUTING-TABLE-NAME>
set from ...
set then ...
```

4. Apply the policy to the `rib-group`

The `import-policy` controls which routes are installed in each routing table.

```
edit routing-options rib-groups <RIB-GROUP-NAME>

set import-policy <POLICY-NAME>
```

## Mac Limiting

|    Juniper Term     |   Cisco term   |
|---------------------|----------------|
| Mac Limiting        |  Port Security |
| Persistent Learning |  Sticky MAC    |

```
show ethernet-switching interface xe-0/0/x.0
```

```
edit switching-options
edit interface xe-0/0/x.0

set interface-mac-limit <NUMBER>
set interface-mac-limit packet-action drop
set persistent-learning
```

## DHCP snooping, DAI (Dynamic ARP Inspection), IP Source Guard

```
show dhcp-security binding [ip-source-guard]
show dhcp-security arp inspection statistics
```

### EX Switch

```
edit ethernet-switching-options

edit secure-access-port interface xe-0/0/x
set dhcp-trusted

edit secure-access-port interface xe-0/0/y
set no-dhcp-trusted

edit vlan default
set examine-dhcp
set arp-inspection
set ip-source-guard
```

### QFX Switch

```
edit vlans default forwarding-options dhcp-security 
set arp-instapection
set ip-source-guard

set group TRUSTED overrides trusted
set group TRUSTED interface xe-0/0/x.0

set group STATIC-binding interface xe-0/0/x.0 static-ip <IP_ADDRESS> mac <MAC_ADDRESS>
```

### LACP - Aggregated Ethernet

```
show lacp interfaces
show interfaces terse | match ae
```

```
set chassis aggregated-devices ethernet device-count 1

set interfaces ae0 unit 0 family ethernet-switching port-mode trunk
set interfaces ae0 aggregated-ether-options lacp active

set interfaces ge-0/0/x ether-options 802.3ad ae0
set interfaces ge-0/0/y ether-options 802.3ad ae0
```

## Configuring Routed VLAN interfaces

```
edit vlans
set example l3-interface vlan.100

top
edit interfaces vlan
set unit 100 description Example
set unit 100 family inet address 192.168.100.2/24
```

## Firewall filters

Firewall filters are not stateful firewall rules, but stateless packet filters just like Cisco IOS ACLs.

```
show firewall
```

Junos OS

- `[edit firewall family inet]`: IPv4 filters for Layer 3 interfaces
- `[edit firewall family inet6]`: IPv6 filters for Layer 3 interfaces
- `[edit firewall family ethernet-switching]`: Filters for Layer 2 interfaces

```
edit firewall family inet

edit filter sample-filter
set term block-bad-subnet from source-address 192.168.0.0/24 then discard
set term access-all then accept
```

Applying the Firewall filter to an interface

```
set interfaces vlan.2 family inet filter input|output sample-filter
```

Applying the Firewall filter to an entire vlan

```
set vlans <VLAN_NAME> filter input|output sample-filter
```

Cisco IOS

```
access-list 100 deny ip 192.168.0.0 0.0.0.255 any
access-list 100 permit ip any any
```

## Routing

```
show route hidden
show route <NETWORK>
show route <NETWORK> exact
show route <NETWORK> exact detail
show route <NETWORK> exact extensive
```

|     Junos OS      |      Cisco IOS       |
|-------------------|----------------------|
| show route        | show ip route        |
| show bgp summary  | show ip bgp summary  |
| show bgp neighbor | show ip bgp neighbor |
| show ospf ...     | show ip ospf         |

|                      Junos OS                      |                         Cisco IOS                         |
|----------------------------------------------------|-----------------------------------------------------------|
| Route Preference                                   |  Administrative Distance                                  |
| Same Route Preference for IBGP and EBGP by default |  IBGP has higher Administrative Distance than EBGP routes |


Route Preference Values

|          Source          | Default Preference |
|--------------------------|--------------------|
| Direct                   |                  0 |
| Local                    |                  0 |
| Static                   |                  5 |
| OSPF internal            |                 10 |
| RIP                      |                100 |
| Aggregate                |                130 |
| OSPF AS external         |                150 |
| BGP (both EBGP and IBGP) |                170 |

### Configure Static Routes

```
edit routing-options

set static route 10.11.0.0/24 next-hop 192.168.3.1
set static route default next-hop 192.168.1.1
```

Multiple next-hops

```
edit routing-options
edit static route 10.12.0.0/24

set qualified-next-hop 192.168.2.15 interface vlan.2
set qualified-next-hop 192.168.2.15 preference 15

set qualified-next-hop 192.168.3.15 interface vlan.3
set qualified-next-hop 192.168.3.15 preference 30
```

## OSPF

```
show ospf database
show ospf interface
show ospf neighbor
show route protocols ospf
```

```
edit protocols ospf

set area 2 interface vlan.5
set area 2 interface ge-0/0/4.0
set area 2 interface ge-0/0/4.0 passive
set area 2 interface vlan.5 metric 200

set area 2 stub
set area 3 nssa

set area 2 stub default-metric 1
set area 3 deafult-lsa default-metric 1
```

NOTE: Loopback interfaces are set to `passive` implicitly in OSPF by Junos OS

### Redistribute from Static Routes to OSPF

```
edit policy-options
edit policy-statement static-to-ospf
edit term match-internal-static

set from protocols static
set from route-filter 192.168.0.0/16 orlonger;
set then metric 100
set then external type 2
set then accept

edit protocols ospf
set export static-to-ospf
```

### OSPF authentication

```
edit protocols ospf area 0.0.0.2
set interface vlan5 authentication md5 1 key <SUPER_SECRET_KEY>
```

## Storm Control

```
show interfaces xe-0/0/x extensive
```

```
edit forwarding-options
set storm-control-profiles default all
set storm-control-profiles drop-at-1G all bandwidth-level 1000000

top
delete interfaces xe-0/0/x.0 family inet
set interfaces xe-0/0/x.0 family ethernet-switching storm-control drop-at-1G
```

## Redundant Trunk Group

```
edit switch-options redundant-trunk-group
set group rtg1 interface xe-0/0/x.0 primary
set group rtg1 interface xe-0/0/y.0

set group rtg1 preempt-cutover-timer 30  # this is in seconds
```

## Graceful Routing Engine Switchover (GRES)

Minimize downtime during Routing Engine Transitions.
GRES often works in conjunction with NSR (Non-Stop Routing) to maintain uninterrupted control plane operation during a switchover event.

```
set virtual-chassis member 0 mastership-priority
set virtual-chassis member 1 mastership-priority

set chassis redundancy graceful-switchover
```

## References

- [Complete JNCIS-ENT (YouTube playlist)](https://www.youtube.com/playlist?list=PLsPPnwREYxwvQMlVtfpKU34uTwShws-3b)
- [JUNOS RIB-GROUPS (1/2)](https://momcanfixanything.com/junos-rib-groups-1-2/)
