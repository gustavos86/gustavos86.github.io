---
title: Juniper Junos OS OSPF Area types
date: 2024-07-10 19:12:00 -0700
categories: [JUNIPER, OSPF]
tags: [juniper, ospf]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

## Network Topology

![]({{ site.baseurl }}/images/2024/07-07-Juniper-OSPF-basic-configs/01-OSPF_Lab_diagram.png)

## Initial Configs

### mxA

```
lab@mxA> show configuration | display set 
set version 20190829.221548_builder.r1052644
set system host-name mxA
set system root-authentication encrypted-password "$1$KI99zGk6$MbYFuBbpLffu9tn2.sI7l1"
set system root-authentication ssh-dsa "ssh-dss AAAAB3NzaC1kc3MAAACBAMQrfP2bZyBXJ6PC7XXZ+MzErI8Jl6jah5L4/O8BsfP2hC7EvRfNoX7MqbrtCX/9gUH9gChVuBCB+ERULMdgRvM5uGhC/gs4UX+4dBbfBgKYYwgmisM8EoT25m7qI8ybpl2YZvHNznvO8h7kr4kpYuQEpKvgsTdH/Jle4Uqnjv7DAAAAFQDZaqA6QAgbW3O/zveaLCIDj6p0dwAAAIB1iL+krWrXiD8NPpY+w4dWXEqaV3bnobzPC4eyxQKBUCOr80Q5YBlWXVBHx9elwBWZwj0SF4hLKHznExnLerVsMuTMA846RbQmSz62vM6kGM13HFonWeQvWia0TDr78+rOEgWF2KHBSIxL51lmIDW8Gql9hJfD/Dr/NKP97w3L0wAAAIEAr3FkWU8XbYytQYEKxsIN9P1UQ1ERXB3G40YwqFO484SlyKyYCfaz+yNsaAJu2C8UebDIR3GieyNcOAKf3inCG8jQwjLvZskuZwrvlsz/xtcxSoAh9axJcdUfSJYMW/g+mD26JK1Cliw5rwp2nH9kUrJxeI7IReDp4egNkM4i15o= configurator@server1.he"
set system login user lab uid 2000
set system login user lab class super-user
set system login user lab authentication encrypted-password "$6$JEnFYM1n$C6pjHzEv3cK/iovqkiJywOgyrmgNnX/U0r3B3kfaeXa4ygKFE1l7De7YsKVbjdJnab3PfylSLmDHjgPVMiilP1"
set system services ssh root-login allow
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set interfaces ge-0/0/0 unit 0 family inet address 172.22.121.1/24
set interfaces ge-0/0/3 unit 0 family inet address 10.0.1.1/24
set interfaces fxp0 unit 0 family inet address 172.25.11.1/24
set interfaces lo0 unit 0 family inet address 172.16.1.1/32
set policy-options policy-statement static-to-ospf term 1 from protocol static
set policy-options policy-statement static-to-ospf term 1 then accept
set routing-options static route 20.20.0.0/24 reject
set routing-options static route 20.20.1.0/24 reject
set routing-options static route 20.20.2.0/24 reject
set routing-options static route 20.20.3.0/24 reject
set routing-options autonomous-system 65512
set protocols ospf area 0.0.0.0 interface lo0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set protocols ospf area 0.0.0.10 interface ge-0/0/3.0
set protocols ospf export static-to-ospf

lab@mxA> 
```

<details markdown=1>
<summary markdown="span">diffs</summary>

```
lab@mxA# show | compare    
[edit]
- version 20190829.221548_builder.r1052644;
+ protocols {
+     ospf {
+         area 0.0.0.0 {
+             interface lo0.0;
+             interface ge-0/0/0.0;
+         }
+         area 0.0.0.10 {
+             interface ge-0/0/3.0;
+         }
+         export static-to-ospf;
+     }
+ }

[edit]
lab@mxA# 

```
</details><br />

### mxB

```
lab@mxB> show configuration | display set 
set version 20190829.221548_builder.r1052644
set system host-name mxB
set system root-authentication encrypted-password "$1$KI99zGk6$MbYFuBbpLffu9tn2.sI7l1"
set system root-authentication ssh-dsa "ssh-dss AAAAB3NzaC1kc3MAAACBAMQrfP2bZyBXJ6PC7XXZ+MzErI8Jl6jah5L4/O8BsfP2hC7EvRfNoX7MqbrtCX/9gUH9gChVuBCB+ERULMdgRvM5uGhC/gs4UX+4dBbfBgKYYwgmisM8EoT25m7qI8ybpl2YZvHNznvO8h7kr4kpYuQEpKvgsTdH/Jle4Uqnjv7DAAAAFQDZaqA6QAgbW3O/zveaLCIDj6p0dwAAAIB1iL+krWrXiD8NPpY+w4dWXEqaV3bnobzPC4eyxQKBUCOr80Q5YBlWXVBHx9elwBWZwj0SF4hLKHznExnLerVsMuTMA846RbQmSz62vM6kGM13HFonWeQvWia0TDr78+rOEgWF2KHBSIxL51lmIDW8Gql9hJfD/Dr/NKP97w3L0wAAAIEAr3FkWU8XbYytQYEKxsIN9P1UQ1ERXB3G40YwqFO484SlyKyYCfaz+yNsaAJu2C8UebDIR3GieyNcOAKf3inCG8jQwjLvZskuZwrvlsz/xtcxSoAh9axJcdUfSJYMW/g+mD26JK1Cliw5rwp2nH9kUrJxeI7IReDp4egNkM4i15o= configurator@server1.he"
set system login user lab uid 2000
set system login user lab class super-user
set system login user lab authentication encrypted-password "$6$JEnFYM1n$C6pjHzEv3cK/iovqkiJywOgyrmgNnX/U0r3B3kfaeXa4ygKFE1l7De7YsKVbjdJnab3PfylSLmDHjgPVMiilP1"
set system services ssh root-login allow
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set interfaces ge-0/0/0 unit 0 family inet address 172.22.121.2/24
set interfaces ge-0/0/1 unit 0 family inet address 172.22.122.2/24
set interfaces fxp0 unit 0 family inet address 172.25.11.2/24
set interfaces lo0 unit 0 family inet address 172.31.100.1/32
set protocols ospf area 0.0.0.0 interface lo0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/1.0

lab@mxB> 
```

