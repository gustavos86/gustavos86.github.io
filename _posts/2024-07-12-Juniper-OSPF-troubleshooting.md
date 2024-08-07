---
title: Juniper Junos OS OSPF Troubleshooting
date: 2024-07-12 08:27:00 -0700
categories: [JUNIPER, OSPF]
tags: [juniper, ospf]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

## 1. OSPF Area mismatch

### Isssue

Connectivity is `mxA# [ge-0/0/0] <> [ge-0/0/0] mxB:R2#`. The OSPF adjacency is not coming up between them.

<details markdown=1>
<summary markdown="span">show ospf neighbor</summary>

```
lab@mxA> show ospf neighbor 

lab@mxA>




lab@mxB:R2> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.131.10    ge-0/0/2.0             Init      192.168.71.3     128    37

lab@mxB:R2> 
```
</details>

<details markdown=1>
<summary markdown="span">show ospf interface detail</summary>

```
lab@mxA> show ospf interface detail 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          DR      0.0.0.2         192.168.71.1    0.0.0.0            0
  Type: LAN, Address: 172.22.131.1, Mask: 255.255.255.252, MTU: 1500, Cost: 1
  DR addr: 172.22.131.1, Priority: 128
  Adj count: 0
  Hello: 10, Dead: 40, ReXmit: 5, Not Stub
  Auth type: None
  Protection type: None
  Topology default (ID 0) -> Cost: 1




lab@mxB:R2> show ospf interface detail 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          DR      0.0.0.1         192.168.71.2    0.0.0.0            0
  Type: LAN, Address: 172.22.131.2, Mask: 255.255.255.252, MTU: 1500, Cost: 1
  DR addr: 172.22.131.2, Priority: 128
  Adj count: 0
  Hello: 10, Dead: 40, ReXmit: 5, Not Stub
  Auth type: None
  Protection type: None
  Topology default (ID 0) -> Cost: 1 
```
</details>

### Troubleshooting

First of all, run `clear ospf statistics` to clear the OSPF statistics counters.

<details markdown=1>
<summary markdown="span">show ospf statistics | find errors</summary>

**area mismatches** counters are increasing on both Routers.

```
lab@mxA> show ospf statistics | find errors 
Receive errors:
  39 area mismatches

lab@mxA> show ospf statistics | find errors    
Receive errors:
  46 area mismatches

lab@mxA> 



lab@mxB:R2> show ospf statistics | find errors    
Receive errors:
  50 area mismatches
  49 netmask mismatches

lab@mxB:R2> 

lab@mxB:R2> show ospf statistics | find errors    
Receive errors:
  56 area mismatches
  55 netmask mismatches

lab@mxB:R2> 
```
</details>

### Solution

- `mxA# [ge-0/0/0]` is configured on OSPF **area 2**
- `mxB:R2# [ge-0/0/0]` is configured on OSPF **area 1**
- This OSPF adjacency should be on OSPF area 1

<details markdown=1>
<summary markdown="span">New Configuration</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# edit protocols ospf 

[edit protocols ospf]
lab@mxA# show 
area 0.0.0.2 {
    interface lo0.0;
    interface ge-0/0/0.0;
    interface ge-0/0/3.415;
}

[edit protocols ospf]
lab@mxA# rename area 2 to area 1 

[edit protocols ospf]
lab@mxA# show 
area 0.0.0.1 {
    interface lo0.0;
    interface ge-0/0/0.0;
    interface ge-0/0/3.415;
}

[edit protocols ospf]
lab@mxA# delete area 1 interface ge-0/0/3.415 

[edit protocols ospf]
lab@mxA# show 
area 0.0.0.1 {
    interface lo0.0;
    interface ge-0/0/0.0;
}

[edit protocols ospf]
lab@mxA# 

[edit protocols ospf]
lab@mxA# set area 1 interface ge-0/0/3.0 passive 

[edit protocols ospf]
lab@mxA# show 
area 0.0.0.1 {
    interface lo0.0;
    interface ge-0/0/0.0;
    interface ge-0/0/3.0 {
        passive;
    }
}

