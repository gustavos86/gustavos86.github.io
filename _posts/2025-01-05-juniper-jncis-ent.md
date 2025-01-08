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
show spanning-tree interface
show spanning-tree bridge
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
set chassis aggregated-devices ethernet device-count 1

set interfaces ge-0/0/x ether-options 802.3ad ae0
set interfaces ge-0/0/y ether-options 802.3ad ae0

set interfaces ae0 unit 0 family ethernet-switching port-mode trunk

set interfaces ae0 aggregated-ether-options lacp active
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

Junos OS

- `[edit firewall family inet]`: IPv4 filters for Layer 3 interfaces
- `[edit firewall family ethernet-switching]`: Filters for Layer 2 interfaces

```
edit firewall family inet

edit filter sample-filter
set term block-bad-subnet from source-address 192.168.0.0/24 then discard
set term access-all then accept
```

Cisco IOS

```
access-list 100 deny ip 192.168.0.0 0.0.0.255 any
access-list 100 permit ip any any
```

## References

- [JUNOS RIB-GROUPS (1/2)](https://momcanfixanything.com/junos-rib-groups-1-2/)