<details markdown=1>
<summary markdown="span">diffs</summary>

```
```
</details><br />

### mxC

```
lab@mxC> show configuration | display set 
set version 20190829.221548_builder.r1052644
set system host-name mxC
set system root-authentication encrypted-password "$1$KI99zGk6$MbYFuBbpLffu9tn2.sI7l1"
set system root-authentication ssh-dsa "ssh-dss AAAAB3NzaC1kc3MAAACBAMQrfP2bZyBXJ6PC7XXZ+MzErI8Jl6jah5L4/O8BsfP2hC7EvRfNoX7MqbrtCX/9gUH9gChVuBCB+ERULMdgRvM5uGhC/gs4UX+4dBbfBgKYYwgmisM8EoT25m7qI8ybpl2YZvHNznvO8h7kr4kpYuQEpKvgsTdH/Jle4Uqnjv7DAAAAFQDZaqA6QAgbW3O/zveaLCIDj6p0dwAAAIB1iL+krWrXiD8NPpY+w4dWXEqaV3bnobzPC4eyxQKBUCOr80Q5YBlWXVBHx9elwBWZwj0SF4hLKHznExnLerVsMuTMA846RbQmSz62vM6kGM13HFonWeQvWia0TDr78+rOEgWF2KHBSIxL51lmIDW8Gql9hJfD/Dr/NKP97w3L0wAAAIEAr3FkWU8XbYytQYEKxsIN9P1UQ1ERXB3G40YwqFO484SlyKyYCfaz+yNsaAJu2C8UebDIR3GieyNcOAKf3inCG8jQwjLvZskuZwrvlsz/xtcxSoAh9axJcdUfSJYMW/g+mD26JK1Cliw5rwp2nH9kUrJxeI7IReDp4egNkM4i15o= configurator@server1.he"
set system login user lab uid 2000
set system login user lab class super-user
set system login user lab authentication encrypted-password "$6$JEnFYM1n$C6pjHzEv3cK/iovqkiJywOgyrmgNnX/U0r3B3kfaeXa4ygKFE1l7De7YsKVbjdJnab3PfylSLmDHjgPVMiilP1"
set system services ssh root-login allow
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set interfaces ge-0/0/0 unit 0 family inet address 172.22.122.1/24
set interfaces ge-0/0/3 unit 0 family inet address 10.0.2.1/24
set interfaces fxp0 unit 0 family inet address 172.25.11.3/24
set interfaces lo0 unit 0 family inet address 172.16.2.1/32
set policy-options policy-statement static-to-ospf term 1 from protocol static
set policy-options policy-statement static-to-ospf term 1 then accept
set routing-options static route 20.20.4.0/24 reject
set routing-options static route 20.20.5.0/24 reject
set routing-options static route 20.20.6.0/24 reject
set routing-options static route 20.20.7.0/24 reject
set routing-options autonomous-system 65512
set protocols ospf area 0.0.0.0 interface lo0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set protocols ospf area 0.0.0.20 interface ge-0/0/3.0
set protocols ospf export static-to-ospf

```

<details markdown=1>
<summary markdown="span">diffs</summary>

```
lab@mxC>

lab@mxC# show | compare 
[edit]
- version 20190829.221548_builder.r1052644;
+ protocols {
+     ospf {
+         area 0.0.0.0 {
+             interface lo0.0;
+             interface ge-0/0/0.0;
+         }
+         area 0.0.0.20 {
+             interface ge-0/0/3.0;
+         }
+         export static-to-ospf;
+     }
+ }

[edit]
lab@mxC#

```
</details><br />

### mxD

```
lab@mxD> show configuration | display set 
set version 20190829.221548_builder.r1052644
set system host-name mxD
set system root-authentication encrypted-password "$1$KI99zGk6$MbYFuBbpLffu9tn2.sI7l1"
set system root-authentication ssh-dsa "ssh-dss AAAAB3NzaC1kc3MAAACBAMQrfP2bZyBXJ6PC7XXZ+MzErI8Jl6jah5L4/O8BsfP2hC7EvRfNoX7MqbrtCX/9gUH9gChVuBCB+ERULMdgRvM5uGhC/gs4UX+4dBbfBgKYYwgmisM8EoT25m7qI8ybpl2YZvHNznvO8h7kr4kpYuQEpKvgsTdH/Jle4Uqnjv7DAAAAFQDZaqA6QAgbW3O/zveaLCIDj6p0dwAAAIB1iL+krWrXiD8NPpY+w4dWXEqaV3bnobzPC4eyxQKBUCOr80Q5YBlWXVBHx9elwBWZwj0SF4hLKHznExnLerVsMuTMA846RbQmSz62vM6kGM13HFonWeQvWia0TDr78+rOEgWF2KHBSIxL51lmIDW8Gql9hJfD/Dr/NKP97w3L0wAAAIEAr3FkWU8XbYytQYEKxsIN9P1UQ1ERXB3G40YwqFO484SlyKyYCfaz+yNsaAJu2C8UebDIR3GieyNcOAKf3inCG8jQwjLvZskuZwrvlsz/xtcxSoAh9axJcdUfSJYMW/g+mD26JK1Cliw5rwp2nH9kUrJxeI7IReDp4egNkM4i15o= configurator@server1.he"
set system login user lab uid 2000
set system login user lab class super-user
set system login user lab authentication encrypted-password "$6$JEnFYM1n$C6pjHzEv3cK/iovqkiJywOgyrmgNnX/U0r3B3kfaeXa4ygKFE1l7De7YsKVbjdJnab3PfylSLmDHjgPVMiilP1"
set system services ssh root-login allow
set system services netconf ssh
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set logical-systems R3-1 interfaces ge-0/0/1 unit 0 family inet address 10.0.1.2/24
set logical-systems R3-1 interfaces lo0 unit 1 family inet address 172.16.1.2/32
set logical-systems R3-1 protocols ospf area 0.0.0.10 interface ge-0/0/1.0
set logical-systems R3-1 protocols ospf area 0.0.0.10 interface lo0.1
set logical-systems R3-1 routing-options autonomous-system 65512
set logical-systems R3-2 interfaces ge-0/0/4 unit 0 family inet address 10.0.2.2/24
set logical-systems R3-2 interfaces lo0 unit 2 family inet address 172.16.2.2/32
set logical-systems R3-2 protocols ospf area 0.0.0.20 interface ge-0/0/4.0
set logical-systems R3-2 protocols ospf area 0.0.0.20 interface lo0.2
set logical-systems R3-2 routing-options autonomous-system 65512
set interfaces fxp0 unit 0 family inet address 172.25.11.4/24

lab@mxD> 
```