[edit protocols ospf]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 




lab@mxA# show | compare 
[edit protocols ospf]
+    area 0.0.0.1 {
+        interface lo0.0;
+        interface ge-0/0/0.0;
+        interface ge-0/0/3.415;
+    }
-    area 0.0.0.2 {
-        interface lo0.0;
-        interface ge-0/0/0.0;
-        interface ge-0/0/3.415;
-    }

[edit]
lab@mxA# 
```
</details>

### Verification

<details markdown=1>
<summary markdown="span">show ospf neighbor</summary>

```
lab@mxA> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.131.2     ge-0/0/0.0             Full      192.168.71.2     128    32

lab@mxA> 




lab@mxB:R2> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.131.10    ge-0/0/2.0             Init      192.168.71.3     128    35
172.22.131.1     ge-0/0/0.0             Full      192.168.71.1     128    35

lab@mxB:R2> 
```
</details>

## 2. Subnet Mask mismatch

### Issue

The OSPF adjacency is not coming up on `mxB:R2# [lt-0/0/10.0]`

<details markdown=1>
<summary markdown="span">show ospf interface</summary>

```
lab@mxB:R2> show ospf interface 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/2.0          DR      0.0.0.0         192.168.71.2    0.0.0.0            1
lo0.2               DR      0.0.0.0         192.168.71.2    0.0.0.0            0
lt-0/0/10.0         DR      0.0.0.0         192.168.71.2    0.0.0.0            0
ge-0/0/0.0          DR      0.0.0.1         192.168.71.2    192.168.71.1       1

lab@mxB:R2>
```
</details>

<details markdown=1>
<summary markdown="span">show ospf neighbor</summary>

```
lab@mxB:R2> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.131.10    ge-0/0/2.0             Init      192.168.71.3     128    31
172.22.131.1     ge-0/0/0.0             Full      192.168.71.1     128    34

lab@mxB:R2>
```
</details>

### Troubleshooting

First of all, run `clear ospf statistics` to clear the OSPF statistics counters.

<details markdown=1>
<summary markdown="span">show ospf statistics | find errors</summary>

**netmask mismatches**  counters are increasing.

```
lab@mxB:R2> clear ospf statistics 

lab@mxB:R2> 

lab@mxB:R2> show ospf statistics | find errors 
Receive errors:
  2 netmask mismatches

lab@mxB:R2> show ospf statistics | find errors               
Receive errors:
  9 netmask mismatches

lab@mxB:R2> 
```
</details>

<details markdown=1>
<summary markdown="span">show interfaces terse</summary>

We see the subnet mask of interface **lt-0/0/10.0** is **/24** as opposed to **/30**.

```
lab@mxB:R2> show interfaces terse | match inet | except : 
ge-0/0/0.0              up    up   inet     172.22.131.2/30 
ge-0/0/2.0              up    up   inet     172.22.131.9/30 
lt-0/0/10.0             up    up   inet     172.22.131.5/24 
lo0.2                   up    up   inet     192.168.71.2        --> 0/0

lab@mxB:R2>
```
</details>

### Solution

<details markdown=1>
<summary markdown="span">New Configuration</summary>

```
lab@mxB:R2> configure 
Entering configuration mode

[edit]
lab@mxB:R2# edit interfaces lt-0/0/10 

[edit interfaces lt-0/0/10]
lab@mxB:R2# show 
unit 0 {
    encapsulation ethernet;
    peer-unit 1;
    family inet {
        address 172.22.131.5/24;
    }
    family iso;
    family inet6;
    family mpls;
}

[edit interfaces lt-0/0/10]
lab@mxB:R2# delete unit 0 family inet address 172.22.131.5/24 

[edit interfaces lt-0/0/10]
lab@mxB:R2# set unit 0 family inet address 172.22.131.5/30       

[edit interfaces lt-0/0/10]
lab@mxB:R2# show | compare 
[edit logical-systems R2 interfaces lt-0/0/10 unit 0 family inet]
+     address 172.22.131.5/30;
-     address 172.22.131.5/24;

[edit interfaces lt-0/0/10]
lab@mxB:R2# commit and-quit 
commit complete
Exiting configuration mode

lab@mxB:R2> 
```
</details>

