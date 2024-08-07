---
title: Juniper Junos OS OSPF Virtual Links
date: 2024-07-14 21:30:00 -0700
categories: [JUNIPER, OSPF]
tags: [juniper, ospf]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

## Adding the OSPF Configuration to mxA# & mxC# Routers

### mxA

```
[edit protocols ospf]
lab@mxA# show | compare 
[edit protocols ospf]
+ area 0.0.0.0 {
+     interface lo0.0;
+     interface ge-0/0/0.0;
+ }
+ area 0.0.0.20 {
+     interface ge-0/0/1.0;
+ }
+ area 0.0.0.10 {
+     nssa {
+         default-lsa default-metric 10;
+     }
+     interface ge-0/0/3.0;
+ }

[edit protocols ospf]
lab@mxA# show | display set 
set protocols ospf area 0.0.0.0 interface lo0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set protocols ospf area 0.0.0.20 interface ge-0/0/1.0
set protocols ospf area 0.0.0.10 nssa default-lsa default-metric 10
set protocols ospf area 0.0.0.10 interface ge-0/0/3.0

[edit protocols ospf]
lab@mxA# 
```

### mxC

```
[edit protocols ospf]
lab@mxC# show | compare 
[edit protocols ospf]
+ area 0.0.0.0 {
+     interface lo0.0;
+     interface ge-0/0/0.0;
+ }
+ area 0.0.0.20 {
+     interface ge-0/0/1.0;
+ }
+ area 0.0.0.10 {
+     nssa {
+         default-lsa default-metric 10;
+     }
+     interface ge-0/0/3.0;
+ }

[edit protocols ospf]
lab@mxC# show | display set 
set protocols ospf area 0.0.0.0 interface lo0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set protocols ospf area 0.0.0.20 interface ge-0/0/1.0
set protocols ospf area 0.0.0.10 nssa default-lsa default-metric 10
set protocols ospf area 0.0.0.10 interface ge-0/0/3.0

[edit protocols ospf]
lab@mxC# 
```

## OSPF interface and neighborship status

### mxA

<details markdown=1>
<summary markdown="span">show ospf interface</summary>

```
[edit protocols ospf]
lab@mxA# run show ospf interface 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          BDR     0.0.0.0         172.31.100.1    172.16.1.1         1
lo0.0               DR      0.0.0.0         172.16.1.1      0.0.0.0            0
ge-0/0/3.0          BDR     0.0.0.10        172.16.1.2      172.16.1.1         1
ge-0/0/1.0          BDR     0.0.0.20        172.31.101.1    172.16.1.1         1

[edit protocols ospf]
lab@mxA# 
```
</details>

<details markdown=1>
<summary markdown="span">show ospf neighbor</summary>

```
[edit protocols ospf]
lab@mxA# run show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.121.2     ge-0/0/0.0             Full      172.31.100.1     128    37
10.0.10.2        ge-0/0/3.0             Full      172.16.1.2       128    31
172.22.123.2     ge-0/0/1.0             Full      172.31.101.1     128    34

[edit protocols ospf]
lab@mxA# 
```
</details>

### mxC

<details markdown=1>
<summary markdown="span">show ospf interface</summary>

```
[edit protocols ospf]
lab@mxC# run show ospf interface 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          BDR     0.0.0.0         172.31.100.1    172.16.2.1         1
lo0.0               DR      0.0.0.0         172.16.2.1      0.0.0.0            0
ge-0/0/3.0          BDR     0.0.0.10        172.16.2.2      172.16.2.1         1
ge-0/0/1.0          BDR     0.0.0.20        172.31.101.1    172.16.2.1         1

[edit protocols ospf]
lab@mxC# 
```
</details>

<details markdown=1>
<summary markdown="span">show ospf neighbor</summary>

```
[edit protocols ospf]
lab@mxC# run show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.122.2     ge-0/0/0.0             Full      172.31.100.1     128    33
10.0.20.2        ge-0/0/3.0             Full      172.16.2.2       128    37
172.22.124.2     ge-0/0/1.0             Full      172.31.101.1     128    36

[edit protocols ospf]
lab@mxC# 
```
</details>

## Adding the OSPF Virtual Link Configuration

### mxA

```
[edit protocols ospf]
lab@mxA# set area 0 virtual-link transit-area 20 neighbor-id 172.16.2.1 

[edit protocols ospf]
lab@mxA# show | compare 
[edit protocols ospf area 0.0.0.0]
+   virtual-link neighbor-id 172.16.2.1 transit-area 0.0.0.20;

[edit protocols ospf]
lab@mxA# show | display set 
set protocols ospf area 0.0.0.0 virtual-link neighbor-id 172.16.2.1 transit-area 0.0.0.20
set protocols ospf area 0.0.0.0 interface lo0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set protocols ospf area 0.0.0.20 interface ge-0/0/1.0
set protocols ospf area 0.0.0.10 nssa default-lsa default-metric 10
set protocols ospf area 0.0.0.10 interface ge-0/0/3.0

[edit protocols ospf]
lab@mxA# commit 
commit complete

[edit protocols ospf]
lab@mxA# 
```