<details markdown=1>
<summary markdown="span">diffs</summary>

```
lab@mxD# show | compare 
[edit]
- version 20190829.221548_builder.r1052644;
[edit logical-systems R3-1]
+   protocols {
+       ospf {
+           area 0.0.0.10 {
+               interface ge-0/0/1.0;
+               interface lo0.1;
+           }
+       }
+   }
[edit logical-systems R3-2]
+   protocols {
+       ospf {
+           area 0.0.0.20 {
+               interface ge-0/0/4.0;
+               interface lo0.2;
+           }
+       }
+   }

[edit]
lab@mxD#
```
</details><br />

## Verification of OSPF adjacencies

### mxA <> mxD:R3-1 (OSPF area 10)

```
lab@mxA> show ospf neighbor 10.0.1.2 
Address          Interface              State     ID               Pri  Dead
10.0.1.2         ge-0/0/3.0             Full      172.16.1.2       128    34
```

<details markdown=1>
<summary markdown="span">OSPF LSAs that mxA is generating</summary>

```
lab@mxA> show ospf database area 10 advertising-router 172.16.1.1 

    OSPF database, Area 0.0.0.10
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router  *172.16.1.1       172.16.1.1       0x80000003   114  0x22 0xa3f3  36
Summary *172.16.1.1       172.16.1.1       0x80000002   124  0x22 0x4d71  28
Summary *172.22.121.0     172.16.1.1       0x80000003   113  0x22 0xe955  28
Summary *172.22.122.0     172.16.1.1       0x80000001   113  0x22 0xec52  28
Summary *172.31.100.1     172.16.1.1       0x80000001   113  0x22 0x5fec  28
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern  *20.20.0.0        172.16.1.1       0x80000001   128  0x22 0x6b60  36
Extern  *20.20.1.0        172.16.1.1       0x80000001   128  0x22 0x606a  36
Extern  *20.20.2.0        172.16.1.1       0x80000001   128  0x22 0x5574  36
Extern  *20.20.3.0        172.16.1.1       0x80000001   128  0x22 0x4a7e  36

lab@mxA> 
```
</details><br />

### mxB <> mxA (OSPF area 0)

```
lab@mxB> show ospf neighbor 172.22.121.1 
Address          Interface              State     ID               Pri  Dead
172.22.121.1     ge-0/0/0.0             Full      172.16.1.1       128    38

lab@mxB> 
```

### mxC (to OSPF Area 0 & to OSPF Area 20)

```
lab@mxC> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.122.2     ge-0/0/0.0             Full      172.31.100.1     128    38
10.0.2.2         ge-0/0/3.0             Full      172.16.2.2       128    31

lab@mxC> 
```

<details markdown=1>
<summary markdown="span">OSPF LSAs that mxC is generating</summary>

```
lab@mxC> show ospf database advertising-router 172.16.2.1 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router  *172.16.2.1       172.16.2.1       0x80000004   167  0x22 0xea75  48
Summary *10.0.2.0         172.16.2.1       0x80000003   192  0x22 0x501e  28
Summary *172.16.2.2       172.16.2.1       0x80000001   192  0x22 0x3d7e  28

    OSPF database, Area 0.0.0.20
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router  *172.16.2.1       172.16.2.1       0x80000003   193  0x22 0xa7eb  36
Summary *10.0.1.0         172.16.2.1       0x80000001   192  0x22 0x73fb  28
Summary *172.16.1.1       172.16.2.1       0x80000001   192  0x22 0x5c60  28
Summary *172.16.1.2       172.16.2.1       0x80000001   192  0x22 0x5c5e  28
Summary *172.16.2.1       172.16.2.1       0x80000002   202  0x22 0x3b81  28
Summary *172.22.121.0     172.16.2.1       0x80000001   192  0x22 0xf04e  28
Summary *172.22.122.0     172.16.2.1       0x80000003   192  0x22 0xd765  28
Summary *172.31.100.1     172.16.2.1       0x80000001   192  0x22 0x58f2  28
ASBRSum *172.16.1.1       172.16.2.1       0x80000001   192  0x22 0x4e6d  28
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern  *20.20.4.0        172.16.2.1       0x80000001   207  0x22 0x388e  36
Extern  *20.20.5.0        172.16.2.1       0x80000001   207  0x22 0x2d98  36
Extern  *20.20.6.0        172.16.2.1       0x80000001   207  0x22 0x22a2  36
Extern  *20.20.7.0        172.16.2.1       0x80000001   207  0x22 0x17ac  36

lab@mxC> 
```
</details><br />

## OSPF Stub Areas

### Before

<details markdown=1>
<summary markdown="span">outputs</summary>