### Verification

<details markdown=1>
<summary markdown="span">show ospf neighbor</summary>

We see the OSPF adjacency over **lt-0/0/10.0** now UP.

```
lab@mxB:R2> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.131.10    ge-0/0/2.0             Init      192.168.71.3     128    32
172.22.131.6     lt-0/0/10.0            Full      192.168.71.4     128    32
172.22.131.1     ge-0/0/0.0             Full      192.168.71.1     128    33

lab@mxB:R2>
```
</details>

## 3. OSPF link type mismatch

### Issue

The OSPF adjacency is not coming up on `mxB:R3# [ge-0/0/3.0]`

<details markdown=1>
<summary markdown="span">show ospf interface</summary>

```
lab@mxB:R3> show ospf interface 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/3.0          PtToPt  0.0.0.0         0.0.0.0         0.0.0.0            0
ge-0/0/4.0          BDR     0.0.0.0         192.168.71.4    192.168.71.3       1
lo0.3               DR      0.0.0.0         192.168.71.3    0.0.0.0            0

lab@mxB:R3> 
```
</details>

<details markdown=1>
<summary markdown="span">show ospf neighbor</summary>

```
lab@mxB:R3> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.131.14    ge-0/0/4.0             Full      192.168.71.4     128    31

lab@mxB:R3> 
```
</details>

### Troubleshooting

First of all, run `clear ospf statistics` to clear the OSPF statistics counters.

<details markdown=1>
<summary markdown="span">show ospf statistics | find error</summary>

**Hellos received on point-to-point LAN with DR/BDR elected** counters are increasing.
We expect this OSPF link to perform an OSPF DR/BDR election.

```
lab@mxB:R3> show ospf statistics | find error  
Receive errors:
  129 Hellos received on point-to-point LAN with DR/BDR elected

lab@mxB:R3> 
```
</details>

### Solution

<details markdown=1>
<summary markdown="span">New Configuration</summary>

```
lab@mxB:R3> show configuration protocols ospf 
area 0.0.0.0 {
    interface ge-0/0/3.0 {
        interface-type p2p;
    }
    interface ge-0/0/4.0;
    interface lo0.3;
}

lab@mxB:R3> 

lab@mxB:R3> configure 
Entering configuration mode

[edit]
lab@mxB:R3# edit protocols ospf 

[edit protocols ospf]
lab@mxB:R3# delete area 0 interface ge-0/0/3.0 interface-type p2p  

[edit protocols ospf]
lab@mxB:R3# show 
area 0.0.0.0 {
    interface ge-0/0/3.0;
    interface ge-0/0/4.0;
    interface lo0.3;
}

[edit protocols ospf]
lab@mxB:R3# top   

[edit]
lab@mxB:R3# show | compare 
[edit logical-systems R3 protocols ospf area 0.0.0.0 interface ge-0/0/3.0]
-     interface-type p2p;

[edit]
lab@mxB:R3# commit and-quit 
commit complete
Exiting configuration mode

lab@mxB:R3> 
```
</details>

### Verification

<details markdown=1>
<summary markdown="span">show ospf neighbor</summary>

```
lab@mxB:R3> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.131.9     ge-0/0/3.0             Full      192.168.71.2     128    34
172.22.131.14    ge-0/0/4.0             Full      192.168.71.4     128    31

lab@mxB:R3> 
```
</details>

## 4. Route unreachable

### Issue

- The problem is that `192.168.71.5` is nowhere to be found in the Routing Table of `mxA` which is a Router in OSPF Area 1
- `192.168.71.5` is the interface loopback (and OSPF Router ID) of a Router in Area 2
- `mxB:R4` is the ABR connection OSPF Area 2 with the OSPF Backbone Area

