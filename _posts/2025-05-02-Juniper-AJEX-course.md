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
show spanning-tree interface msti 1 detail
show spanning-tree interface msti 2 detail
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
show spanning-tree interface vlan-id XXX detail
show spanning-tree bridge
```

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/vstp-03.png)

## Authentication and Access Control

802.1X
MAC Radius
Captive Portal

Radius clients are network devices that rqeuest AAA services from a RADIUS server.

802.1X IEEE defines a method to authenticate and associate users with network access rights based on an assigned profile and VLAN.

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/8021x-01.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/8021x-02.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/8021x-03.png)

Dynamic VLAN Assigment

802.1X can dynamically associate hosts with their corresponding VLANs during authentication process.
RADIUS server returns VLAN attributes as part of the access-accept message (VLAN must be configured on the switch).

Guest VLAN

Provides limited access, typically just Internet access, for Guests and devices that don't support 802.1X authentication

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/8021x-05.png)

Suplicants can be moved to a specific VLAN upon the Authenticator receiving an access-reject message from the RADIUS Server.

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/8021x-06.png)

Server Fail Fallback Options

If RADIUS server fails to **respond or authenticate** a device, these are the supported actions:

- Permit: Enables traffic from devices as if they are successfully authenticated by the RADIUS server
- Deny: Prevents traffic from devices **(default behavior)**
- Move: Associates device with the specified VLAN
- Sustain: Maintains authentication for devices that already have LAN access and denies unauthenticated devices

Static MAC Bypass

Exclusion list specifying MAC addresses of devices allowed access to the ALN without authentication.
Commonly used with printers and IP phones.
You must use the **Multiple Supplicant** mode.

Configuring 802.1X

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/8021x-07.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/8021x-08.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/8021x-09.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/8021x-10.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/8021x-11.png)

## Access Control Features - MAC Radius and Captive Portal

### MAC RADIUS

Uses a centralized RADIUS Server and database to authenticate end user devices not 802.1X enabled.
Hosts that are not 802.1X-enabled can be authenticated using MAC RADIUS.

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/mac-radius-01.png)

MAC RADIUS and 802.1X on the same port

- MAC RADIUS can be configured in conjuction with 802.1X on ports that also serve 802.1X clients
- The switch uses the **Extensible Authentication Protocol-Message Digest 5 (EAP-MD5)** method, when both **802.1X amd MAC RADIUS** authentication methods are enabled on the same port
- With the preivuos case, if only **non-802.1X-enabled** end devices connect to a given interface, you can eliminate the delay (which can be upto 90 seconds) by configuring the `mac-radius restrict` option

Configuring MAC RADIUS

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/mac-radius-02.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/mac-radius-03.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/mac-radius-04.png)

### Captive Portal Access Control Features

Uses authentication with RADIUS server.
Captive Portal is enabled on individual interfaces on EX switches.
Captive Portal can be configured on Layer 2 interfaces.
Captive Portal **does not support** dynamic VLAN assigments through the RADIUS server.
Captive Portal is a supported fallback option for 802.1X.
By Default, Captive Portal uses the single supplicant mode.
Captive Portal limits the number of user authentication attempts, the default is 3. After these attempts, the wait period to retry is 60 seconds by default.
Upon expiry of the re-authentication timer, Captive Portal tears down the connection.
Sessions by default are 3600 seconds (1 hour), which can be changed in the configuration per interface or for all the interfaces with the command "session expiry".
When an end-user device opens a Web browswer and points to a remote website, the connected switch, with the Captive Portal feature enabled, intercepts the request and presents the user with the Captive Portal login HTML page. The Captive Portal login HTML page requests the user's username and password.
End-device should have IP/Mask, Gateway, DNS information so DHCP & DNS traffic is bypassed.

Authentication Allowlist

This is just to bypass devices from the Captive Portal (printers, IP Phones, etc) by having their MAC address statically configured in a local list in the Switch's configuration.

Captive Portal configuration

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/captiveportal-01.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/captiveportal-02.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/captiveportal-03.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/captiveportal-04.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/captiveportal-05.png)

Captive Portal Authentication Allowlist configuration

**NOTE**: Multiple supplicant must is required.

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/captiveportal-07.png)

Captive Portal verification

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/captiveportal-07.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/captiveportal-08.png)

### Authentication Process Considerations

Listing all authentication processes here:
- 802.1X
- MAC RADIUS
- Captive Portal
- MAC Allowlist & Static MAC Bypass
- Server-Reject VLAN & Guest VLAN

Captive Portal **cannot be mixed for active authentications** with 802.1X and/or MAC Radius.

Fallback order:
1. 802.1X Authentication
2. MAC RADIUS Authentication
3. Captive Portal Authentication

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/auth-proc-considerations-01.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/auth-proc-considerations-02.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/auth-proc-considerations-03.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/auth-proc-considerations-04.png)

### Configuring 802.1X

```
set access radius-server 172.25.11.254 secret Juniper
set access profile my-profile authentication-order radius
set access profile my-profile radius authentication-server 172.25.11.254