```
lab@mxD:R3-1> show route 20.20/16 

inet.0: 19 destinations, 19 routes (19 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

20.20.0.0/24       *[OSPF/150] 00:02:35, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.1.0/24       *[OSPF/150] 00:02:35, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.2.0/24       *[OSPF/150] 00:02:35, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.3.0/24       *[OSPF/150] 00:02:35, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.4.0/24       *[OSPF/150] 00:02:35, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.5.0/24       *[OSPF/150] 00:02:35, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.6.0/24       *[OSPF/150] 00:02:35, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.7.0/24       *[OSPF/150] 00:02:35, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0

lab@mxD:R3-1> 
lab@mxD:R3-1> show ospf database 

    OSPF database, Area 0.0.0.10
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   172.16.1.1       172.16.1.1       0x80000003   291  0x22 0xa3f3  36
Router  *172.16.1.2       172.16.1.2       0x80000004   290  0x22 0x596d  48
Network *10.0.1.2         172.16.1.2       0x80000001   290  0x22 0xf5f8  32
Summary  10.0.2.0         172.16.1.1       0x80000001   289  0x22 0x6fff  28
Summary  172.16.1.1       172.16.1.1       0x80000002   299  0x22 0x4d71  28
Summary  172.16.2.1       172.16.1.1       0x80000001   289  0x22 0x5864  28
Summary  172.16.2.2       172.16.1.1       0x80000001   289  0x22 0x5862  28
Summary  172.22.121.0     172.16.1.1       0x80000003   289  0x22 0xe955  28
Summary  172.22.122.0     172.16.1.1       0x80000001   289  0x22 0xec52  28
Summary  172.31.100.1     172.16.1.1       0x80000001   289  0x22 0x5fec  28
ASBRSum  172.16.2.1       172.16.1.1       0x80000001   289  0x22 0x4a71  28
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern   20.20.0.0        172.16.1.1       0x80000001   304  0x22 0x6b60  36
Extern   20.20.1.0        172.16.1.1       0x80000001   304  0x22 0x606a  36
Extern   20.20.2.0        172.16.1.1       0x80000001   304  0x22 0x5574  36
Extern   20.20.3.0        172.16.1.1       0x80000001   304  0x22 0x4a7e  36
Extern   20.20.4.0        172.16.2.1       0x80000001   354  0x22 0x388e  36
Extern   20.20.5.0        172.16.2.1       0x80000001   354  0x22 0x2d98  36
Extern   20.20.6.0        172.16.2.1       0x80000001   354  0x22 0x22a2  36
Extern   20.20.7.0        172.16.2.1       0x80000001   354  0x22 0x17ac  36

lab@mxD:R3-1> 
```
</details><br />

### Configuration

`set protocols ospf area 10 stub`

<details markdown=1>
<summary markdown="span">Outputs on mxA & mxD:R3-1</summary>

```
configure
set protocols ospf area 10 stub
commit and-quit


lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# edit protocols ospf 

[edit protocols ospf]
lab@mxA# set area 10 stub 

[edit protocols ospf]
lab@mxA# commit 
commit complete

[edit protocols ospf]
lab@mxA# 


lab@mxD:R3-1> configure        
Entering configuration mode

[edit]
lab@mxD:R3-1# edit protocols ospf 

[edit protocols ospf]
lab@mxD:R3-1# set area 10 stub  

[edit protocols ospf]
lab@mxD:R3-1# commit 
commit complete

[edit protocols ospf]
lab@mxD:R3-1#
```
</details><br />

<details markdown=1>
<summary markdown="span">OSPF came UP between Routers in Area 10. OSPF Stub configuration is working properly</summary>

```
lab@mxD:R3-1# run show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
10.0.1.1         ge-0/0/1.0             Full      172.16.1.1       128    39

[edit protocols ospf]
lab@mxD:R3-1# 

lab@mxA# run show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.121.2     ge-0/0/0.0             Full      172.31.100.1     128    38
10.0.1.2         ge-0/0/3.0             Full      172.16.1.2       128    32

[edit protocols ospf]
lab@mxA# 

```
</details><br />

### After

<details markdown=1>
<summary markdown="span">outputs</summary>

```
lab@mxD:R3-1> show route 20.20/16    

lab@mxD:R3-1> 

lab@mxD:R3-1> show ospf database     

    OSPF database, Area 0.0.0.10
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   172.16.1.1       172.16.1.1       0x80000003    25  0x20 0xbbdf  36
Router  *172.16.1.2       172.16.1.2       0x80000003    24  0x20 0x7950  48
Network *10.0.1.2         172.16.1.2       0x80000001    24  0x20 0x14dc  32
Summary  10.0.2.0         172.16.1.1       0x80000001    65  0x20 0x8de3  28
Summary  172.16.1.1       172.16.1.1       0x80000001    65  0x20 0x6d54  28
Summary  172.16.2.1       172.16.1.1       0x80000001    65  0x20 0x7648  28
Summary  172.16.2.2       172.16.1.1       0x80000001    65  0x20 0x7646  28
Summary  172.22.121.0     172.16.1.1       0x80000001    65  0x20 0xc37   28
Summary  172.22.122.0     172.16.1.1       0x80000001    65  0x20 0xb36   28
Summary  172.31.100.1     172.16.1.1       0x80000001    65  0x20 0x7dd0  28
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern   20.20.0.0        172.16.1.1       0x80000001   417  0x22 0x6b60  36
Extern   20.20.1.0        172.16.1.1       0x80000001   417  0x22 0x606a  36
Extern   20.20.2.0        172.16.1.1       0x80000001   417  0x22 0x5574  36
Extern   20.20.3.0        172.16.1.1       0x80000001   417  0x22 0x4a7e  36
Extern   20.20.4.0        172.16.2.1       0x80000001   467  0x22 0x388e  36
Extern   20.20.5.0        172.16.2.1       0x80000001   467  0x22 0x2d98  36
Extern   20.20.6.0        172.16.2.1       0x80000001   467  0x22 0x22a2  36
Extern   20.20.7.0        172.16.2.1       0x80000001   467  0x22 0x17ac  36

lab@mxD:R3-1>
```
</details><br />