mxA# in OSPF Area 1

<details markdown=1>
<summary markdown="span">show ospf route 192.168/16</summary>

```
lab@mxA> show ospf route 192.168/16 
Topology default Route Table:

Prefix             Path  Route      NH       Metric NextHop       Nexthop      
                   Type  Type       Type            Interface     Address/LSP
192.168.71.2       Intra Area BR    IP            1 ge-0/0/0.0    172.22.131.2
192.168.71.1/32    Intra Network    IP            0 lo0.0
192.168.71.2/32    Inter Network    IP            1 ge-0/0/0.0    172.22.131.2
192.168.71.3/32    Inter Network    IP            2 ge-0/0/0.0    172.22.131.2
192.168.71.4/32    Inter Network    IP            2 ge-0/0/0.0    172.22.131.2

lab@mxA> 
```
</details>

<details markdown=1>
<summary markdown="span">show ospf database netsummary</summary>


```
lab@mxA> show ospf database netsummary 

    OSPF database, Area 0.0.0.1
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Summary  172.22.131.4     192.168.71.2     0x80000002   391  0x22 0xd769  28
Summary  172.22.131.8     192.168.71.2     0x80000003   173  0x22 0xad8e  28
Summary  172.22.131.12    192.168.71.2     0x80000002   173  0x22 0x91a6  28
Summary  172.22.131.63    192.168.71.2     0x80000001   391  0x22 0x340d  28
Summary  192.168.71.2     192.168.71.2     0x80000003   189  0x22 0xa431  28
Summary  192.168.71.3     192.168.71.2     0x80000002   173  0x22 0xa62e  28
Summary  192.168.71.4     192.168.71.2     0x80000001   391  0x22 0x9e36  28

lab@mxA> 
```
</details>

### Troubleshooting

mxB:R2# is the ABR between OSPF Area 1 and the OSPF Backbone Area.

<details markdown=1>
<summary markdown="span">set cli logical-system R2</summary>

```
lab@mxB:R4> set cli logical-system R2
Logical system: R2

lab@mxB:R2>
```
</details>

<details markdown=1>
<summary markdown="span">show ospf database area 0 netsummary</summary>

```
lab@mxB:R2> show ospf database area 0 netsummary 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Summary *172.22.131.0     192.168.71.2     0x80000005   693  0x22 0xf948  28
Summary  172.22.131.0     192.168.71.4     0x80000002  1400  0x22 0x94e9  28
Summary *172.22.131.36    192.168.71.2     0x80000001   693  0x22 0xa27e  28
Summary *192.168.71.1     192.168.71.2     0x80000001   693  0x22 0xbc1b  28

lab@mxB:R2> 
```
</details><br />

mxB:R4# is the ABR between OSPF Area 2 and the OSPF Backbone Area.

<details markdown=1>
<summary markdown="span">set cli logical-system R4</summary>

```
lab@mxB:R2> set cli logical-system R4
Logical system: R4

lab@mxB:R4>
```
</details>

<details markdown=1>
<summary markdown="span">show ospf interface</summary>

```
lab@mxB:R4> show ospf interface 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/5.0          DR      0.0.0.0         192.168.71.4    192.168.71.3       1
lo0.4               DR      0.0.0.0         192.168.71.4    0.0.0.0            0
lt-0/0/10.1         DR      0.0.0.0         192.168.71.4    192.168.71.2       1
ge-0/0/1.0          BDR     0.0.0.2         192.168.71.5    192.168.71.4       1

lab@mxB:R4> 
```
</details>

<details markdown=1>
<summary markdown="span">show ospf neighbor</summary>

```
lab@mxB:R4> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.131.13    ge-0/0/5.0             Full      192.168.71.3     128    30
172.22.131.5     lt-0/0/10.1            Full      192.168.71.2     128    39
172.22.131.18    ge-0/0/1.0             Full      192.168.71.5     128    36

lab@mxB:R4> 
```
</details>

