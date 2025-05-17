---
title: Juniper Advanced Junos Enterprise Routing (AJER) notes
date: 2025-05-08 21:40:00 -0700
categories: [JUNIPER, JUNOS OS]
tags: [juniper]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/juniper_logo.png)

## OSPF

OSPF configuration

```
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0 interface-type p2p
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0 metric 100

set protocols ospf area 0.0.0.0 interface lo0.0
or
set protocols ospf area 0.0.0.0 interface 172.16.1.1   # Exact lo0 IP address

set protocols ospf area 0.0.0.1 nssa
set protocols ospf area 0.0.0.1 interface ge-0/0/3.0

set protocols ospf area 0.0.0.2 stub
set protocols ospf area 0.0.0.2 interface ge-0/0/4.0
```

Verify OSPF neighborship

```
show ospf neighbor
```

Set Router ID

```
set routing-options router-id 192.168.1.1
```

OSPF verification commands

```
show ospf database
show ospf database summary
show ospf database detail
show ospf database area x.x.x.x
```

Display OSPF learned routes

```
show ospf route
```

Display information about the Router LSA: Type 1

```
show ospf database router extensive
```

Display information about the Network LSA: Type 2

```
show ospf database network extensive
```

Display information about the Summary LSA: Type 3

```
show ospf database netsummary extensive
```

Display information about the ASBR SummaryLSA: Type 4

```
show ospf database asbrsummary extensive
```

Display information about the External LSA: Type 5

```
show ospf database external extensive
```

Display information about the NSSA LSA: Type 7

```
show ospf database nssa extensive
```

### OSPF Route Selection Order

1. Intra-Area links
2. Interarea links
3. External Type E1
4. External Type E2

### OSPF Reference Bandwidth

Default OSPF cost for all links is 10^8 / bandwidth (bps).
With these defauls, links with bandwidth >= 100 Mbps have a cost of 1.

Suggestion is to modify it:

```
set protocols ospf reference-bandwidth 1000g
```

### OSPF Cost on interfaces

```
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0 metric 12
set protocols ospf area 0.0.0.0 interface ge-0/0/1.100 metric 73
```

Verify OSPF interface metric with

```
show ospf interface ge-0/0/0.0 detail
```

### OSPF overload

Sets metric 65,535 in the Router LSA on all transit links.
The large metric values ensure that transit traffic through the overload router uses alternative paths.

```
set protocols ospf overload
```

### OSPF Authentication

- None (default)

- Simple

```
set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 authentication simple-password <PASSWORD>
```

- MD5

```
set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 authentication md5 <KEY_ID_0-255> key <PASSWORD>


set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 authentication md5 2 key <PASSWORD> start-time now
set protocols ospf area 0.0.0.0 interface ge-0/0/2.0 authentication md5 2 key <PASSWORD> start-time "2016-1-20.12:00:00 +0000"
```

Verify OSPF authentication with

```
show ospf interface detail
```

### OSPFv3

Must have `family inet6` configured since OSPFv3 uses IPv6 link-local addresses to pass messages between Routers on the same network segment.
Requires a Router-ID.

IPv6 OSPFv3

```
set protocols ospf3 ...
```

IPv4 OSPFv3

```
set protocols ospf3 realm ipv4-unicast ...
```

Flooding scope

| S2 | S1 |  Flooding Scope  |
|----|----|------------------|
|  0 |  0 | Link-local scope |
|  0 |  1 | Area scope       |
|  1 |  0 | AS scope         |
|  1 |  1 | Reserved         |

OSPFv3 LSA Types

- U-bit: Used to show how a router that does not understand the LS function code should handle the LSA
- S-bit and flooding scope: Used to show the flooding scope for the LSA

| LSA Function | LS type |      Description      |      Like OSPFv2       |
|--------------|---------|-----------------------|------------------------|
|            1 | 0x2001  | Router LSA            | Type 1 Router LSAs     |
|            2 | 0x2002  | Network LSA           | Type 2 Network LSAs    |
|            3 | 0x2003  | Inter-Area-Prefix LSA | Type 3 Summary LSAs    |
|            4 | 0x2004  | Inter-Area-Router LSA | Type 4 ASBR-Summary    |
|            5 | 0x4005  | AS-External-LSA       | Type 5 AS-External LSA |
|            6 | 0x2006  | Group Membership LSA  | Type 6 Multicast       |
|            7 | 0x2007  | Type-7 LSA            | Type 7 NSSA External   |
|            8 | 0x0008  | Link LSA              | None                   |
|            9 | 0x2009  | Intra-Area-Prefix LSA | Types 1 and 2          |