### mxC

```
[edit protocols ospf]
lab@mxC# set area 0 virtual-link transit-area 20 neighbor-id 172.16.1.1 

[edit protocols ospf]
lab@mxC# show | compare 
[edit protocols ospf area 0.0.0.0]
+   virtual-link neighbor-id 172.16.1.1 transit-area 0.0.0.20;

[edit protocols ospf]
lab@mxC# show | display set 
set protocols ospf area 0.0.0.0 virtual-link neighbor-id 172.16.1.1 transit-area 0.0.0.20
set protocols ospf area 0.0.0.0 interface lo0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set protocols ospf area 0.0.0.20 interface ge-0/0/1.0
set protocols ospf area 0.0.0.10 nssa default-lsa default-metric 10
set protocols ospf area 0.0.0.10 interface ge-0/0/3.0

[edit protocols ospf]
lab@mxC# 
```

## OSPF interface and neighborship status with Virtual Link configuration

### mxA

<details markdown=1>
<summary markdown="span">show ospf interface</summary>

```
[edit protocols ospf]
lab@mxA# run show ospf interface 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          BDR     0.0.0.0         172.31.100.1    172.16.1.1         1
lo0.0               DR      0.0.0.0         172.16.1.1      0.0.0.0            0
vl-172.16.2.1       PtToPt  0.0.0.0         0.0.0.0         0.0.0.0            1
ge-0/0/3.0          BDR     0.0.0.10        172.16.1.2      172.16.1.1         1
ge-0/0/1.0          BDR     0.0.0.20        172.31.101.1    172.16.1.1         1

[edit protocols ospf]
lab@mxA# 
```
</details>

<details markdown=1>
<summary markdown="span">show ospf neighbor</summary>

```
[edit protocols ospf]
lab@mxA# run show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.121.2     ge-0/0/0.0             Full      172.31.100.1     128    30
172.22.124.1     vl-172.16.2.1          Full      172.16.2.1         0    33
10.0.10.2        ge-0/0/3.0             Full      172.16.1.2       128    39
172.22.123.2     ge-0/0/1.0             Full      172.31.101.1     128    38

[edit protocols ospf]
lab@mxA# 
```
</details>

### mxC

<details markdown=1>
<summary markdown="span">show ospf interface</summary>

```
[edit protocols ospf]
lab@mxC# run show ospf interface 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          BDR     0.0.0.0         172.31.100.1    172.16.2.1         1
lo0.0               DR      0.0.0.0         172.16.2.1      0.0.0.0            0
vl-172.16.1.1       PtToPt  0.0.0.0         0.0.0.0         0.0.0.0            1
ge-0/0/3.0          BDR     0.0.0.10        172.16.2.2      172.16.2.1         1
ge-0/0/1.0          BDR     0.0.0.20        172.31.101.1    172.16.2.1         1

[edit protocols ospf]
lab@mxC# 
```
</details>

<details markdown=1>
<summary markdown="span">show ospf neighbor</summary>

```
[edit protocols ospf]
lab@mxC# run show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.122.2     ge-0/0/0.0             Full      172.31.100.1     128    37
172.22.123.1     vl-172.16.1.1          Full      172.16.1.1         0    39
10.0.20.2        ge-0/0/3.0             Full      172.16.2.2       128    38
172.22.124.2     ge-0/0/1.0             Full      172.31.101.1     128    39

[edit protocols ospf]
lab@mxC# 
```
</details>

## Change on Route Table example on mxC

### Before Virtual Link

```
[edit protocols ospf]
lab@mxC# run show route 172.16.1.1 

inet.0: 23 destinations, 23 routes (23 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.1/32      *[OSPF/10] 00:01:32, metric 2
                    >  to 172.22.122.2 via ge-0/0/0.0

[edit protocols ospf]
lab@mxC# 
```

### After Virtual Link

```
[edit protocols ospf]
lab@mxC# run show route 172.16.1.1 

inet.0: 23 destinations, 23 routes (23 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.1/32      *[OSPF/10] 00:00:56, metric 2
                       to 172.22.122.2 via ge-0/0/0.0
                    >  to 172.22.124.2 via ge-0/0/1.0

[edit protocols ospf]
lab@mxC# 
```