<details markdown=1>
<summary markdown="span">show ospf database area 2 router</summary>

```
lab@mxB:R4> show ospf database area 2 router 

    OSPF database, Area 0.0.0.2
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router  *192.168.71.4     192.168.71.4     0x80000003  1468  0x22 0xb85e  36
Router   192.168.71.5     192.168.71.5     0x80000004  1469  0x22 0x933e  60

lab@mxB:R4>
```
</details>

<details markdown=1>
<summary markdown="span">show ospf database area 2 lsa-id 192.168.71.5 extensive</summary>

```
lab@mxB:R4> show ospf database area 2 lsa-id 192.168.71.5 extensive 

    OSPF database, Area 0.0.0.2
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   192.168.71.5     192.168.71.5     0x80000004  1496  0x22 0x933e  60
  bits 0x0, link count 3
  id 172.22.131.18, data 172.22.131.18, Type Transit (2)
    Topology count: 0, Default metric: 1
  id 172.22.131.40, data 255.255.255.252, Type Stub (3)
    Topology count: 0, Default metric: 1
  id 192.168.71.5, data 255.255.255.255, Type Stub (3)
    Topology count: 0, Default metric: 0
  Topology default (ID 0)
    Type: Transit, Node ID: 172.22.131.18
      Metric: 1, Bidirectional
  Aging timer 00:35:04
  Installed 00:24:53 ago, expires in 00:35:04
  Last changed 00:24:53 ago, Change count: 2

lab@mxB:R4> 
```

</details>

<details markdown=1>
<summary markdown="span">show configuration protocols ospf</summary>

The problem is that on the ABR, the configuration line `area-range 0.0.0.0/0 restrict` is filtering all OSPF LSA type 3 from being advertised to the OSPF Backbone area.

```
lab@mxB:R4> show configuration protocols ospf 
area 0.0.0.0 {
    interface ge-0/0/5.0;
    interface lt-0/0/10.1;
    interface lo0.4;
}
area 0.0.0.2 {
    area-range 172.22.131.0/26;
    area-range 0.0.0.0/0 restrict;
    interface ge-0/0/1.0;
}
lab@mxB:R4>
```
</details>

### Solution

</details>

<details markdown=1>
<summary markdown="span">New Configuration</summary>

Adding `area-range 192.168.71.5/32` to allow the OSPF LSA type 3 for it into the OSPF Backbone Area.

```
lab@mxB:R4> configure 
Entering configuration mode

[edit]
lab@mxB:R4# set protocols ospf area 2 area-range 192.168.71.5/32 

[edit]
lab@mxB:R4# show protocols ospf 
area 0.0.0.0 {
    interface ge-0/0/5.0;
    interface lt-0/0/10.1;
    interface lo0.4;
}
area 0.0.0.2 {
    area-range 172.22.131.0/26;
    area-range 0.0.0.0/0 restrict;
    area-range 192.168.71.5/32;
    interface ge-0/0/1.0;
}

[edit]
lab@mxB:R4# show | compare 
[edit logical-systems R4 protocols ospf area 0.0.0.2]
      area-range 0.0.0.0/0 { ... }
+     area-range 192.168.71.5/32;

[edit]
lab@mxB:R4# 
```
</details>

### Verification

<details markdown=1>
<summary markdown="span">show ospf database area 0 netsummary</summary>

```
lab@mxB:R4> show ospf database area 0 netsummary 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Summary  172.22.131.0     192.168.71.2     0x80000005   903  0x22 0xf948  28
Summary *172.22.131.0     192.168.71.4     0x80000003     7  0x22 0x92ea  28
Summary  172.22.131.36    192.168.71.2     0x80000001   903  0x22 0xa27e  28
Summary  192.168.71.1     192.168.71.2     0x80000001   903  0x22 0xbc1b  28
Summary *192.168.71.5     192.168.71.4     0x80000001     7  0x22 0x8849  28

lab@mxB:R4> 
```
</details>