### OSPF debugging

```
set protocols ospf traceoptions file trace-ospf.log size 10m files 3
set protocols ospf traceoptions flag error detail
set protocols ospf traceoptions flag hello detail
set protocols ospf traceoptions flag hello send receive

show log trace-ospf.log
show log trace-ospf.log | match mismatch
```

To disable it

```
set protocols ospf deactivate traceoptions

run file delete /var/log/trace-ospf
```

### Stub Area configuration

```
set protocols ospf area 0.0.0.1 stub
```

This is the ABR configuration to inject an OSPF Type 3 Summary LSA `0.0.0./0` route into the Stub Area with a specific metric.

```
set protocols ospf area 0.0.0.1 stub default-metric 10
```

### Totally Stub Area configuration

The Totally Stub Area configuration is only required on the ABRs.

```
set protocols ospf area 0.0.0.1 stub no-summaries default-metric 10
```

### NSSA Area configuration

```
set protocols ospf area 0.0.0.3 nssa
```

For the ABR to inject a `0.0.0.0/0` default route into the NSSA area as **OSPF LSA Type 7**

```
set protocols ospf area 0.0.0.3 nssa default-lsa default-metric 10
```

When an ASBR is also an ABR with an NSSA area attached to it, all NSSA areas receive the LSA type 7 from it.
To disable the behaviour, use this command:

```
set protocols ospf no-nssa-abr
```

### Totally NSSA Area configuration

```
set protocols ospf area 0.0.0.3 nssa no-summaries
```

For the ABR to inject a `0.0.0.0/0` default route into the NSSA area as **OSPF LSA Type 3**

```
set protocols ospf area 0.0.0.3 nssa default-lsa default-metric 10
set protocols ospf area 0.0.0.3 nssa no-summaries
```

For the ABR to inject a `0.0.0.0/0` default route into the NSSA area as **OSPF LSA Type 7**

```
set protocols ospf area 0.0.0.3 nssa default-lsa default-metric 10
set protocols ospf area 0.0.0.3 nssa default-lsa type-7
set protocols ospf area 0.0.0.3 nssa no-summaries
```

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/ospf-nssa-default-route-types-01.png)

### Summarize OSPF Type 1 and Type 2 LSAs advertised to the Backbone Area by the ABR

```
set protocols ospf area 1 area-range 192.168.16/20
```

### Block OSPF Type 1 and Type 2 LSAs from being advertised to the Backbone Area by the ABR

```
set protocols ospf area 1 area-range 192.168.16/20 restric
```

### Summarize OSPF Type 7 LSAs advertised to the Backbone Area by the ABR

```
set protocols ospf area 1 nssa area-range  192.168.16/20
```

### Block OSPF Type 7 LSAs from being advertised to the Backbone Area by the ABR

```
set protocols ospf area 1 nssa area-range  192.168.16/20
```

### OSPF Multi-Area Adjacencies

Interfaces can belong to more than one Area.
In this example, `ge-0/0/1` is configured on both, **Area 0** and **Area 1**.

Note that the `secondary` is configured as **OSPF point-to-point**.

```
set protocols ospf area 0.0.0.0 interface ge-0/0/1.0
set protocols ospf area 0.0.0.1 interface ge-0/0/1.0 secondary
```

### OSPF Virtual Links

Router A

```
set protocols ospf area 0.0.0.0 virtual-link neighbor-id 192.168.0.2 transit-area 0.0.0.10
```

Router B

```
set protocols ospf area 0.0.0.0 virtual-link neighbor-id 172.16.0.4 transit-area 0.0.0.10
```

### Redistribute Static Routes into OSPF

By default external routes into OSPF are advertised as **type E2**.

```
set policy-options policy-statement REDISTRIBUTE-STATICS term STATIC-ROUTES from protocol static
set policy-options policy-statement REDISTRIBUTE-STATICS term STATIC-ROUTES then external type 1   <<< To make the routes type E1
set policy-options policy-statement REDISTRIBUTE-STATICS term STATIC-ROUTES then accept

set protocols ospf expoert REDISTRIBUTE-STATICS
```

### OSPF Prefix Limit

If the number of network prefixes to be redistributed into OSPF exceed `prefix-export-limit <#>` then none is redistributed.

```
set protocols ospf prefix-export-limit <#>
```

### Display Routing

| # |       Source       |         Command          |
|---|--------------------|--------------------------|
| 1 | Link State Databse | show ospf database       |
| 2 | Tree Database      | show ospf route          |
| 3 | inet.0             | show route protocol ospf |

