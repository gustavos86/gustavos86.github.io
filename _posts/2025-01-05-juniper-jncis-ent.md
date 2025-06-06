---
title: Juniper Junos OS R&S commands
date: 2025-01-05 04:00:00 -0700
categories: [JUNIPER, JUNOS OS]
tags: [juniper]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/juniper_logo.png)

## Vlans

In the pre-ELS **(Enhanced Layer 2 Software)** version of Junos OS, the default VLAN did not have a **VLAN ID** associated with it and was untagged.

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
or
set interface-mode trunk vlan members all  # To allow all the configured VLANs in the Trunk

set native-vlan-id <VLAN_ID>      (optional)
```

## Voice VLAN

To assign a port to a "voice" vlan, additionally to assigning the port to a vlan, configure the following under `switch-options` hierarchy.

```
edit switch-options
edit voip interface (access-ports | ge-0/0/x.0)
set vlan (<VLAN_NAME> | <VLAN_ID>)
set forwarding-class assured-forwarding
```

## Interface Range

Define a range of interfaces that share common configuration parameters.

```
edit interfaces
edit interface-range <RANGE-NAME>
set member-range ge-0/0/x to ge-0/0/z
set unit 0 family ethernet-switching
```

or

```
edit interfaces
edit interface-range <RANGE-NAME>
set member ge-0/0/x
set member ge-0/0/y
set member ge-0/0/z
set unit 0 family ethernet-switching
```

## Spanning-Tree

802.1d STP
802.1w RSTP

```
show spanning-tree bridge
show spanning-tree interface
show spanning-tree statistics interface
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

## BPDU Protection, Root Protection, and Loop Protection

### BPDU Protection

Configured on Edge Ports to drop/block incoming BPDUs and blocks the interface.

To unblock the interface, run

```
clear error bpdu interface ge-0/0/x
```

or automatically unblock the port after certain time with:

```
edit protocols layer2-control
set bpdu-block disable-timeout <10..3600 SECONDS>
```

#### With STP enabled

```
show spanning-tree interface ge-0/0/x.0
```

```
edit protocols rstp
set interface ge-0/0/x.0 edge
set bpdu-block-on-edge
```

#### With STP disabled

```
show interfaces ge-0/0/x
```

```
edit protocols layer2-control
set bpdu-block interface ge-0/0/x.0
```

### Root Protection

Enable Root Protection on ports that should not receive superior BPDUs from the root bridge and should not be elected as the root port.
Commonly configured on Aggregation layer Switches torwards Access layer Switches.

```
show spanning-tree interface
```

```
edit protocols rstp
set interface ge-0/0/x.0 no-root-poort
```

### Loop Protection

The loop protection feature provides additional protection against Layer 2 loops by preventing non-designated ports from becoming designated ports.
Enable loop protection on all non-designated ports.

Ports that detect the loss of BPDUs transition to the loop inconsistent role, which maintains the blocking state.
Port automatically transitions back to previous or new role when it receives a BPDU.

It is recommended when enabling loop protection, enable it on all switch interfaces that have a chance of becoming root or designated ports. It is most effective when it is enable don all switches within the network.

```
show spanning-tree interface
show log messages | match "loop|protect"
```

```
edit protocols rstp
set interface ge-0/0/x.0 bpdu-timeout-action block
```

**NOTE:** An interface can be configured for either loop protection or root protection but not both.

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

4. Apply an `import-policy` to the `rib-group` (optional)

The `import-policy` controls which routes are installed in each routing table.

```
set routing-options rib-groups <RIB-GROUP-NAME> import-policy <POLICY-NAME>
```

