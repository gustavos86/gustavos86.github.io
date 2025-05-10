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

Set Router ID

```
set routing-options router-id 192.168.1.1
```

OSPF verification commands

```
show ospf database detail
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