### OSPF Neighbor State Machine

- Down
- Init
- 2Way
- ExStart
- Exchange
- Loading
- Full

NOTE: If 2 directly connected Routers have the same Router ID, OSPF will NOT form adjacency.

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/ospf-troubleshooting-1.png)

Items that Must Match:

1. Interface Types (p2p or MultiAccess)
2. Network (MultiAccess only)
3. Hello Intervals
4. Dead Intervals
5. Area Types
6. Area Numbers
7. Authentication

```
clear ospf statistics
show ospf statistics
```

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/ospf-troubleshooting-2.png)

Internal command that displays some timing statistics about SPF runs. Not very useful.

```
show ospf log
```

Monitor Traffic Interface

```
monitor traffic interface ge-0/0/1 detail no-resolve
monitor traffic interface ge-0/0/1 matching dst 224.0.0.5
```

### Troubleshoot OSPF Routing issues

- `show route protocol ospf` Displays OSPF-computed routes and their attributes
- `show ospf route` Notifies the type (intra-area, interarea, external type-1 and type-2 and so on) of each of the prefixes computed by OSPF
- `show ospf database` Enables you to check the content of the link-state database (LSDB)

## BGP

BGP Message Types:

- Open
- Keepalive
- Update
- Notification
- Refresh

BGP Neighbor States

| TCP Connectivity | BGP Connectivity |
|------------------|------------------|
| Idle             | OpenSent         |
| Connect          | OpenConfirm      |
| Active           | Established      |

### Configure BGP ASN and Router-ID

```
set routing-options autonomous-system 65503
set routing-options router-id 192.168.100.1
```

### iBGP Configuration example

```
set protocols bgp group int-65503 type internal
set protocols bgp group int-65503 local-address 192.168.100.1
set protocols bgp group int-65503 neighbor 192.168.100.2
```

### eBGP Configuration example

```
set protocols bgp group ext-65501 type external
set protocols bgp group ext-65501 peer-as 65501
set protocols bgp group ext-65501 neighbor 172.30.1.2
```

### BGP Authentication

BGP Authentication configured at BGP protocol level

```
set protocols bgp authentication-key <PASSWORD>
```

BGP Authentication configured at BGP group level

```
set protocols bgp group int-65503 authentication-key <PASSWORD>
```

BGP Authentication configured at BGP neighbor level

```
set protocols bgp group ext-65501 neighbor 172.30.1.2 authentication-key <PASSWORD>
```

BGP Authentication using Key Chains

```
set security authentication-key-chains key-chain KEY-CHAIN-NAME key 1 secret SECRET-DATA
set security authentication-key-chains key-chain KEY-CHAIN-NAME key 1 start-time YYYY-MM-DD.HH:MM:SS

set security authentication-key-chains key-chain KEY-CHAIN-NAME key 2 secret SECRET-DATA
set security authentication-key-chains key-chain KEY-CHAIN-NAME key 2 start-time YYYY-MM-DD.HH:MM:SS
```

```
set protocols bgp group int-65503 type internal
set protocols bgp group int-65503 local-address 192.168.100.1
set protocols bgp group int-65503 authentication-key-chain KEY-CHAIN-NAME
set protocols bgp group int-65503 neighbor 192.168.100.2
```

### BGP TTL Security - Generalized TTL Security Mechanism (GTSM)

BGP peer 1

```
set firewall filter TTL-SECURITY term GTSM from soruce-address 10.1.2.1/32
set firewall filter TTL-SECURITY term GTSM from protocol tcp
set firewall filter TTL-SECURITY term GTSM from ttl-except 255
set firewall filter TTL-SECURITY term GTSM from port 179
set firewall filter TTL-SECURITY term GTSM then discard

set firewall filter TTL-SECURITY term ELSE then accept

set interfaces ge-1/0/0 unit 0 family inet address 10.1.2.2/30 filter input TTL-SECURITY
```

BGP peer 2

```
set protocols bgp group toAS2 type external
set protocols bgp group toAS2 peer-as 2
set protocols bgp group toAS2 ttl 255           <<<<<<<<<<<<<<-----------
set protocols bgp group toAS2 neighbor 10.1.2.2
```

### Protecting BGP session

```
set protocols bgp group ext-peers family inet unicast prefix-limit maximum 25000
set protocols bgp group ext-peers family inet unicast prefix-limit teardown 80 idle-timeout 10   <<< Optional
```