## OSPF No Summaries Areas (Totally Stub)

### Before

<details markdown=1>
<summary markdown="span">outputs</summary>

```
lab@mxA> show ospf database 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router  *172.16.1.1       172.16.1.1       0x80000005    85  0x22 0xcf94  48
Router   172.16.2.1       172.16.2.1       0x80000004  1283  0x22 0xea75  48
Router   172.31.100.1     172.31.100.1     0x80000023  1265  0x22 0x2a38  60
Network  172.22.121.2     172.31.100.1     0x80000001  1265  0x22 0x409a  32
Network  172.22.122.2     172.31.100.1     0x80000002    96  0x22 0x4097  32
Summary *10.0.1.0         172.16.1.1       0x80000005   880  0x22 0x5e10  28
Summary  10.0.2.0         172.16.2.1       0x80000003  1307  0x22 0x501e  28
Summary *172.16.1.2       172.16.1.1       0x80000001   880  0x22 0x4f6e  28
Summary  172.16.2.2       172.16.2.1       0x80000001  1307  0x22 0x3d7e  28

    OSPF database, Area 0.0.0.10
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router  *172.16.1.1       172.16.1.1       0x80000003   880  0x20 0xbbdf  36
Router   172.16.1.2       172.16.1.2       0x80000003   881  0x20 0x7950  48
Network  10.0.1.2         172.16.1.2       0x80000001   881  0x20 0x14dc  32
Summary *10.0.2.0         172.16.1.1       0x80000001   920  0x20 0x8de3  28
Summary *172.16.1.1       172.16.1.1       0x80000001   920  0x20 0x6d54  28
Summary *172.16.2.1       172.16.1.1       0x80000001   920  0x20 0x7648  28
Summary *172.16.2.2       172.16.1.1       0x80000001   920  0x20 0x7646  28
Summary *172.22.121.0     172.16.1.1       0x80000001   920  0x20 0xc37   28
Summary *172.22.122.0     172.16.1.1       0x80000001   920  0x20 0xb36   28
Summary *172.31.100.1     172.16.1.1       0x80000001   920  0x20 0x7dd0  28
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern  *20.20.0.0        172.16.1.1       0x80000002   801  0x22 0x6961  36
Extern  *20.20.1.0        172.16.1.1       0x80000002   630  0x22 0x5e6b  36
Extern  *20.20.2.0        172.16.1.1       0x80000002   459  0x22 0x5375  36
Extern  *20.20.3.0        172.16.1.1       0x80000002   287  0x22 0x487f  36
Extern   20.20.4.0        172.16.2.1       0x80000002   850  0x22 0x368f  36
Extern   20.20.5.0        172.16.2.1       0x80000002   677  0x22 0x2b99  36
Extern   20.20.6.0        172.16.2.1       0x80000002   504  0x22 0x20a3  36
Extern   20.20.7.0        172.16.2.1       0x80000002   331  0x22 0x15ad  36

lab@mxA> 

lab@mxD:R3-1> show route    

inet.0: 11 destinations, 11 routes (11 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.1.0/24        *[Direct/0] 23:32:50
                    >  via ge-0/0/1.0
10.0.1.2/32        *[Local/0] 23:32:50
                       Local via ge-0/0/1.0
10.0.2.0/24        *[OSPF/10] 00:00:05, metric 4
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.1.1/32      *[OSPF/10] 00:00:05, metric 1
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.1.2/32      *[Direct/0] 23:33:09
                    >  via lo0.1
172.16.2.1/32      *[OSPF/10] 00:00:05, metric 3
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.2.2/32      *[OSPF/10] 00:00:05, metric 4
                    >  to 10.0.1.1 via ge-0/0/1.0
172.22.121.0/24    *[OSPF/10] 00:00:05, metric 2
                    >  to 10.0.1.1 via ge-0/0/1.0
172.22.122.0/24    *[OSPF/10] 00:00:05, metric 3
                    >  to 10.0.1.1 via ge-0/0/1.0
172.31.100.1/32    *[OSPF/10] 00:00:05, metric 2
                    >  to 10.0.1.1 via ge-0/0/1.0
224.0.0.5/32       *[OSPF/10] 00:27:47, metric 1
                       MultiRecv

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

ff02::2/128        *[INET6/0] 23:33:12
                       MultiRecv

lab@mxD:R3-1>
```
</details><br />

### Configuration

`set protocols ospf area 10 stub no-summaries`

<details markdown=1>
<summary markdown="span">outputs</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# set protocols ospf area 10 stub no-summaries 

[edit]
lab@mxA# show | compare 
[edit protocols ospf area 0.0.0.10 stub]
+     no-summaries;

[edit]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA>
```
</details><br />

### After

<details markdown=1>
<summary markdown="span">outputs</summary>

```
lab@mxA> show ospf database 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router  *172.16.1.1       172.16.1.1       0x80000005   162  0x22 0xcf94  48
Router   172.16.2.1       172.16.2.1       0x80000004  1360  0x22 0xea75  48
Router   172.31.100.1     172.31.100.1     0x80000023  1342  0x22 0x2a38  60
Network  172.22.121.2     172.31.100.1     0x80000001  1342  0x22 0x409a  32
Network  172.22.122.2     172.31.100.1     0x80000002   173  0x22 0x4097  32
Summary *10.0.1.0         172.16.1.1       0x80000006    30  0x22 0x5c11  28
Summary  10.0.2.0         172.16.2.1       0x80000003  1384  0x22 0x501e  28
Summary *172.16.1.2       172.16.1.1       0x80000002    30  0x22 0x4d6f  28
Summary  172.16.2.2       172.16.2.1       0x80000001  1384  0x22 0x3d7e  28

    OSPF database, Area 0.0.0.10
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router  *172.16.1.1       172.16.1.1       0x80000005    30  0x20 0xb7e1  36
Router   172.16.1.2       172.16.1.2       0x80000005    31  0x20 0x7552  48
Network  10.0.1.2         172.16.1.2       0x80000003    31  0x20 0x10de  32
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern  *20.20.0.0        172.16.1.1       0x80000002   878  0x22 0x6961  36
Extern  *20.20.1.0        172.16.1.1       0x80000002   707  0x22 0x5e6b  36
Extern  *20.20.2.0        172.16.1.1       0x80000002   536  0x22 0x5375  36
Extern  *20.20.3.0        172.16.1.1       0x80000002   364  0x22 0x487f  36
Extern   20.20.4.0        172.16.2.1       0x80000002   927  0x22 0x368f  36
Extern   20.20.5.0        172.16.2.1       0x80000002   754  0x22 0x2b99  36
Extern   20.20.6.0        172.16.2.1       0x80000002   581  0x22 0x20a3  36
Extern   20.20.7.0        172.16.2.1       0x80000002   408  0x22 0x15ad  36