Referece: [Junos CLI Reference - rib-groups](https://www.juniper.net/documentation/us/en/software/junos/cli-reference/topics/ref/statement/rib-groups-edit-routing-options.html)

```
[edit routing-options]

rib-groups {
    group-name {
        export-rib group-name;
        import-policy [ policy-names ];
        import-rib [ group-names ];
    }
}
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

### LAGs - LACP - Aggregated Ethernet

802.3as standard. Link aggregation combines multiple Ethernet interfaces into a single link layer interface, also known as a **Link Aggregation Group (LAG)** or bundle.

- Full duplex and link speed must match
- Up to 16 member links per LAG
- Member links do not need to be contiguous ports, nor must they be on the same switch when part of a Virtual Chassis

LACP exchanges are made between actors and partners:

- Actor: Local interface
- Partner: Remote interface

LACP exchanges protocol data units (PDUs) across all member links to ensure that each physical interface is configured and is functioning properly.

```
show lacp interfaces
show lacp statistics interfaces
show interfaces terse | match ae
show interfaces extensive ae0.0 | find "LACP Statistics:"
```

1. Create aggregated Ethernet interfaces

```
set chassis aggregated-devices ethernet device-count <NUMBER_OF_aeX_INTERFACES>
```

2. Configure the aggregated Ethernet interface and associate desire dmember links with the LAG

```
set interfaces ae0 unit 0 family ethernet-switching interface-mode trunk
set interfaces ae0 unit 0 family ethernet-switching vlan members all
set interfaces ae0 aggregated-ether-options lacp active

set interfaces ge-0/0/x ether-options 802.3ad ae0
set interfaces ge-0/0/y ether-options 802.3ad ae0
```

Optional.

Set LACP exchange speed rate

```
edit interfaces ae0 aggregated-ether-options lacp
set periodic (fast|slow)
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

Firewall filters are not stateful firewall rules, but stateless packet filters just like **Cisco IOS ACLs**.

**Port-based** and **VLAN based** filters use the `family ethernet-switching` option, while **router-based** filters use `family inet` or `family inet6` depending on traffic type.

- Input order: Rx Packet, Port Filter, VLAN Filter, Router Filter
- Output order: Router Filter, VLAN Filter, Port Filter, Tx Packet

NOTE: A **router-based filter** that is applied to an integrated routing and bridging (IRB) does not apply to switched packets in the same VLAN.

```
show firewall
```

Junos OS

When a Firewall Filter is applied, the software **discards** (drops silently) all traffic not explicitly enabled.

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

## MAC limiting

By default, interfaces have no defined limit on the number of MAC addresses they can learn.

- VLAN can be configured to limit the number of times, a MAC address can move to a new interface within a period of time.
- Interface can be configured to only learn a certain number of MAC addresses or to process traffic only from specific MAC addresses.

Use MAC limiting to protect the network by:

- Limiting the number of MAC addresses learned on a port
- Preventing MAC address spoofing by explicitly configured accepted MAC addresses for a port
- Monitoring MAC address movement between the ports in a VLAN

MAC move limitng is enabled on a per-VLAN basis. For example, if MAC address moves more than the configured limit within one second, the switch performs the configured action.

When a MAC address or MAC move limit is exceeded, the switch can perform one of the following actions.

|    Action    |                                  Description                                  |
|--------------|-------------------------------------------------------------------------------|
| none         | Does nothing                                                                  |
| drop         | Drops the packet and generates an alarm, an SNMP trap, or a system log entry. |
| log          | Does not drop the packet but generates a system log entry.                    |
| drop-and-log | Drops the packet and generates an alarm, an SNMP trap, or a system log entry. |
| shutdown     | Disables the port, blocks data traffic, and generates a system log entry.     |

Use `recovery-timeout #` under `family ethernet-switching` interface hierarchy to recover the port automatically. Otherwise, manually clearing the disable port is required to recover it running `clear ethernet-switching recovery-timeout`.

Configuration example for **Static Source MAC addresses**

```
show ethernet-switching interface ge-0/0/x
show log messages | match l2ald
clear ethernet-switching recovery-timeout interface ge-0/0/x
```

```
set interfaces ge-0/0/x.0 accept-source-mac mac-address XX:XX:XX:XX:XX:XX
set interfaces ge-0/0/x.0 accept-source-mac mac-address YY:YY:YY:YY:YY:YY
```

```
set interfaces ge-0/0/x.0 interface-mac-limit 2
set interfaces ge-0/0/x.0 interface-mac-limit packet-action log|drop|shutdown|drop-and-log
```

```
set vlans <VLAN_NAME> switch-options mac-move-limit 1 packet-action shutdown
```

## Persistent MAC Learning

Persistent MAC learning or "Sticky MAC" enables the retation of dynamically learned MAC addresses on a port even after the switch is reloaded or the port is bounced.

User the `clear ethernet-switching table persistent-learning` to clear the persistent MAC address entry from the interface.

If the original port is down when moving the device, then the new port learns the MAC address and the device can connect. But if the original port comes up the original entry is re-installed and the devices now in the new port looses connectivity.

```
show ethernet-switching table
```

```
set switch-options interface ge-0/0/x.0 persistent-learning
```

## MACsec

Terms:

- CKN - Connecitivty Association Key Name
- CAK - Connecitivty Association Key
- MKA - MACsec Key Agreement
- SAK - Secure Association Key

Configuring static Connectivity Association Key (CAK) security mode:

1. Create connectivity association:

```
set security macsec connecitivty-association <CA_NAME>
```

2. Configure the MACsec mode as static CAK:

```
set security macsec connecitivty-association <CA_NAME> security-mode static-cak
```

3. Configure the preshared key with the Connecitivty Associate Key Name (CKN) and CAK:

```
set security macsec connecitivty-association <CA_NAME> pre-shared-key ckn <HEXADECIMAL_NUMBER>
set security macsec connecitivty-association <CA_NAME> pre-shared-key cak <HEXADECIMAL_NUMBER>
```

4. Associate interfaces with the connecitvity association:

```
set security macsec interfaces <INTERFACE_NAME> connectivity-association <CA_NAME>
```

Example:

```
edit security macsec
set connectivity-association ca1
set connectivity-association ca1 security-mode static-cak
set connectivity-association ca1 pre-shared-key ckn <HEXADECIMAL_NUMBER>
set connectivity-association ca1 pre-shared-key cak <HEXADECIMAL_NUMBER>
set interfaces xe-0/1/0 connectivity-association ca1
```

```
show security macsec connections
show security mka statistics
```

## DHCP snooping

Attackers can exploit DHCP by setting up a rogue DHCP server, effectively launching a denial of service (DoS) attack.
DHCP snooping inspects all DHCP packets on **untrusted** ports.

- By default, Junos OS detects **access ports** as **untrusted** and **trunk ports** as **trusted**
- DHCP Servers should be behind **trusted** ports

DHCP snooping supports **DHCP option 82**, aka the **DHCP relay agent** information option.

EX Series switch implementation of option 82 contains three sub-options:

- circuit-id - Identifies the circuit (interface, VLAN or both) on the switch on which the request was received. Example: `ge-0/0/10:vlan1` or `ge-0/0/10`
- remote-id - Identifies the host. By default, it is the MAC address of the Switch but it could be the hostname of the Switch, the interface description, or a character string of your choice.
- vendor-id - Identifies the vendor of the host. If enabled but not specified the value `Juniper` is used.

The DHCP Server must be configured to accept **Option 82** if enabled on Network devices.

```
show dhcp-security binding

clear dhcp-security binding
clear dhcp-security binding vlan <VLAN_ID>
clear dhcp-security binding interface <INTERFACE_NAME>
clear dhcp-security binding ip-address <IP_ADDRESS>
```

```
# Enabling dhcp security features under forwarding-options automatically turns on DHCP snooping
set vlans <VLAN_NAME> forwarding-options dhcp-security

# Overrides default behavior and enables specified access interface to receive DHCP server traffic (DHCPOFFER, DHCPACK, DHCPNAK). Default setting to Trunk ports.
# DHCP Servers should be behind trusted ports
set vlans <VLAN_NAME> forwarding-options dhcp-security group trusted-1 overrides trusted
set vlans <VLAN_NAME> forwarding-options dhcp-security group trusted-1 interface ge-0/0/x.0

# Access ports are untrusted by default anyway...
set vlans <VLAN_NAME> forwarding-options dhcp-security group untrusted interface ge-0/0/y.0
set vlans <VLAN_NAME> forwarding-options dhcp-security group untrusted interface ge-0/0/z.0

# Optional. Add Static Entries for hosts with ARP disabled
set vlans <VLAN_NAME> forwarding-options dhcp-security group untrusted interface ge-0/0/z.0 static-ip X.X.X.X mac XX:XX:XX:XX:XX:XX
```

Keep the DHCP Snooping database persistent across reboots

```
edit system processes
set dhcp-service dhcp-snooping-file /var/tmp/dhcp-snooping-database
set dhcp-service dhcp-snooping-file write-interval 60
```

```
file show /var/tmp/dhcp-snooping-database
show dhcp-security binding statistics
```

## Persistent Dynamic ARP Inspection (DAI)

**DHCP snoooping must be enabled for DAI to work**

Dynamic ARP Inspection (DAI) examines ARP requests and responses on the LAN. Each ARP packet received on an **untrusted access port** is validated against the **DHCP snooping database**.
By validating each ARP packet received on untrusted access ports, DAI can prevent ARP spoofing.
If the DHCP snooping database does not contain an IP address-to-MAC entry for the information within the ARP packet, DAI drops the ARP packet, preventing the propagation of invalid host address information.
DAI also drops ARP packets when the IP address in the packet is invalid because DAI depends on the entries found within the DHCP snooping database.
ARP packets bypass DAI on **trusted** ports.

```
show arp
show dhcp-security binding
show dhcp-security arp inspection statistics
show log messages | match DAI
```

DAI is enabled per VLAN and not on individual ports

**DAI must set ports to Trusted on those ports which connects to Hosts configured with an Static IP address in order to accept ARP packets to pass**

```
# Enable DAI
set vlans <VLAN_NAME> forwarding-options dhcp-security arp-inspection

# DAI must set ports to Trusted on those ports which connects to Hosts configured with an Static IP address in order to accept ARP packets to pass
set vlans <VLAN_NAME> forwarding-options dhcp-security group group-1 overrides trusted
set vlans <VLAN_NAME> forwarding-options dhcp-security group group-1 interface ge-0/0/x.0
```

ARP packets are sent to the Routing Engine (RE). To prevent CPU overloading, Junos OS retes limit these ARP packets hitting the RE.

## IP Source Guard

IP Source Guard checks the source IP and MAC address in packets entering **untrusted ports** agains the **DHCP snooping database**. Packets failing this check are **discarded**.

IP Source Guard is enabled per VLAN and it check packets only on **untrusted access interfaces** and never on **Trunk interfaces** or **trusted access interfaces**.

IP Source Guard prevents IP spoofing attacks by:
- Inspecting IP packets on untrusted ports and validating them against the DHCP snooping database
- Check if the source MAC address of the IP packet matches a valid entry in the DHCP snooping database
- If no IP-MAC entry in the database corresponds to the information in the IP packet, IP source guard drops the IP packet

```
show dhcp-security binding
show dhcp-security binding ip-source-guard
```

```
set vlans <VLAN_NAME> forwarding-options dhcp-security ip-source-guard
```

802.1X user authentication features is applied in one of the three modes:
- Single supplicant: Works with IP source guard
- Single-source supplicant: Does not work with IP source guard
- Multiple supplicant: Does not work with IP source guard

**Define a Static IP-MAC under dhcp-security for those hosts with Static IP address configured.**

## Graceful Routing Engine Switchover (GRES)

Enables system control to switch from the **primary RE** to the **backup RE** with minimal interruption to network communications by synchronizing the kernel tables and **Packet Forwarding Engine (PFE)** tables. This feature requires **redundant REs** or **Virtual Chassis**

GRES enables a Switch with redundant REs (or participating in a Virtual Chassis) to continue forwarding packets, even if one RE fails by enabling control of switch from the primary RE to the backup RE with minimal interruption to network communications.

GRES preserves **Interface and Kernel information** and ensures that traffic forwarding is not interrupted during a primary role change. GRES does not, however, preserve **Control Plane** information which means the Routing Protocols Process (rpd) must restart.
In such case, the information learned through rpd must be relearned unless **Nonstop Active Routing (NSR)** is also configured.

Without GRES, a failure on the primary RE or a manual switchover to the backup RE, causes a **Packet Forwarding Engine (PFE)** restart and all interfaces are discovered by the new primary RE.

With GRES enabled, the PFE is NOT restarted and all interface and kernet information is preserved.

```
show chassis routing-engine
show system switchover  # Only on Backup RE
commit synchronize
```

- **Enable GRES**

```
set chassis redundancy graceful-switchover
```

Manual Switchover:

```
request chassis routing-engine master switch
request chassis routing-engine master ?
```

## Nonstop Acting Routing (NSR)

Provides high availability in a Switch with **redundant REs** or on a **Virtual Chassis** by enabling transparent switchover of the REs **without requiring a restart of supporting routing protocols** by synchronizing the **Routing Protocol Process (rpd)** and routing information.

NSR allows switchover of the REs (or in Virtual Chassis) without alerting peering devices. On top of synchronizing configuration, interface and Kernel information (GRES), NSR also synchronizes Routing Protocol information by running the **Routing Protocol Process (rpd)** on the backup RE.

```
show task replication
```

Alternatively, issue operational `show` commands such as `show ospf neighbor, show bgp summary, show route` on the backup RE to verify the state information was successfully replicated.

- **Enable NSR**

**NOTE:** NSR requires GRES to be configured.

```
set chassis redundancy graceful-switchover  # Enable GRES first
set routing-options nonstop-routing
commit synchronize
```

**NOTE:** `commit synchronize` can become the default behavior once we configure it with `set system commit synchronize`

## Nonstop Briding (NSB)

Provides high availability in a Switch with **redundant REs** or on a **Virtual Chassis** by enabling transparent switchover of the REs by enabling transparent switchover of the REs **without requiring a restart of supported L2 protocols** by synchronizing the RE process and switching information.

NSB allows switchover of the REs (or in Virtual Chassis) without alerting peering devices. NSB also saves supported Layer 2 (L2) information by running the **l2cpd** process on the backup RE.

**NSB does the same for L2 protocols that NSR does for L3 Routing Protocols.**

-- **Enable NSB**

**NOTE:** NSB requires GRES to be configured.

```
set chassis redundancy graceful-switchover  # Enable GRES first
set protocols layer2-control nonstop-bridging
commit synchronize
```

**NOTE:** `commit synchronize` can become the default behavior once we configure it with `set system commit synchronize`

If NSB is enabled, the backup RE will show some information in the output of `show spanning-tree bridge`, otherwise it would show that the `l2cpd-service` subsystem is NOT running.

## Unified ISSU - Unified In-Service Software Upgrade

Requires:

1. Have GRES and NSR enabled
2. Both RE running same Junos OS release
3. Have the new Juons OS release downloaded to both RE
4. Execute the command `request system software in-service-upgrade`

```
show chassis in-service-upgrade
```

## Virtual Chassis

Renumber the **member-id** of a Switch member of a Virtual Chassis

```
request virtual-chassis renumber member-id 0 new-member-id 5
```

Shutdown a specific member of a Virtual Chassis.

```
request system halt member <MEMBER-ID>
```

Access individual member of a Virtual Chassis.

```
request session number <MEMBER-ID>
```

Upgrade Switches part of a Virtual Chassis.

```
# All switches
request system software add

# Specific Swich
request system software add member <MEMBER-ID>
```

Auto Software Update for new members of the Virtual Chassis

```
set virtual-chassis auto-sw-update package-name /var/tmp/jinstall-ex-4300-21.3R1.9-signed.tgz
```

Enabling Virtual Chassis Ports

```
request virtual-chassis vc-port set pic-slot 1 port 0
request virtual-chassis vc-port set pic-slot 1 port 1
show virtual-chassis vc-port
```

Disabling and Deleting Virtual Chassis Ports

```
request virtual-chassis vc-port set interface vcp-255/1/0 disable
request virtual-chassis vc-port delete pic-slot 1 port 0
show virtual-chassis vc-port
```

Example of Virtual Chassis configuration

```
edit virtual-chassis
set preprovisioned
set member 0 role routing-engine
set member 0 serial-number XX0123456789
set member 1 role line-card
set member 1 serial-number XX0987654321
set member 2 role routing-engine
set member 2 serial-number XX1234123412
set member 3 role line-card
set member 3 serial-number XX9128387465
```

Commands to monitor Virtual Chassis

```
show configuration virtual-chassis
show virtual-chassis status
show virtual-chassis vc-port
```

Failover between Switches acting as Routing-Engines in Virtual-Chassis

```
request chassis routing-engine master switch
```

Another example of configuring Virtual-Chassis

EX1

```
edit virtual-chassis
set member 0 mastership-priority 255
set no-split-detection

request virtual-chassis vc-port set interface vcp-255/1/0
request virtual-chassis vc-port set interface vcp-255/1/1

request virtual-chassis vc-port set interface vcp-255/1/1 member 1
```

EX2

```
request virtual-chassis vc-port set interface vcp-255/1/0
```

To return to standalone mode:

EX1

```
request virtual-chassis vc-port delete pic-slot 1 port 0
request virtual-chassis vc-port delete pic-slot 1 port 0 member 1

request virtual-chassis vc-port delete pic-slot 1 port 1
request virtual-chassis vc-port delete pic-slot 1 port 1 member 1

request virtual-chassis recycle member-id 0
request virtual-chassis renumber member-id 1 new-member-id 0
request virtual-chasiss recycle member-id 1
```

## MSTP

```
show spanning-tree mstp configuration
show spanning-tree interface
show spanning-tree bridge
```

```
edit protocols mstp
set configuration-name <CONFIGURATION_NAME>  # Optional but must match on all Swithches part of the MSTP region. Default to blank.
set revision-level <REVISION_LEVEL>          # Optional but must match on all Swithches part of the MSTP region. Default to zero (0).

set interface <INTERFACE_NAME>  # Interfaces participating in MSTP

# VLANS not part of any MSTI are assigned to MSTI 0 (CST)
set msti <MSTI_ID> bridge-priority <PRIORITY>
set msti <MSTI_ID> vlan <VLAN_ID>|<VLAN_NAME>

set msti <MSTI_ID> bridge-priority <PRIORITY>
set msti <MSTI_ID> vlan <VLAN_ID>|<VLAN_NAME>
```

## Non-ELS vs ELS

### ethernet-switching-options vs switch-options

- Non-ELS use `edit ethernet-switching-options` hierarchy for Layer 2 feature configurations.
- ELS uses `edit switch-options` instead.

### RVI interfaces vs IRB interfaces

These are uses for Layer 3, similar to Cisco SVIs.

- Non-ELS uses RVIs (Routed Vlan Interfaces)

```
set interfaces vlan.11 family inet address 172.23.11.10/24
set interfaces vlan.12 family inet address 172.23.12.10/24
set vlans <VLAN11> vlan-id 11
set vlans <VLAN11> l3-interface vlan.11
set vlans <VLAN12> vlan-id 12
set vlans <VLAN12> l3-interface vlan.12
```

- ELS uses IRB (Internet Routing and Bridging)

```
set interfaces irb.11 family inet address 172.23.11.10/24
set interfaces irb.12 family inet address 172.23.12.10/24
set vlans <VLAN11> vlan-id 11
set vlans <VLAN11> l3-interface irb.11
set vlans <VLAN12> vlan-id 12
set vlans <VLAN12> l3-interface irb.12
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

## Static Routes

```
set routing-options static route 192.168.7.0/24 next-hop 192.168.2.1
```

Configure Static Route to Null0

- `reject` device will reply with an ICMP Network Unreachable back to the source
- `discard` will drop the packet silently

```
set routing-options static route 192.168.7.0/24 reject
set routing-options static route 192.168.8.0/24 discard

```

```
edit routing-options

set static route 10.11.0.0/24 next-hop 192.168.3.1
set static route default next-hop 192.168.1.1
```

Multiple next-hops - Qualified Next Hop

**Qualified Next Hop** in Juniper is the equivalent to **Floating Static Route** in Cisco.
It is about configuring a 2nd Static Route to the same destination with a less preferred Route Preference.

```
edit routing-options
edit static route 10.12.0.0/24

set qualified-next-hop 192.168.2.15 preference 15
set qualified-next-hop 192.168.3.15 preference 30
```

Recursive static route

Requires the parameter `resolve` for next-hop not in the Routing Table as `direct`

```
set routing-options static route 3.3.3.3/32 next-hop 192.168.1.32 resolve
```

No-readvertise option

Restricts route from being advertised through routing policy; highly suggested for static routes used for management traffic.

```
set routing-options static route 172.28.102.0/24 next-hop 10.210.11.190
set routing-options static route 172.28.102.0/24 no-readvertise
```

More Static Route options

|  Options   |                                                                                        Description                                                                                         |
|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| as-path    | Used if the route is intended to be redistributed into BGP and you want to add values manually to the AS path attribute.                                                                   |
| community  | Used if this route is intended for BGP, and you want to add community values to the route for your AS.                                                                                     |
| metric     | If multiple routes share the same preference value, then the route with the best metric becomes active in the routing table. Use this value to prefer one route over another in this case. |
| preference | Used to increase the value of the static routes to prefer other sources of routing information.                                                                                            |

## Aggregate Static Routes

```
show route protocol aggregate
show route 172.29.0.0/22 exact detail
```

```
edit routing-options
set aggregate route 172.29.0.0/22
set aggregate route 172.25.0.0/16 discard
```

## Generated Static Route

Similar to the **Aggregate Static Route**. The difference is that the **next-hop** is the one from the "primary contributing route".
The **primary contributing route** is the route with the lowest Preference that falls with the aggregated range of prefixes.
If there are multiple routes that fall within the aggregated range that share the same route preference, the route with the lowest number prefix (not lowest prefix range) is selected as the primary contributing route.

```
edit routing-options
set generate route 10.0.0.0/16
```

Example of **Conditional Route Advertisemente**

OSPF to advertise default route 0.0.0.0/0 only if we have received **10.0.0.0/16** via BGP (and is installed in the Routing Table)

```
edit policy-options
set policy-statement MATCH_CONTRIBUTING_PREFIX term MATCH_BGP_PREFIX from protocol bgp
set policy-statement MATCH_CONTRIBUTING_PREFIX term MATCH_BGP_PREFIX from route-filter 10.0.0.0/16 exact
set policy-statement MATCH_CONTRIBUTING_PREFIX term MATCH_BGP_PREFIX then accept
set policy-statement MATCH_CONTRIBUTING_PREFIX term ELSE-REJECT then reject

set policy-statement EXPORT_DEFAULT term MATCH_DEFAULT from protocol aggregate
set policy-statement EXPORT_DEFAULT term MATCH_DEFAULT from route-filter 0.0.0.0/0 exact
set policy-statement EXPORT_DEFAULT term MATCH_DEFAULT then accept

top
edit routing-options
set generate route 0.0.0.0/0 policy MATCH_CONTRIBUTING_PREFIX

top
edit protocols ospf
set export EXPORT_DEFAULT
```

## Martian addresses

Hosts or network addresses for which all routing information (also known as routing table) is ignored.
Martian addresses are never installed in the routign table (also known as routing information base [RIB])

Default Martian Addresses for IPv4 and IPv6

|         IPv4          |                      IPv6                      |
|-----------------------|------------------------------------------------|
| 0.0.0.0/8 orlonger    | Loopback address                               |
| 127.0.0.0/8 orlonger  | Reserved and unassigned prefixes from RFC 2373 |
| 192.0.0.0/24 orlonger | Link-local unicast prefix                      |
| 240.0.0.0/4 orlonger  |                                                |

Match types: `exact`, `longer`, `orlonger`, `prefix-length-range`, `through` and `upto`

```
show route martians
```

```
edit routing-options
set martians 23.0.0.0/8 orlonger
set martians 31.0.0.0/8 orlonger
set martians 36.0.0.0/8 orlonger

# To remove and IP address block from the Martian address list use the allow command
set martians 240.0.0.0/4 orlonger allow
```

## Routing instances

```
show route instance

show interfaces terse routing-instance <INSTANCE_NAME>
show route table <INSTANCE_NAME>

ping X.X.X.X routing-instance <INSTANCE_NAME>
```

Configure:

```
edit routing-instances <INSTANCE_NAME>
set instance-type virtual-router
set interface ge-0/0/x.0
set interface ge-0/0/y.0
set interface lo0.x
set routing-options static route 0.0.0.0/0 next-hop 172.26.25.1
set protocols ospf area 0.0.0.0 interface ge-0/0/x.0
set protocols ospf area 0.0.0.0 interface ge-0/0/y.0
set protocols ospf area 0.0.0.0 interface lo0.x
```

## OSPF

```
show ospf statistics
show ospf database
show ospf interface [extensive]
show ospf neighbor [extensive]
show ospf route
show ospf log
show route protocol ospf
```

```
set routing-options router-id <X.X.X.X>

set protocols ospf area <AREA-ID> <AREA-OPTIONS>
set protocols ospf area <AREA-ID> interface <INTERFACE-NAME> <INTERFACE-OPTIONS>

set protocols ospf3 area <AREA-ID> <AREA-OPTIONS>
set protocols ospf3 area <AREA-ID> interface <INTERFACE-NAME> <INTERFACE-OPTIONS>
```

Another example:

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

### OSPF interface type

```
edit protocols ospf area 0
set interface all interface-type p2p
```

### Set Router-ID

```
edit routing-options
set router-id 10.10.10.10
```

### Debug OSPF

```
set protocols ospf traceoptions file ospf-trace
set protocols ospf traceoptions flag error detail
set protocols ospf traceoptions flag event detail
show log ospf-trace
```

Use `show log <FILENAME>` or `monitor start` to see the "trace" information.

### Summarize in OSPF

```
edit protocols ospf
edit area <X>
set area-range 192.168.0.0/21
or
set nssa area-range 192.168.0.0/21 [restrict]
```

## Storm Control

Storm control monitors traffic levels and drops traffic when the threshold (storm control level) is exceeded.
Prevents traffic from proliferating and degrading the LAN.
The storm control feature ensures that traffic storms do not degrade LAN performance.

When the storm control level is exceeded, the switch can either:
- Drop offending traffic (default) or
- Shut down the interface through which the traffic is passing.

Using the default configuration, all **broadcast, multicast, and unknown unicast (BUM)** traffic that **exceed 80 percent** is **dropped**.

```
show interfaces xe-0/0/x extensive
show ethernet-switching interface xe-0/0/x
show log messages | match l2ald | match xe-0/0/x
```

When `action-shutdown` is configured, this commands manually recovers the port.

```
clear ethernet-switching recovery-timeout
```

Example 1:

```
edit forwarding-options
set storm-control-profiles drop-at-1G-profile all bandwidth-level 1000000

top
set interfaces xe-0/0/x.0 family ethernet-switching storm-control drop-at-1G-profile
```

Example 2:

```
edit forwarding-options
set storm-control-profiles my-profile all bandwidth-level 5000
set storm-control-profiles my-profile action-shutdown

top
set interfaces xe-0/0/x.0 family ethernet-switching storm-control my-profile
set interfaces xe-0/0/x.0 family ethernet-switching recovery-timeout 3600

```

## RTG - Redundant Trunk Group

RTGs are used as an alternative to STP on trunk ports in redundant enterprise networks.
RTG is typically only configured on access switches.
RTG and STP are mutually exclusive on a given port.


```
show redundant-trunk-group
```

```
edit switch-options redundant-trunk-group
set group <RTG_NAME> interface xe-0/0/x.0 primary
set group <RTG_NAME> interface xe-0/0/y.0
```

Optional.

```
set group <RTG_NAME> preempt-cutover-timer 30  # this is in seconds
```

## Graceful Routing Engine Switchover (GRES)

Minimize downtime during Routing Engine Transitions.
GRES often works in conjunction with NSR (Non-Stop Routing) to maintain uninterrupted control plane operation during a switchover event.

```
show system switchover
show chassis routing-engine
```

```
set virtual-chassis member 0 mastership-priority
set virtual-chassis member 1 mastership-priority

set chassis redundancy graceful-switchover
```

## IRB Bridging

IRB interfaces are used to do **inter-vlan** Routing. They are the equivalent to Cisco SVIs.

IRBs must be associated with a VLAN and must have an operational L2 interface participating in that VLAN before they become operational.

All EX-Series switches running ELS (Enhanced Layer 2 Software) support IRBs as well as other Layer 3 routing operations.

```
set vlans blue vlan-id 10 l3-interface irb.10
set vlan green vlan-id 20 l3-interface irb.20

set interfaces irb.10 family inet address 192.168.10.1/24
set interfaces irb.20 family inet address 192.168.20.1.24

delete interfaces xe-0/0/x.0 family inet
delete interfaces xe-0/0/y.0 family inet
set interfaces xe-0/0/x.0 family ethernet-switching vlan members blue
set interfaces xe-0/0/y.0 family ethernet-switching vlan members green
```

## Load Balancing

This is ECMP (Equal Cost Multi-Path)

- Per packet (not recommended)
- Per flow. This is the one we are configuring

**IMPORTANT:** Defining and applying a load-balancing policy affects only the **forwarding table (show route forwarding-table)**.
The **routing table (show route)** remains as it was before you defined and applied the load-balancing policy.

```
show route 1.1.1.1
show route forwarding-table | match 1.1.1.1   # Here is where we should the ECMP entry
```

```
edit policy-options policy-statement load-balance-loopback
set from route-filter 1.1.1.1/32 exact
set then load-balance per-packet   # this actually means "per flow"

top edit routing-options
set forwarding-table export load-balance-loopback
```

Another example, this time for all routes with ECMP entries in the Routig Table.

```
set policy-options policy-statement LOAD_BALANCE_ALL then load-balance per-packet
set routing-options forwarding-table export LOAD_BALANCE_ALL
```

By default, ECMP hash is based on Source IP, Destionation IP and L4 protocol.
To include Layer 4 ports information in the ECMP forwarding hash. Note that `layer-3` and `layer-4` keywords must both be used in the configuration.

```
set forwarding-options hash-key family inet layer-3
set forwarding-options hash-key family inet layer-4
```

This also is optional for MPLS (`set family mpls`) and VPLS (`set family multiservices`).
For IPv6, Junos OS already performs ECMP based on L3 & L4 information in the Forwarding Table by default.

## Filter-Based Forwarding

This is like **PBR (Policy-Based Routing)** in Cisco IOS.

1. Creating a match filter
Defined under the `[edit firewall]` hierarchy.
Matches traffic with respective routing instance.
The defined filter is applied as an `input` filter to ingress interface.

Example:

```
set firewall family inet filter <FILTER_NAME> term <TERM_NAME_A> from <MATCH_CONDITIONS>
set firewall family inet filter <FILTER_NAME> term <TERM_NAME_A> then routing-instance <INSTANCE_NAME_A>

set firewall family inet filter <FILTER_NAME> term <TERM_NAME_B> from <MATCH_CONDITIONS>
set firewall family inet filter <FILTER_NAME> term <TERM_NAME_B> then routing-instance <INSTANCE_NAME_B>

set firewall family inet filter <FILTER_NAME> term <TERM_NAME_C> then accept
```

2. Creating routing instances
Defined under the `[edit routing-instances]` hierarchy.
Creates unique routing tables to forward traffic towards destination along associated path.
The instance includes routing information (typically a default static route) that is stored in the corresponding route table.

```
set routing-instances <INSTANCE_NAME_A> instance-type forwarding
set routing-instances <INSTANCE_NAME_A> routing-options static route 0.0.0.0/0 next-hop <NEXT_HOP_IP>

set routing-instances <INSTANCE_NAME_B> instance-type forwarding
set routing-instances <INSTANCE_NAME_B> routing-options static route 0.0.0.0/0 next-hop <NEXT_HOP_IP>
```

3. Creating a RIB group
Defined under the `[edit routing-options]` hierarchy.
Shares interface routes between intances so that directly connected next hops can be resolved.

**NOTE:** You cannot associate any interface to a routing-instance of type `forwarding`.
Therefore, **RIB-Groups** is required to redistribute the **next-hop IP** of the static routes to the routing-instances.

```
set routing-options rib-groups <GROUP_NAME> import-rib [ inet.0 <INSTANCE_NAME_A>.inet.0 <INSTANCE_NAME_B>.inet.0 ]
set routing-options interface-routes rib-group inet <GROUP_NAME>
```

Alternatively to **RIB groups**, use the `instance-import` option.

```
# the master keyword is used to refer to master routing instance to import routes from inet.0 into ISP-A.inet.0
set policy-options policy-statement <ISP-A-import> from instance master then accept

set routing-instances <INSTANCE_NAME_B> instance-type forwarding
set routing-instances <INSTANCE_NAME_B> routing-options static route 0.0.0.0/0 next-hop <NEXT_HOP_IP>
set routing-instances <INSTANCE_NAME_B> routing-options instance-import <ISP-A-import>
```

Another example:

```
show firewall family inet filter customer-servers
show route-instances
show route table ISP-A.inet.0
show route table ISP-B.inet.0
```

Step 1

```
edit firewall family inet filter customer-servers
set term match-serverA-subnet from source-address 12.1.1.0/24
set term match-serverA-subnet then routing-instance ISP-A
set term match-serverB-subnet from source-address 12.2.2.0/24
set term match-serverB-subnet then routing-instance ISP-B

edit interfaces ge-0/0/x.0 family inet
set filter input customer-servers
```

Step 2

```
edit routing-instances
set ISP-A instace-type forwarding
set ISP-A routing-options static route 0/0 next-hop 10.1.0.2
set ISP-B instace-type forwarding
set ISP-B routing-options static route 0/0 next-hop 10.1.0.6
```

Step 3

```
edit routing-options
set rib-group FBF-rib-group import-rib [inet.0 ISP-A.inet.0 ISP-B.inet.0]
set interface-routes rib-group inet FBF-rib-group

set rib-group FBF-rib-group import-policy <POLICY_NAME>  # optional
```

## BGP

```
show bgp summary
show bgp neighbor <X.X.X.X>
show bgp group
show route protocol bgp [detail|extensive]
show route receive-protocol bgp <NEIGHBOR-ADDRESS>  # Before the effects of import-policy
show route advertising-protocol bgp <NEIGHBOR-ADDRESS>  # Before the effects of export-policy
```

```
edit routing-options
set autonomous-system <ASN>

top
edit protocols bgp
edit group <GROUP-NAME>
set type external
set neighbor <NEIGHBOR_IP>
set peer-as <PEER_ASN>
```

Another example:

```
set routing-options router-id <X.X.X.X>
set routing-options autonomous-system <ASN>

set protocols bgp group <GROUP_NAME> type external
set protocols bgp group <GROUP_NAME> peer-as <ASN>
set protocols bgp group <GROUP_NAME> neighbor <X.X.X.X>

set protocols bgp group <GROUP_NAME> type internal
set protocols bgp group <GROUP_NAME> local-address <X.X.X.X>
set protocols bgp group <GROUP_NAME> neighbor <X.X.X.X>
set protocols bgp group <GROUP_NAME> export NEXT_HOP_SELF_POLICY  # Next-hop-self example
set protocols bgp group <GROUP_NAME> export ADVERTISE_AGGREGATE   # Advertise an aggregate route example

# Next-hop-self example
set policy-options policy-statement NEXT_HOP_SELF_POLICY then next-hop self

# Advertise an aggregate route example
set routing-options aggregate route 172.24.0.0/22
set policy-options policy-statement ADVERTISE_AGGREGATE term MATCH_AGGREGATE from protocol aggregate
set policy-options policy-statement ADVERTISE_AGGREGATE term MATCH_AGGREGATE from route-filter 172.24.0.0/22 exact
set policy-options policy-statement ADVERTISE_AGGREGATE term MATCH_AGGREGATE then accept
```

### Redistribute connected into BGP

```
edit policy-options policy-statement BGP-connected
set term 1 from protocol direct
set term 1 then accept

top
edit protocols bgp group <GROUP-NAME>
set export BGP-connected
```

## Tunneling

- GRE adds 24 bytes of overhead to the packet.
- GRE uses IP protocol 47.
- GRE uses `gr-x/y/z` naming convention.
- GRE configuration example

```
show interfaces gr-0/0/0 terse
show route X.X.X.X
show interfaces gr-0/0/0.0 detail | find "traffic statistics"
```

```
set interfaces gr-0/0/0.0 tunnel source <X.X.X.X>
set interfaces gr-0/0/0.0 tunnel destination <Y.Y.Y.Y>
set interfaces gr-0/0/0.0 family inet

set routing-options static route 192.168.2.0/24 next-hop gr-0/0/0.0
```

- GRE keepalives configuration:

```
set protocols oam gre-tunnel interface gr-x/y/z.A keepalive-time 10
set protocols oam gre-tunnel interface gr-x/y/z.A hold-time 30
```


- IP over IP (IP-IP) adds 20 bytes of overhead to the packet.
- IP-IP can only tunnel IP traffic.
- IP-IP uses `ip-x/y/z` naming convention. 


Configure PMTUD (Path MTU Discovery)

```
set system internet-options gre-path-mtu-discovery|ipip-path-mtu-discovery
```

## Graceful Restart

```
set routing-options graceful-restart
```

This command restarts the **rpd** process.

```
restart routing
```

## BFD

```
show bfd session
```

Enabling BFD on OSPF

```
set protocols ospf area X interface ge-0/0/x.0 bfd-liveness-detection minimum-interval 300
```

Enabling BFD on BGP

```
set protocols bgp group <GROUP_NAME> neighbor <X.X.X.X> bfd-liveness-detection minimum-interval 300
```

## VRRP - Virtual Router Redundancy Protocol

Industry standard defined in **RFC 2338**.
Master & Backup Routers.
VRRP cummunication uses multicast destionation IP address of **224.0.0.18** with **TTL of 255**
Hello interval is **1 second** by default
**VRID** and authentication must match for VRRP routers to sync with each other
The Master Router replies to ARP requests with MAC **00:00:5E:00:01:VRID**
Default **priority is 100**. The VRRP Router with the highest priority get elected as the **Master Router**
If the VRRP Router owns the VIP address, the priority must be set to 255

|      Term      |                                                                     Description                                                                      |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| VRRP Router    | Any router participating in VRRP, including the master and all backup routers                                                                        |
| Master Router  | VRRP router performing packet forwarding and responding to ARP requests                                                                              |
| Backup Routers | VRRP router available to accept the role of the master router upon failure                                                                           |
| Virtual Router | Virtual entity that functionas as default router on a LAN; consists of virtual router ID and IP address used as gateway address known as VIP address |

VRRP States

| VRRP State |                                               Description                                               |
|------------|---------------------------------------------------------------------------------------------------------|
| Initialize | Router negotiates VRRP roles through startup events; no forwarding can be performed while in this state |
| Master     | Router assumes traffic forwarding responsibilities for the LAN and responds to ARP requests             |
| Backup     | Router monitors master VRRP router and is ready to assume forwarding responsibilities if failure occurs |
| Transition | Router switches between master and backup states; no forwarding can be performed while in this state    |

Sample VRRP configuration

R1 - Master Router

```
set interface ge-0/0/x.0 family inet address 192.168.1.2/24 vrrp-group <X> virtual-address 192.168.1.1 priority 100
```

R2 - Backup Router

```
set interface ge-0/0/x.0 family inet address 192.168.1.3/24 vrrp-group <X> virtual-address 192.168.1.1 priority 90
```

| Configuration Option |                                                                                     Description                                                                                     |
|----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| track                | Monitors state of specified interface (typically a WAN interface) or route and reduces designated priority value of VRRP group if tracker interface or route is no longer available |
| accepts-data         | Enables master router to respond to ICMP requests sent to VIP address-by default, master router does not respond                                                                    |
| authentication-type  | Authenticates VRRP messages between VRRP routers (type and key must match on all VRRP routers in the same group)                                                                    |
| authentication-key   |                                                                                                                                                                                     |
| no-preempt           | Disables preemption to avoid unwanted mastership changes. Note: Preemption is enabled by default, which means the router with the highest priority always assumes the master role   |

Show commands

```
show vrrp summary
```

## IPv6

```
show route table inet6
show route table inet6 protoco static
show ipv6 neighbors
show interface terse
```

### IPv6 Static Route configuration example

```
set routing-options rib inet.6 static route ::/0 next-hop fec0:0:0:2003::2
set routing-options rib inet.6 static route ::/0 preference 250
```

### OSPFv3 configuration example

```
set routing-options router-id 192.168.100.1
set protocols ospf3 area 0.0.0.0 interface ge-0/0/1.0
```

|       OSPFv2        |        OSPFv3        |
|---------------------|----------------------|
| show ospf neighbor  | show ospf3 neighbor  |
| show ospf interface | show ospf3 interface |
| show ospf database  | show ospf3 database  |
| show ospf route     | show ospf3 route     |

### IS-IS configuration example

```
show isis interface
show isis adjacency
show isis adjacency detail
```

```
set interfaces ge-0/0/1.0 family iso
set interfaces ge-0/0/1.0 family inet6 address fec0:0:0:2003::1/64

set interfaces lo0.0 family iso address 49.0002.1111.1111.1111.00
set interfaces lo0.0 family inet6 address fec0:0:0:1001::1/128

set protocols isis interface ge-0/0/1.0
set protocols isis interface lo0.0
```

### BGP configuration example

```
show bgp summary
```

```
set routing-options router-id 192.168.100.1
set routing-options autonomous-system 64700

set protocols bgp group INT-64700 type internal
set protocols bgp group INT-64700 local-address fec0:0:0:1001:1
set protocols bgp group INT-64700 neighbor fec0:0:0:1002:1

set protocols bgp group EXT-65100 type external
set protocols bgp group EXT-65100 peer-as 65100
set protocols bgp group EXT-65100 neighbor fec0:0:0:2005:2
```

### SRX IPv6 configuration

```
set security forwarding-options family inet6 mode packet-based
```
## IS-IS

```
show isis interface
show isis database
show isis database extensive
show isis adjacency
show isis adjacency detail
show isis spf log
show isis statistics
show isis route
show route protocol isis
clear isis adjacency <>
```

By default, all interfaces are L1 & L2. We can disable either in the `protocols isis` configuration hierarcy.

```
set protocols isis interface ge-0/0/0.0 level 1 disable
set protocols isis interface ge-0/0/1.0 level 2 disable
```

We must include the `family iso` on all interfaces on which we want to run IS-IS and a network entity title on one of the router's interfaces (usually **lo0**).

```
set interfaces ge-0/0/1.0 family iso

set interfaces lo0.0 family iso address 49.0002.1111.1111.1111.00

set protocols isis interface ge-0/0/1.0
set protocols isis interface lo0.0
```

To debug IS-IS, monitor the resulting **ISIS_TRACE** log file using `monitor start <LOG_FILE_NAME>` or the `show log <LOG_FILE_NAME>` commands.

```
set protocols isis traceoptions file ISIS_TRACE
set protocols isis traceoptions flag error detail
set protocols isis traceoptions flag hello detail
set protocols isis traceoptions flag lsp detail
```

## References

- [Complete JNCIS-ENT (YouTube playlist)](https://www.youtube.com/playlist?list=PLsPPnwREYxwvQMlVtfpKU34uTwShws-3b)
- [JUNOS RIB-GROUPS (1/2)](https://momcanfixanything.com/junos-rib-groups-1-2/)
- [Junos CLI Reference - rib-groups](https://www.juniper.net/documentation/us/en/software/junos/cli-reference/topics/ref/statement/rib-groups-edit-routing-options.html)
