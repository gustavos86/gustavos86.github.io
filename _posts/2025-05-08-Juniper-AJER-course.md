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
show ospf database detail
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
set protocols ospf traceoptions file trace-ospf
set protocols ospf traceoptions flag error detail
```

To disable it

```
set protocols ospf deactivate traceoptions

run file delete /var/log/trace-ospf
```