lab@mxA> 

lab@mxD:R3-1> show route 

inet.0: 4 destinations, 4 routes (4 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.1.0/24        *[Direct/0] 23:32:19
                    >  via ge-0/0/1.0
10.0.1.2/32        *[Local/0] 23:32:19
                       Local via ge-0/0/1.0
172.16.1.2/32      *[Direct/0] 23:32:38
                    >  via lo0.1
224.0.0.5/32       *[OSPF/10] 00:27:16, metric 1
                       MultiRecv

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

ff02::2/128        *[INET6/0] 23:32:41
                       MultiRecv

lab@mxD:R3-1> 

```
</details><br />

## Advertising Default Route in Totally Stub Area

### Before

<details markdown=1>
<summary markdown="span">outputs</summary>

```
lab@mxD:R3-1> show route    

inet.0: 4 destinations, 4 routes (4 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.1.0/24        *[Direct/0] 23:33:43
                    >  via ge-0/0/1.0
10.0.1.2/32        *[Local/0] 23:33:43
                       Local via ge-0/0/1.0
172.16.1.2/32      *[Direct/0] 23:34:02
                    >  via lo0.1
224.0.0.5/32       *[OSPF/10] 00:28:40, metric 1
                       MultiRecv

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

ff02::2/128        *[INET6/0] 23:34:05
                       MultiRecv

lab@mxD:R3-1> 

lab@mxD:R3-1> show ospf database 

    OSPF database, Area 0.0.0.10
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   172.16.1.1       172.16.1.1       0x80000009    50  0x20 0xafe5  36
Router  *172.16.1.2       172.16.1.2       0x80000009    49  0x20 0x6d56  48
Network *10.0.1.2         172.16.1.2       0x80000007    49  0x20 0x8e2   32
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern   20.20.0.0        172.16.1.1       0x80000001  1659  0x22 0x6b60  36
Extern   20.20.1.0        172.16.1.1       0x80000001  1659  0x22 0x606a  36
Extern   20.20.2.0        172.16.1.1       0x80000001  1659  0x22 0x5574  36
Extern   20.20.3.0        172.16.1.1       0x80000001  1659  0x22 0x4a7e  36
Extern   20.20.4.0        172.16.2.1       0x80000001  1709  0x22 0x388e  36
Extern   20.20.5.0        172.16.2.1       0x80000001  1709  0x22 0x2d98  36
Extern   20.20.6.0        172.16.2.1       0x80000001  1709  0x22 0x22a2  36
Extern   20.20.7.0        172.16.2.1       0x80000001  1709  0x22 0x17ac  36

lab@mxD:R3-1> 

lab@mxD:R3-1> ping 172.16.2.2 count 5 
PING 172.16.2.2 (172.16.2.2): 56 data bytes
ping: sendto: No route to host
ping: sendto: No route to host
ping: sendto: No route to host
ping: sendto: No route to host
ping: sendto: No route to host

--- 172.16.2.2 ping statistics ---
5 packets transmitted, 0 packets received, 100% packet loss

lab@mxD:R3-1> 
```
</details><br />

### Configuration

`set protocols ospf area 10 stub default-metric 10`

<details markdown=1>
<summary markdown="span">outputs</summary>

```
[edit]
lab@mxA# set protocols ospf area 10 stub default-metric 10 

[edit]
lab@mxA# show | compare 
[edit protocols ospf area 0.0.0.10 stub]
+     default-metric 10;

[edit]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA>
```
</details><br />

### After

<details markdown=1>
<summary markdown="span">outputs</summary>

```
lab@mxD:R3-1> show route 

inet.0: 5 destinations, 5 routes (5 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0.0.0.0/0          *[OSPF/10] 00:00:18, metric 11
                    >  to 10.0.1.1 via ge-0/0/1.0
10.0.1.0/24        *[Direct/0] 23:36:02
                    >  via ge-0/0/1.0
10.0.1.2/32        *[Local/0] 23:36:02
                       Local via ge-0/0/1.0
172.16.1.2/32      *[Direct/0] 23:36:21
                    >  via lo0.1
224.0.0.5/32       *[OSPF/10] 00:30:59, metric 1
                       MultiRecv

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

ff02::2/128        *[INET6/0] 23:36:24
                       MultiRecv

lab@mxD:R3-1> show ospf database 

    OSPF database, Area 0.0.0.10
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   172.16.1.1       172.16.1.1       0x80000009   149  0x20 0xafe5  36
Router  *172.16.1.2       172.16.1.2       0x80000009   148  0x20 0x6d56  48
Network *10.0.1.2         172.16.1.2       0x80000007   148  0x20 0x8e2   32
Summary  0.0.0.0          172.16.1.1       0x80000001    25  0x20 0x6c0a  28
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern   20.20.0.0        172.16.1.1       0x80000001  1758  0x22 0x6b60  36
Extern   20.20.1.0        172.16.1.1       0x80000001  1758  0x22 0x606a  36
Extern   20.20.2.0        172.16.1.1       0x80000001  1758  0x22 0x5574  36
Extern   20.20.3.0        172.16.1.1       0x80000001  1758  0x22 0x4a7e  36
Extern   20.20.4.0        172.16.2.1       0x80000001  1808  0x22 0x388e  36
Extern   20.20.5.0        172.16.2.1       0x80000001  1808  0x22 0x2d98  36
Extern   20.20.6.0        172.16.2.1       0x80000001  1808  0x22 0x22a2  36
Extern   20.20.7.0        172.16.2.1       0x80000001  1808  0x22 0x17ac  36

lab@mxD:R3-1> 

lab@mxD:R3-1> ping 172.16.2.2 count 5 
PING 172.16.2.2 (172.16.2.2): 56 data bytes
64 bytes from 172.16.2.2: icmp_seq=0 ttl=61 time=4.022 ms
64 bytes from 172.16.2.2: icmp_seq=1 ttl=61 time=3.244 ms
64 bytes from 172.16.2.2: icmp_seq=2 ttl=61 time=3.799 ms
64 bytes from 172.16.2.2: icmp_seq=3 ttl=61 time=3.825 ms
64 bytes from 172.16.2.2: icmp_seq=4 ttl=61 time=3.597 ms

--- 172.16.2.2 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/stddev = 3.244/3.697/4.022/0.264 ms

lab@mxD:R3-1> 
```
</details><br />

## OSPF Not-So-stubby-Areas (NSSA)

### Before

<details markdown=1>
<summary markdown="span">outputs</summary>

```
lab@mxD:R3-1> show ospf database 

    OSPF database, Area 0.0.0.10
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   172.16.1.1       172.16.1.1       0x80000004    10  0x22 0xa1f4  36
Router  *172.16.1.2       172.16.1.2       0x80000003     9  0x22 0x5b6c  48
Network *10.0.1.2         172.16.1.2       0x80000001     9  0x22 0xf5f8  32
Summary  10.0.2.0         172.16.1.1       0x80000001    54  0x22 0x6fff  28
Summary  172.16.1.1       172.16.1.1       0x80000001    54  0x22 0x4f70  28
Summary  172.16.2.1       172.16.1.1       0x80000001    54  0x22 0x5864  28
Summary  172.16.2.2       172.16.1.1       0x80000001    54  0x22 0x5862  28
Summary  172.22.121.0     172.16.1.1       0x80000001    54  0x22 0xed53  28
Summary  172.22.122.0     172.16.1.1       0x80000001    54  0x22 0xec52  28
Summary  172.31.100.1     172.16.1.1       0x80000001    54  0x22 0x5fec  28
ASBRSum  172.16.2.1       172.16.1.1       0x80000001    54  0x22 0x4a71  28
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern   20.20.0.0        172.16.1.1       0x80000003   117  0x22 0x6762  36
Extern   20.20.1.0        172.16.1.1       0x80000002  1327  0x22 0x5e6b  36
Extern   20.20.2.0        172.16.1.1       0x80000002  1156  0x22 0x5375  36
Extern   20.20.3.0        172.16.1.1       0x80000002   984  0x22 0x487f  36
Extern   20.20.4.0        172.16.2.1       0x80000002  1547  0x22 0x368f  36
Extern   20.20.5.0        172.16.2.1       0x80000002  1374  0x22 0x2b99  36
Extern   20.20.6.0        172.16.2.1       0x80000002  1201  0x22 0x20a3  36
Extern   20.20.7.0        172.16.2.1       0x80000002  1028  0x22 0x15ad  36

lab@mxD:R3-1>

lab@mxD:R3-1> show route 20.20/16 

inet.0: 19 destinations, 19 routes (19 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

20.20.0.0/24       *[OSPF/150] 00:00:32, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.1.0/24       *[OSPF/150] 00:00:32, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.2.0/24       *[OSPF/150] 00:00:32, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.3.0/24       *[OSPF/150] 00:00:32, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.4.0/24       *[OSPF/150] 00:00:32, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.5.0/24       *[OSPF/150] 00:00:32, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.6.0/24       *[OSPF/150] 00:00:32, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.7.0/24       *[OSPF/150] 00:00:32, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0

lab@mxD:R3-1> 
```
</details><br />

### Configuration

`set protocols ospf area 10 nssa`

<details markdown=1>
<summary markdown="span">outputs</summary>

```
[edit]
lab@mxA# set protocols ospf area 10 nssa 

[edit]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 
```
</details><br />

### After

<details markdown=1>
<summary markdown="span">outputs</summary>

```
lab@mxD:R3-1> show ospf database 

    OSPF database, Area 0.0.0.10
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   172.16.1.1       172.16.1.1       0x80000003   385  0x20 0xc1d7  36
Router  *172.16.1.2       172.16.1.2       0x80000003   384  0x20 0x7950  48
Network *10.0.1.2         172.16.1.2       0x80000001   384  0x20 0x14dc  32
Summary  10.0.2.0         172.16.1.1       0x80000001   426  0x20 0x8de3  28
Summary  172.16.1.1       172.16.1.1       0x80000001   426  0x20 0x6d54  28
Summary  172.16.2.1       172.16.1.1       0x80000001   426  0x20 0x7648  28
Summary  172.16.2.2       172.16.1.1       0x80000001   426  0x20 0x7646  28
Summary  172.22.121.0     172.16.1.1       0x80000001   426  0x20 0xc37   28
Summary  172.22.122.0     172.16.1.1       0x80000001   426  0x20 0xb36   28
Summary  172.31.100.1     172.16.1.1       0x80000001   426  0x20 0x7dd0  28
NSSA     20.20.0.0        172.16.1.1       0x80000001   426  0x20 0x6d5e  36
NSSA     20.20.1.0        172.16.1.1       0x80000001   426  0x20 0x6268  36
NSSA     20.20.2.0        172.16.1.1       0x80000001   426  0x20 0x5772  36
NSSA     20.20.3.0        172.16.1.1       0x80000001   426  0x20 0x4c7c  36
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern   20.20.0.0        172.16.1.1       0x80000003   631  0x22 0x6762  36
Extern   20.20.1.0        172.16.1.1       0x80000002  1841  0x22 0x5e6b  36
Extern   20.20.2.0        172.16.1.1       0x80000002  1670  0x22 0x5375  36
Extern   20.20.3.0        172.16.1.1       0x80000002  1498  0x22 0x487f  36
Extern   20.20.4.0        172.16.2.1       0x80000002  2061  0x22 0x368f  36
Extern   20.20.5.0        172.16.2.1       0x80000002  1888  0x22 0x2b99  36
Extern   20.20.6.0        172.16.2.1       0x80000002  1715  0x22 0x20a3  36
Extern   20.20.7.0        172.16.2.1       0x80000002  1542  0x22 0x15ad  36

lab@mxD:R3-1>

lab@mxD:R3-1> show route 20.20/16   

inet.0: 15 destinations, 15 routes (15 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

20.20.0.0/24       *[OSPF/150] 00:06:30, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.1.0/24       *[OSPF/150] 00:06:30, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.2.0/24       *[OSPF/150] 00:06:30, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.3.0/24       *[OSPF/150] 00:06:30, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0

lab@mxD:R3-1>
```
</details><br />

## OSPF NSSA No Summaries (Totally NSSA)

### Before

<details markdown=1>
<summary markdown="span">outputs</summary>

```
lab@mxD:R3-1> show ospf database 

    OSPF database, Area 0.0.0.10
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   172.16.1.1       172.16.1.1       0x80000003   385  0x20 0xc1d7  36
Router  *172.16.1.2       172.16.1.2       0x80000003   384  0x20 0x7950  48
Network *10.0.1.2         172.16.1.2       0x80000001   384  0x20 0x14dc  32
Summary  10.0.2.0         172.16.1.1       0x80000001   426  0x20 0x8de3  28
Summary  172.16.1.1       172.16.1.1       0x80000001   426  0x20 0x6d54  28
Summary  172.16.2.1       172.16.1.1       0x80000001   426  0x20 0x7648  28
Summary  172.16.2.2       172.16.1.1       0x80000001   426  0x20 0x7646  28
Summary  172.22.121.0     172.16.1.1       0x80000001   426  0x20 0xc37   28
Summary  172.22.122.0     172.16.1.1       0x80000001   426  0x20 0xb36   28
Summary  172.31.100.1     172.16.1.1       0x80000001   426  0x20 0x7dd0  28
NSSA     20.20.0.0        172.16.1.1       0x80000001   426  0x20 0x6d5e  36
NSSA     20.20.1.0        172.16.1.1       0x80000001   426  0x20 0x6268  36
NSSA     20.20.2.0        172.16.1.1       0x80000001   426  0x20 0x5772  36
NSSA     20.20.3.0        172.16.1.1       0x80000001   426  0x20 0x4c7c  36
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern   20.20.0.0        172.16.1.1       0x80000003   631  0x22 0x6762  36
Extern   20.20.1.0        172.16.1.1       0x80000002  1841  0x22 0x5e6b  36
Extern   20.20.2.0        172.16.1.1       0x80000002  1670  0x22 0x5375  36
Extern   20.20.3.0        172.16.1.1       0x80000002  1498  0x22 0x487f  36
Extern   20.20.4.0        172.16.2.1       0x80000002  2061  0x22 0x368f  36
Extern   20.20.5.0        172.16.2.1       0x80000002  1888  0x22 0x2b99  36
Extern   20.20.6.0        172.16.2.1       0x80000002  1715  0x22 0x20a3  36
Extern   20.20.7.0        172.16.2.1       0x80000002  1542  0x22 0x15ad  36

lab@mxD:R3-1>
```
</details><br />

### Configuration

`set protocols ospf area 10 nssa no-summaries`

<details markdown=1>
<summary markdown="span">outputs</summary>

```
Configuration

lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# set protocols ospf area 10 nssa no-summaries 

[edit]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA>
```
</details><br />

### After

<details markdown=1>
<summary markdown="span">outputs</summary>

```
lab@mxD:R3-1> show ospf database 

    OSPF database, Area 0.0.0.10
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   172.16.1.1       172.16.1.1       0x80000005    68  0x20 0xbdd9  36
Router  *172.16.1.2       172.16.1.2       0x80000005    67  0x20 0x7552  48
Network *10.0.1.2         172.16.1.2       0x80000003    67  0x20 0x10de  32
NSSA     20.20.0.0        172.16.1.1       0x80000001   636  0x20 0x6d5e  36
NSSA     20.20.1.0        172.16.1.1       0x80000001   636  0x20 0x6268  36
NSSA     20.20.2.0        172.16.1.1       0x80000001   636  0x20 0x5772  36
NSSA     20.20.3.0        172.16.1.1       0x80000001   636  0x20 0x4c7c  36
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern   20.20.0.0        172.16.1.1       0x80000003   841  0x22 0x6762  36
Extern   20.20.1.0        172.16.1.1       0x80000002  2051  0x22 0x5e6b  36
Extern   20.20.2.0        172.16.1.1       0x80000002  1880  0x22 0x5375  36
Extern   20.20.3.0        172.16.1.1       0x80000002  1708  0x22 0x487f  36
Extern   20.20.4.0        172.16.2.1       0x80000002  2271  0x22 0x368f  36
Extern   20.20.5.0        172.16.2.1       0x80000002  2098  0x22 0x2b99  36
Extern   20.20.6.0        172.16.2.1       0x80000002  1925  0x22 0x20a3  36
Extern   20.20.7.0        172.16.2.1       0x80000002  1752  0x22 0x15ad  36

lab@mxD:R3-1> 
```
</details><br />







<details markdown=1>
<summary markdown="span">outputs</summary>

```

```
</details><br />