### show route hidden extensive

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/bgp-showroute-hidden-extensive-output.png)

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/bgp-reasons-hidden-routes.png)

### Selecting the Active BGP Route

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/bgp-path-selection.png)

### BGP Multipath

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/bgp-multipath.png)

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/bgp-multipath-2.png)

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/bgp-multipath-3.png)

### BGP Multihop peering

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/bgp-multihop-peering.png)

### Multiple Hops with Per-Flow Load Balancing

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/per-flow-load-balancing.png)

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/per-flow-load-balancing-config-1.png)

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/per-flow-load-balancing-config-2.png)

### Advertise-inactive

Overrides the default behavior and advertise BGP routes that are not currently selected as active because of route preference.

```
set protocol bgp advertise-inactive
```

### Aggregate routes

```
set routing-options aggregate route 172.21.0.0/22
set routing-options aggregate route 172.22.0.0/22
set routing-options aggregate route 192.168.1.0/30
set routing-options aggregate route 192.168.2.0/30
```

```
set routing-options policy-options policy-statement ADV-AGGREGATES term MATCH-AGGREGATE-ROUTES from protocol aggregate
set routing-options policy-options policy-statement ADV-AGGREGATES term MATCH-AGGREGATE-ROUTES then accept
set routing-options policy-options policy-statement ADV-AGGREGATES term DENY-OTHER then reject
```

```
set protocols bgp group MY-EXT-GROUP export ADV-AGGREGATES
```

### Verify BGP

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/bgp-import.png)

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/bgp-export.png)

- RIB-IN
- RIB-LOCAL
- RIB-OUT

```
show route receive-protocol bgp <X.X.X.X> [hidden]
show route protocol bgp [source-gateway <X.X.X.X>]
show route advertising-protocol bgp <X.X.X.X>
```

```
show route aspath-regex "65510 .*"
show route 0/0 exact extensive
```

BGP sessions

```
show bgp summary
```

### Common BGP Path Attributes

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/bgp-common-path-attributes.png)

|         Attribute Name         |      Attribute Type      |
|--------------------------------|--------------------------|
| Next Hop                       | Well-known mandatory     |
| Local Preference               | Well-known discretionary |
| AS Path                        | Well-known mandatory     |
| Origin                         | Well-known mandatory     |
| Multi Exit Discriminator (MED) | Optional non-transitive  |
| Community                      | Optional transitive      |

### BGP Regex

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/bgp-regex.png)

### BGP Well-Known Communities

|        Type         |   Value    |                                                Notes                                                |
|---------------------|------------|-----------------------------------------------------------------------------------------------------|
| No-export           | 0xFFFFFF01 | These routes are nto advertised outside a BGP confederation or AS                                   |
| No-advertise        | 0xFFFFFF02 | These routes are not advertised to other BGP peers                                                  |
| No-export-subconfed | 0xFFFFFF03 | These routes are advertised to IBGP peers in the same AS but not to members of other confederations |

BGP Communities configuration

```
set policy-options community <COMMUNITY_NAME> members [ <COMMUNITY-ID> <COMMUNITY-ID> ... ]
```

```
set policy-options policy-statement <PS_NAME> term <X> from community <COMMUNITY_NAME>
set policy-options policy-statement <PS_NAME> term <X> then local-preference 200
set policy-options policy-statement <PS_NAME> term <X> then accept
```

```
set protocols bgp group <MY-GROUP> export <PS_NAME>
set protocols bgp group <MY-GROUP> neighbor <X.X.X.X>
```

### BGP Communities Regular Expressions

Looking for routes with communities matching a regex in the Routing Table

```
show route community *:20 terse
show route community *:20 detail
show route community-name <NAME> detail
```

![]({{ site.baseurl }}/images/2025/05-11-Juniper-AJER-course/bgp-communities-regex.png)

### BGP Next-Hop-Self

```
set policy-options policy-statement NHS term <X> then next-hop-self

set protocols bgp group <GROUP-NAME> export NHS
```

### Advertise only routes generated within the AS to the eBGP peer

```
set policy-options as-path null-as "()"

set policy-options policy-statement EXPORT-EBGP term LOCAL-ROUTES from as-path null-as
set policy-options policy-statement EXPORT-EBGP term LOCAL-ROUTES from protocol bgp
set policy-options policy-statement EXPORT-EBGP term LOCAL-ROUTES then accept
set policy-options policy-statement EXPORT-EBGP term LAST then reject
```

```
set protocols bgp group MY-EXT-GROUP exprt EXPORT-EBGP
```