set protocols dot1x authenticator authentication-profile-name my-profile
set protocols dot1x authenticator interface ge-0/0/10
set protocols dot1x authenticator interface ge-0/0/12
```

Single-secure suplicant mode

```
set protocols dot1x authenticator interface ge-0/0/10 no-reauthentication   # Disabling re-authentication
set protocols dot1x authenticator interface ge-0/0/12 suplicant single-secure   # By default suplicant mode is "Single", we're changing that on this line
set protocols dot1x authenticator interface ge-0/0/12 reauthentication 7200    # By default it is 3600 (1 hour)
```

MAC Bypass

```
set protocols dot1x authenticator static 00:26:88:00:00:00/24
```

Multiple suplicant mode

```
set protocols dot1x authenticator interface ge-0/0/10 suplicant multiple
set protocols dot1x authenticator interface ge-0/0/12 suplicant multiple
```

MAC RADIUS

```
set protocols dot1x authenticator interface ge-0/0/10 mac-radius restrict
set protocols dot1x authenticator interface ge-0/0/12 mac-radius restrict
```

Fail fallback - allows traffic on port even when RADIUS was unresponsive to the request

```
set protocols dot1x authenticator interface ge-0/0/10 server-fail permit
set protocols dot1x authenticator interface ge-0/0/12 server-fail permit
```

Commands to verify the configuration

```
show dot1x interface
show dot1x interface detail
show dot1x interface ge-0/0/10 detail
show dot1x static-mac-address
```

## IP Telephony Features: Voice VLAN

The voice VLAN feature enables access ports to accept both untagged (data) and tagged (voice) traffic and separate that traffic into different VLANs.
Used with the class of service (CoS) to differentiate data and voice traffic
Voice VLAN and CoS values can be communicated to IP phones through LLDP-MED (Link Layer Discovery Protocol-Media Endpoint Discovery).

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/voice-vlan-01.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/voice-vlan-02.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/voice-vlan-03.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/voice-vlan-04.png)

Commands to verify

```
show lldp local-information
show lldp statistics interface ge-0/0/10
```

L3 port with 802.1Q / VLAN tagging

```
set interfaces ge-0/0/11 vlan-tagging
deactivate interfaces ge-0/0/11 unit 0
set interfaces ge-0/0/11 unit 25 vlan-id 25
set interfaces ge-0/0/11 unit 25 family inet addresss 172.23.25.11/24
```

Capture traffic on an interface

```
monitor traffic interface ge-0/0/10 print-ascii no-resolve detail
```

## IP Telephony Features: Power over Ethernet and Neighbor Discovery using LLDP

Power over Ethernet Plus (PoE+) was designed to support PDs with higher power level requirements.

- PoE (IEEE 802.3af) provides up to 15.4W of power
- PoE (IEEE 802.3at) provides up to 30W of power

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/poe-01.png)

Configuring PoE

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/poe-02.png)

PoE Verification Commands

```
show chassis hardware
show poe interface
show poe interface ge-0/0/0
```

When 802.1X is enabled, LLDP frames are not transmitted or received until the port is authenticated.

When an IP Phone and a PC both connected to the same 802.1X enabled port and only one of them is configured to be 802.1X supplicant, use the **Single supplicate mode** on the Switch's port. When both are configured as 802.1X supplicant use the **Multiple supplicant mode**.

Configuring LLDP

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/lldp-01.png)

Verification commands

```
show lldp detail
show lldp neighbors
show lldp local-info
show lldp statistics
```

## Class of Service

- **Policers** monitor and limit traffic on **ingress interfaces**
- **Shapers** monitor and limit traffic on **egress interfaces or queues**
  - **Port shaping** defines the maximum bandwidth allocated to a port
  - **Queue shaping** defines a limit at which queues transmit packets

```
set class-of-service code-point-aliases dscp custom-ef 101110
```

```
show class-of-service interface ge-0/0/6
show class-of-service code-point-aliases
show class-of-service forwarding-class
show interfaces ge-0/0/16 extensive
```

Rewrite rules are applied to logical interfaces

```
show class-of-service rewrite-rule type dscp
set interfaces ge-* unit * rewrite-rules dscp
```

Changing the Default behavior on EX Switches

The default behavior is overwritting the CoS fields, so in case the EX switch receives a frame with the CoS set to EF, the frame will be allocated to Queue 0 and forwarded with CoS all zeros.

Enabling Behavior Aggregate (BA)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/cos-01.png)

Defining Schedulers

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/cos-02.png)

Configuring Schedulers Maps

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/cos-03.png)

EZQoS configuration template

Template specifically designed for VoIP deployments

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/cos-04.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/cos-05.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/cos-06.png)

## Class of Service Implementation

1. Use EZQoS template to consistently implement class of service (CoS)
2. Use operational mode commands to verify proper operations.

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/cos-07.png)

![]({{ site.baseurl }}/images/2025/05-02-Juniper-AJEX-course/cos-08.png)

```
show class-of-service code-point-aliases dscp | match ef
show class-of-service forwarding-class
show class-of-service classifier name ezqos-dscp-classifier
show class-of-service interface ge-0/0/6
show interfaces ge-0/0/10 extensive | find "Queue counters"
show interfaces ge-0/0/10 extensive | find "CoS information"
show interfaces queue ge-0/0/10 egress
traceroute x.x.x.x tos 0
```

| command                         | description                                                                                                                 |
|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| show interfaces extensive       | Show interface statistics and Packet Forwarding Engine queue configuration                                                  |
| show interface queue            | Show detailed per-queue interface statistics                                                                                |
| show class-of-service interface | Display every CoS-related feature active on the interface                                                                   |
| traceroute                      | Can help for a summary checking of rewrite rules                                                                            |
| show firewall                   | Defines counters on multifield classifiers at the edge or forwarding classes in the core-is a valuable troubleshooting tool |
