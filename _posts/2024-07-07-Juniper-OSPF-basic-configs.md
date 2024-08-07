---
title: Juniper Junos OS OSPF basic configurations
date: 2024-07-07 16:37:00 -0700
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

lab@mxA> 
```

Static Routes on mxA

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> show route protocol static table inet.0 

inet.0: 11 destinations, 11 routes (11 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

20.20.0.0/24       *[Static/5] 08:42:43
                       Reject
20.20.1.0/24       *[Static/5] 08:42:43
                       Reject
20.20.2.0/24       *[Static/5] 08:42:43
                       Reject
20.20.3.0/24       *[Static/5] 08:42:43
                       Reject

lab@mxA> 
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

lab@mxC>
```

Static Routes on mxC

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxC> show route protocol static table inet.0   

inet.0: 11 destinations, 11 routes (11 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

20.20.4.0/24       *[Static/5] 09:44:23
                       Reject
20.20.5.0/24       *[Static/5] 09:44:23
                       Reject
20.20.6.0/24       *[Static/5] 09:44:23
                       Reject
20.20.7.0/24       *[Static/5] 09:44:23
                       Reject

lab@mxC> 
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
set logical-systems R3-1 routing-options autonomous-system 65512
set logical-systems R3-2 interfaces ge-0/0/4 unit 0 family inet address 10.0.2.2/24
set logical-systems R3-2 interfaces lo0 unit 2 family inet address 172.16.2.2/32
set logical-systems R3-2 routing-options autonomous-system 65512
set interfaces fxp0 unit 0 family inet address 172.25.11.4/24

lab@mxD> 
```

## Configure OSPF

### mxA

```
configure
edit protocols ospf
set area 0 interface ge-0/0/0.0
set area 0 interface lo0.0
set area 10 interface ge-0/0/3.0
commit and-quit
```

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# 

[edit]
lab@mxA# edit protocols ospf 

[edit protocols ospf]
lab@mxA# 

[edit protocols ospf]
lab@mxA# set area 0 interface ge-0/0/0.0 

[edit protocols ospf]
lab@mxA# 

[edit protocols ospf]
lab@mxA# set area 0 interface lo0.0 

[edit protocols ospf]
lab@mxA# 

[edit protocols ospf]
lab@mxA# set area 10 interface ge-0/0/3.0 

[edit protocols ospf]
lab@mxA# 

[edit protocols ospf]
lab@mxA# commit and-quit 

commit complete
Exiting configuration mode

lab@mxA>
```
</details><br />

`show ospf interface brief`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> show ospf interface brief 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          BDR     0.0.0.0         172.31.100.1    172.16.1.1         1
lo0.0               DR      0.0.0.0         172.16.1.1      0.0.0.0            0
ge-0/0/3.0          BDR     0.0.0.10        172.16.1.2      172.16.1.1         1
```
</details><br />

`show ospf neighbor`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.121.2     ge-0/0/0.0             Full      172.31.100.1     128    32
10.0.1.2         ge-0/0/3.0             Full      172.16.1.2       128    39

lab@mxA>
```
</details><br />

### mxB

*OSPF was already configured

`show ospf interface brief`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxB> show ospf interface brief 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          DR      0.0.0.0         172.31.100.1    172.16.1.1         1
ge-0/0/1.0          DR      0.0.0.0         172.31.100.1    172.16.2.1         1
lo0.0               DR      0.0.0.0         172.31.100.1    0.0.0.0            0
```
</details><br />

`show ospf neighbor`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxB> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.121.1     ge-0/0/0.0             Full      172.16.1.1       128    38
172.22.122.1     ge-0/0/1.0             Full      172.16.2.1       128    39

lab@mxB>
```
</details><br />

### mxC

```
configure
edit protocols ospf
set area 0 interface ge-0/0/0.0
set area 0 interface lo0.0
set area 20 interface ge-0/0/3.0
commit and-quit
```

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxC> configure 
Entering configuration mode

[edit]
lab@mxC# 

[edit]
lab@mxC# edit protocols ospf 

[edit protocols ospf]
lab@mxC# 

[edit protocols ospf]
lab@mxC# set area 0 interface ge-0/0/0.0 

[edit protocols ospf]
lab@mxC# 

[edit protocols ospf]
lab@mxC# set area 0 interface lo0.0 

[edit protocols ospf]
lab@mxC# 

[edit protocols ospf]
lab@mxC# set area 20 interface ge-0/0/3.0 

[edit protocols ospf]
lab@mxC# 

[edit protocols ospf]
lab@mxC# commit and-quit 
commit complete
Exiting configuration mode

lab@mxC> 
```
</details><br />

`show ospf interface brief`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxC> show ospf interface brief 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          BDR     0.0.0.0         172.31.100.1    172.16.2.1         1
lo0.0               DR      0.0.0.0         172.16.2.1      0.0.0.0            0
ge-0/0/3.0          DR      0.0.0.20        172.16.2.1      172.16.2.2         1
```
</details><br />

`show ospf neighbor`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxC> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.122.2     ge-0/0/0.0             Full      172.31.100.1     128    31
10.0.2.2         ge-0/0/3.0             Full      172.16.2.2       128    35

lab@mxC> 
```
</details><br />

### mxD:R3-1

```
set cli logical-system R3-1
configure
edit protocols ospf
set area 10 interface ge-0/0/1.0
set area 10 interface lo0.1
commit and-quit
```

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxD> set cli logical-system R3-1 
Logical system: R3-1

lab@mxD:R3-1> configure 
Entering configuration mode

[edit]
lab@mxD:R3-1# edit protocols ospf 

[edit protocols ospf]
lab@mxD:R3-1# set area 10 interface ge-0/0/1.0 

[edit protocols ospf]
lab@mxD:R3-1# set area 10 interface lo0.1 

[edit protocols ospf]
lab@mxD:R3-1# show | compare 
[edit logical-systems R3-1 protocols ospf]
+ area 0.0.0.10 {
+     interface ge-0/0/1.0;
+     interface lo0.1;
+ }

[edit protocols ospf]
lab@mxD:R3-1# commit and-quit   
commit complete
Exiting configuration mode

lab@mxD:R3-1> 

```
</details><br />

`show ospf interface brief`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxD:R3-1> show ospf interface brief 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/1.0          DR      0.0.0.10        172.16.1.2      172.16.1.1         1
lo0.1               DR      0.0.0.10        172.16.1.2      0.0.0.0            0
```
</details><br />

`show ospf neighbor`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxD:R3-1> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
10.0.1.1         ge-0/0/1.0             Full      172.16.1.1       128    37

lab@mxD:R3-1> 
```
</details><br />

### mxD:R3-2

```
set cli logical-system R3-2
configure
edit protocols ospf
set area 20 interface ge-0/0/4.0
set area 20 interface lo0.2
commit and-quit
```

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxD:R3-1> set cli logical-system R3-2 
Logical system: R3-2

lab@mxD:R3-2> 

lab@mxD:R3-2> configure 
Entering configuration mode

[edit]
lab@mxD:R3-2# 

[edit]
lab@mxD:R3-2# edit protocols ospf 

[edit protocols ospf]
lab@mxD:R3-2# 

[edit protocols ospf]
lab@mxD:R3-2# set area 20 interface ge-0/0/4.0 

[edit protocols ospf]
lab@mxD:R3-2# 

[edit protocols ospf]
lab@mxD:R3-2# set area 20 interface lo0.2 

[edit protocols ospf]
lab@mxD:R3-2# 

[edit protocols ospf]
lab@mxD:R3-2# commit and-quit 

commit complete
Exiting configuration mode

lab@mxD:R3-2>
```
</details><br />

`show ospf interface brief`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxD:R3-2> show ospf interface brief 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/4.0          BDR     0.0.0.20        172.16.2.1      172.16.2.2         1
lo0.2               DR      0.0.0.20        172.16.2.2      0.0.0.0            0

lab@mxD:R3-2> 
```
</details><br />

`show ospf neighbor`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxD:R3-2> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
10.0.2.1         ge-0/0/4.0             Full      172.16.2.1       128    34

lab@mxD:R3-2> 
```
</details><br />

## Redistribute Static routes into OSPF

There are static routes configured on **mxA** & **mxC** Routers

`show configuration routing-options static`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> show configuration routing-options static 
route 20.20.0.0/24 reject;
route 20.20.1.0/24 reject;
route 20.20.2.0/24 reject;
route 20.20.3.0/24 reject;

lab@mxA> 

lab@mxC> show configuration routing-options static 
route 20.20.4.0/24 reject;
route 20.20.5.0/24 reject;
route 20.20.6.0/24 reject;
route 20.20.7.0/24 reject;

lab@mxC> 
```
</details><br />

Command to redistribute static routes, where `static-to-ospf` is a `policy-statment` already configured.

On mxA# Router `set protocols ospf export static-to-ospf`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> show configuration policy-options 
policy-statement static-to-ospf {
    term 1 {
        from protocol static;
        then accept;
    }
}

configure
set protocols ospf export static-to-ospf
commit and-quit

lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# 

[edit]
lab@mxA# set protocols ospf export static-to-ospf 

[edit]
lab@mxA# 

[edit]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode
```
</details><br />

Verification in the OSPF LSDB, we see LSAs type 5 external.

```
lab@mxA> show ospf database external    
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern  *20.20.0.0        172.16.1.1       0x80000001    63  0x22 0x6b60  36
Extern  *20.20.1.0        172.16.1.1       0x80000001    63  0x22 0x606a  36
Extern  *20.20.2.0        172.16.1.1       0x80000001    63  0x22 0x5574  36
Extern  *20.20.3.0        172.16.1.1       0x80000001    63  0x22 0x4a7e  36
Extern   20.20.4.0        172.16.2.1       0x80000001   156  0x22 0x388e  36
Extern   20.20.5.0        172.16.2.1       0x80000001   156  0x22 0x2d98  36
Extern   20.20.6.0        172.16.2.1       0x80000001   156  0x22 0x22a2  36
Extern   20.20.7.0        172.16.2.1       0x80000001   156  0x22 0x17ac  36

lab@mxA> 
```

On mxC# Router `set protocols ospf export static-to-ospf`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxC> show configuration policy-options 
policy-statement static-to-ospf {
    term 1 {
        from protocol static;
        then accept;
    }
}

lab@mxC> 


configure
set protocols ospf export static-to-ospf
commit and-quit


lab@mxC> configure 
Entering configuration mode

[edit]
lab@mxC# edit protocols ospf 

[edit protocols ospf]
lab@mxC# show 
area 0.0.0.0 {
    interface ge-0/0/0.0;
    interface lo0.0;
}
area 0.0.0.20 {
    interface ge-0/0/3.0;
}

[edit protocols ospf]
lab@mxC# set export static-to-ospf  

[edit protocols ospf]
lab@mxC# show | compare 
[edit protocols ospf]
+ export static-to-ospf;

[edit protocols ospf]
lab@mxC# commit and-quit 
commit complete
Exiting configuration mode

lab@mxC> 
```
</details><br />

Verification in the OSPF LSDB, we see LSAs type 5 external.

```
lab@mxC> show ospf database external 
    OSPF AS SCOPE link state database
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Extern   20.20.0.0        172.16.1.1       0x80000001    82  0x22 0x6b60  36
Extern   20.20.1.0        172.16.1.1       0x80000001    82  0x22 0x606a  36
Extern   20.20.2.0        172.16.1.1       0x80000001    82  0x22 0x5574  36
Extern   20.20.3.0        172.16.1.1       0x80000001    82  0x22 0x4a7e  36
Extern  *20.20.4.0        172.16.2.1       0x80000001   172  0x22 0x388e  36
Extern  *20.20.5.0        172.16.2.1       0x80000001   172  0x22 0x2d98  36
Extern  *20.20.6.0        172.16.2.1       0x80000001   172  0x22 0x22a2  36
Extern  *20.20.7.0        172.16.2.1       0x80000001   172  0x22 0x17ac  36

lab@mxC> 
```

## Configure OSPF Cost on interfaces

On mxA# `show ospf interface detail`, notice `Cost: 1`

```
lab@mxA> show ospf interface detail 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          BDR     0.0.0.0         172.31.100.1    172.16.1.1         1
...
  Topology default (ID 0) -> Cost: 1
...
ge-0/0/3.0          BDR     0.0.0.10        172.16.1.2      172.16.1.1         1
...
  Topology default (ID 0) -> Cost: 1
```

<details markdown=1>
<summary markdown="span">Complete Output</summary>

```
lab@mxA> show ospf interface detail 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          BDR     0.0.0.0         172.31.100.1    172.16.1.1         1
  Type: LAN, Address: 172.22.121.1, Mask: 255.255.255.0, MTU: 1500, Cost: 1
  DR addr: 172.22.121.2, BDR addr: 172.22.121.1, Priority: 128
  Adj count: 1
  Hello: 10, Dead: 40, ReXmit: 5, Not Stub
  Auth type: None
  Protection type: None
  Topology default (ID 0) -> Cost: 1
lo0.0               DR      0.0.0.0         172.16.1.1      0.0.0.0            0
  Type: LAN, Address: 172.16.1.1, Mask: 255.255.255.255, MTU: 65535, Cost: 0
  DR addr: 172.16.1.1, Priority: 128
  Adj count: 0
  Hello: 10, Dead: 40, ReXmit: 5, Not Stub
  Auth type: None
  Protection type: None
  Topology default (ID 0) -> Cost: 0
ge-0/0/3.0          BDR     0.0.0.10        172.16.1.2      172.16.1.1         1
  Type: LAN, Address: 10.0.1.1, Mask: 255.255.255.0, MTU: 1500, Cost: 1
  DR addr: 10.0.1.2, BDR addr: 10.0.1.1, Priority: 128
  Adj count: 1
  Hello: 10, Dead: 40, ReXmit: 5, Not Stub
  Auth type: None
  Protection type: None
  Topology default (ID 0) -> Cost: 1

lab@mxA> 
```
</details><br />

On mxA# we now configure `set protocols ospf reference-bandwidth 10g`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# set protocols ospf reference-bandwidth 10g 

[edit]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 

```
</details><br />

Result on mxA#, notice `Cost: 10`

```
lab@mxA> show ospf interface detail    
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          BDR     0.0.0.0         172.31.100.1    172.16.1.1         1
...
  Topology default (ID 0) -> Cost: 10
...
ge-0/0/3.0          BDR     0.0.0.10        172.16.1.2      172.16.1.1         1
...
  Topology default (ID 0) -> Cost: 10
```

<details markdown=1>
<summary markdown="span">Complete Output</summary>

```
lab@mxA> show ospf interface detail    
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/0.0          BDR     0.0.0.0         172.31.100.1    172.16.1.1         1
  Type: LAN, Address: 172.22.121.1, Mask: 255.255.255.0, MTU: 1500, Cost: 10
  DR addr: 172.22.121.2, BDR addr: 172.22.121.1, Priority: 128
  Adj count: 1
  Hello: 10, Dead: 40, ReXmit: 5, Not Stub
  Auth type: None
  Protection type: None
  Topology default (ID 0) -> Cost: 10
lo0.0               DR      0.0.0.0         172.16.1.1      0.0.0.0            0
  Type: LAN, Address: 172.16.1.1, Mask: 255.255.255.255, MTU: 65535, Cost: 0
  DR addr: 172.16.1.1, Priority: 128
  Adj count: 0
  Hello: 10, Dead: 40, ReXmit: 5, Not Stub
  Auth type: None
  Protection type: None
  Topology default (ID 0) -> Cost: 0
ge-0/0/3.0          BDR     0.0.0.10        172.16.1.2      172.16.1.1         1
  Type: LAN, Address: 10.0.1.1, Mask: 255.255.255.0, MTU: 1500, Cost: 10
  DR addr: 10.0.1.2, BDR addr: 10.0.1.1, Priority: 128
  Adj count: 1
  Hello: 10, Dead: 40, ReXmit: 5, Not Stub
  Auth type: None
  Protection type: None
  Topology default (ID 0) -> Cost: 10

lab@mxA> 
```
</details><br />


Still on mxA# Router `set protocols ospf area 0 interface lo0.0 metric 10000`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# set protocols ospf area 0 interface lo0.0 metric 10000 

[edit]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 
```
</details><br />


On mxD:R3-1# the 172.16.1.1/32 advertised by mxA# has now `metric 10001`

```
lab@mxD:R3-1> show route 172.16/16    

inet.0: 19 destinations, 19 routes (19 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.1/32      *[OSPF/10] 00:00:40, metric 10001
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.1.2/32      *[Direct/0] 10:05:46
                    >  via lo0.1
172.16.2.1/32      *[OSPF/10] 00:02:13, metric 12
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.2.2/32      *[OSPF/10] 00:02:13, metric 13
                    >  to 10.0.1.1 via ge-0/0/1.0

lab@mxD:R3-1> 
```

On mxA# we configure a higher cost on the physical interface

`set protocols ospf area 10 interface ge-0/0/3.0 metric 5000`


<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# set protocols ospf area 10 interface ge-0/0/3.0 metric 5000 

[edit]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 
```
</details><br />

On mxD:R3-1# we configure a higher cost on the physical interface

`set protocols ospf area 10 interface ge-0/0/1.0 metric 2500`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxD:R3-1> configure 
Entering configuration mode

[edit]
lab@mxD:R3-1# edit protocols ospf 

[edit protocols ospf]
lab@mxD:R3-1# set area 10 interface ge-0/0/1.0 metric 2500 

[edit protocols ospf]
lab@mxD:R3-1# commit and-quit 
commit complete
Exiting configuration mode

lab@mxD:R3-1>
```
</details><br />

The results are seen both on mxA# and mxD:R3-1# Routing Table.

Metric of `5000` on mxD:R3-1# loopback received on mxA#

```
172.16.1.2/32      *[OSPF/10] 00:00:42, metric 5000
```

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> show route 172.16/16 

inet.0: 22 destinations, 22 routes (22 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.1/32      *[Direct/0] 10:15:24
                    >  via lo0.0
172.16.1.2/32      *[OSPF/10] 00:00:42, metric 5000
                    >  to 10.0.1.2 via ge-0/0/3.0
172.16.2.1/32      *[OSPF/10] 00:05:08, metric 11
                    >  to 172.22.121.2 via ge-0/0/0.0
172.16.2.2/32      *[OSPF/10] 00:05:08, metric 12
                    >  to 172.22.121.2 via ge-0/0/0.0

lab@mxA>

lab@mxA> show ospf interface detail 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/3.0          BDR     0.0.0.10        172.16.1.2      172.16.1.1         1
...
  Topology default (ID 0) -> Cost: 5000

```
</details><br />

Metric of `12500` (`10000` + `2500`) on mxD:R3-1# loopback received on mxA#


```
172.16.1.1/32      *[OSPF/10] 00:03:00, metric 12500
```

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxD:R3-1> show route 172.16/16    

inet.0: 19 destinations, 19 routes (19 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.1/32      *[OSPF/10] 00:03:00, metric 12500
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.1.2/32      *[Direct/0] 10:10:11
                    >  via lo0.1
172.16.2.1/32      *[OSPF/10] 00:03:00, metric 2511
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.2.2/32      *[OSPF/10] 00:03:00, metric 2512
                    >  to 10.0.1.1 via ge-0/0/1.0

lab@mxD:R3-1> 


lab@mxA> show ospf interface detail 
Interface           State   Area            DR ID           BDR ID          Nbrs
lo0.0               DR      0.0.0.0         172.16.1.1      0.0.0.0            0
...
  Topology default (ID 0) -> Cost: 10000


lab@mxD:R3-1> show ospf interface detail 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/1.0          DR      0.0.0.10        172.16.1.2      172.16.1.1         1
...
  Topology default (ID 0) -> Cost: 2500

```
</details><br />

## OSPF Overload

Before overload

show route protocol ospf 

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxD:R3-1> show route protocol ospf 

inet.0: 19 destinations, 19 routes (19 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.2.0/24        *[OSPF/10] 00:06:26, metric 2512
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.0.0/24       *[OSPF/150] 00:15:23, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.1.0/24       *[OSPF/150] 00:15:23, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.2.0/24       *[OSPF/150] 00:15:23, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.3.0/24       *[OSPF/150] 00:15:23, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.4.0/24       *[OSPF/150] 00:16:54, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.5.0/24       *[OSPF/150] 00:16:54, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.6.0/24       *[OSPF/150] 00:16:54, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.7.0/24       *[OSPF/150] 00:16:54, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.1.1/32      *[OSPF/10] 00:06:26, metric 12500
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.2.1/32      *[OSPF/10] 00:06:26, metric 2511
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.2.2/32      *[OSPF/10] 00:06:26, metric 2512
                    >  to 10.0.1.1 via ge-0/0/1.0
172.22.121.0/24    *[OSPF/10] 00:06:26, metric 2510
                    >  to 10.0.1.1 via ge-0/0/1.0
172.22.122.0/24    *[OSPF/10] 00:06:26, metric 2511
                    >  to 10.0.1.1 via ge-0/0/1.0
172.31.100.1/32    *[OSPF/10] 00:06:26, metric 2510
                    >  to 10.0.1.1 via ge-0/0/1.0
224.0.0.5/32       *[OSPF/10] 00:35:35, metric 1
                       MultiRecv

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

lab@mxD:R3-1> 
```
</details><br />

Configuring Overload

`set protocols ospf overload `

<details markdown=1>
<summary markdown="span">Output</summary>

```
set protocols ospf overload 


lab@mxD:R3-1> configure 
Entering configuration mode

[edit]
lab@mxD:R3-1# set protocols ospf overload 

[edit]
lab@mxD:R3-1# commit and-quit 
commit complete
Exiting configuration mode

lab@mxD:R3-1>
```
</details><br />

After overload

show route protocol ospf 

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxD:R3-1> show route protocol ospf    

inet.0: 19 destinations, 19 routes (19 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.2.0/24        *[OSPF/10] 00:00:01, metric 65547
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.0.0/24       *[OSPF/150] 00:16:10, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.1.0/24       *[OSPF/150] 00:16:10, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.2.0/24       *[OSPF/150] 00:16:10, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.3.0/24       *[OSPF/150] 00:16:10, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.4.0/24       *[OSPF/150] 00:17:41, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.5.0/24       *[OSPF/150] 00:17:41, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.6.0/24       *[OSPF/150] 00:17:41, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
20.20.7.0/24       *[OSPF/150] 00:17:41, metric 0, tag 0
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.1.1/32      *[OSPF/10] 00:00:01, metric 75535
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.2.1/32      *[OSPF/10] 00:00:01, metric 65546
                    >  to 10.0.1.1 via ge-0/0/1.0
172.16.2.2/32      *[OSPF/10] 00:00:01, metric 65547
                    >  to 10.0.1.1 via ge-0/0/1.0
172.22.121.0/24    *[OSPF/10] 00:00:01, metric 65545
                    >  to 10.0.1.1 via ge-0/0/1.0
172.22.122.0/24    *[OSPF/10] 00:00:01, metric 65546
                    >  to 10.0.1.1 via ge-0/0/1.0
172.31.100.1/32    *[OSPF/10] 00:00:01, metric 65545
                    >  to 10.0.1.1 via ge-0/0/1.0
224.0.0.5/32       *[OSPF/10] 00:36:22, metric 1
                       MultiRecv

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

lab@mxD:R3-1> 
```
</details><br />

## OSPF authentication

First enable traceoptions to debug the mismatch on the key that we will add temporarily

```
configure 
edit protocols ospf 
set traceoptions file ospf.log 
set traceoptions flag all detail 
commit and-quit 
```

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# edit protocols ospf 

[edit protocols ospf]
lab@mxA# set traceoptions file ospf.log 

[edit protocols ospf]
lab@mxA# set traceoptions flag all detail 

[edit protocols ospf]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 
```
</details><br />


Then on mxA# `set protocols ospf area 10 interface ge-0/0/3.0 authentication md5 10 key juniper`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# edit protocols ospf 

[edit protocols ospf]
lab@mxA# set area 10 interface ge-0/0/3.0 authentication md5 10 key juniper 

[edit protocols ospf]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 
```
</details><br />

We see the Dead timer slowly reaching zero.

```
lab@mxA> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.121.2     ge-0/0/0.0             Full      172.31.100.1     128    32
10.0.1.2         ge-0/0/3.0             Full      172.16.1.2       128    17

lab@mxA> show ospf neighbor    
Address          Interface              State     ID               Pri  Dead
172.22.121.2     ge-0/0/0.0             Full      172.31.100.1     128    34
10.0.1.2         ge-0/0/3.0             Full      172.16.1.2       128    10

lab@mxA> show ospf neighbor    
Address          Interface              State     ID               Pri  Dead
172.22.121.2     ge-0/0/0.0             Full      172.31.100.1     128    30
10.0.1.2         ge-0/0/3.0             Full      172.16.1.2       128     5

lab@mxA> show ospf neighbor    
Address          Interface              State     ID               Pri  Dead
172.22.121.2     ge-0/0/0.0             Full      172.31.100.1     128    35
10.0.1.2         ge-0/0/3.0             Full      172.16.1.2       128     1

lab@mxA> show ospf neighbor    
Address          Interface              State     ID               Pri  Dead
172.22.121.2     ge-0/0/0.0             Full      172.31.100.1     128    31

lab@mxA> 
```

Traceoutputs

```
lab@mxA> show log ospf.log | match auth 
Jul  7 23:22:31.151224   checksum 0x0, authtype 0
Jul  7 23:22:32.153249   checksum 0x0, authtype 0
Jul  7 23:22:32.153407   checksum 0x0, authtype 0
Jul  7 23:22:33.193100   checksum 0x0, authtype 0
Jul  7 23:22:34.160226   checksum 0x0, authtype 0
Jul  7 23:22:34.160378   checksum 0x0, authtype 0
Jul  7 23:22:36.862264 OSPF packet ignored: authentication type mismatch (0) from 10.0.1.2
Jul  7 23:22:36.864519 OSPF packet ignored: authentication type mismatch (0) from 10.0.1.2
Jul  7 23:22:44.952395 OSPF packet ignored: authentication type mismatch (0) from 10.0.1.2
Jul  7 23:22:44.952975 OSPF packet ignored: authentication type mismatch (0) from 10.0.1.2
Jul  7 23:22:52.563552 OSPF packet ignored: authentication type mismatch (0) from 10.0.1.2
Jul  7 23:22:52.563901 OSPF packet ignored: authentication type mismatch (0) from 10.0.1.2
Jul  7 23:23:01.090564 OSPF packet ignored: authentication type mismatch (0) from 10.0.1.2
Jul  7 23:23:01.091294 OSPF packet ignored: authentication type mismatch (0) from 10.0.1.2

lab@mxA> 
```

Now match the authentication on the other end.

On mxD:R3-1#, `set protocols ospf area 10 interface ge-0/0/1.0 authentication md5 10 key juniper`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxD:R3-1> configure   
Entering configuration mode

[edit]
lab@mxD:R3-1# edit protocols ospf 

[edit protocols ospf]
lab@mxD:R3-1# set area 10 interface ge-0/0/1.0 authentication md5 10 key juniper 

[edit protocols ospf]
lab@mxD:R3-1# commit and-quit 
commit complete
Exiting configuration mode

lab@mxD:R3-1> 
```
</details><br />

The OSPF adjacency recovers

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
172.22.121.2     ge-0/0/0.0             Full      172.31.100.1     128    33
10.0.1.2         ge-0/0/3.0             Full      172.16.1.2       128    35

lab@mxA> 


lab@mxD:R3-1> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
10.0.1.1         ge-0/0/1.0             Full      172.16.1.1       128    31

lab@mxD:R3-1> 
```
</details><br />


## Interface type P2P

By default the OSPF interface shows `Type: LAN`

```
lab@mxA> show ospf interface detail 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/3.0          BDR     0.0.0.10        172.16.1.2      172.16.1.1         1
  Type: LAN, Address: 10.0.1.1, Mask: 255.255.255.0, MTU: 1500, Cost: 5000
...
```

We now configure interface type P2P on mxA# using the command `interface-type p2p`

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA#  set protocols ospf area 10 interface ge-0/0/3.0 interface-type ?
Possible completions:
  nbma                 Nonbroadcast multiaccess
  p2mp                 Point-to-multipoint NBMA
  p2p                  Point-to-point
[edit]
lab@mxA# set protocols ospf area 10 interface ge-0/0/3.0 interface-type p2p 

[edit]
lab@mxA# show | compare                                                        
[edit protocols ospf area 0.0.0.10 interface ge-0/0/3.0]
+     interface-type p2p;

[edit]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 
```
</details><br />

We also configure interface type P2P on mxD:R3-1 using the command `interface-type p2p`


<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxD:R3-1> configure 
Entering configuration mode

[edit]
lab@mxD:R3-1# set protocols ospf area 10 interface ge-0/0/1.0 interface-type p2p 

[edit]
lab@mxD:R3-1# commit and-quit 
commit complete
Exiting configuration mode

lab@mxD:R3-1> 
```
</details><br />



The result is `Type: P2P`

```
lab@mxA> show ospf interface detail    
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/3.0          PtToPt  0.0.0.10        0.0.0.0         0.0.0.0            1
  Type: P2P, Address: 10.0.1.1, Mask: 255.255.255.0, MTU: 1500, Cost: 5000
...
```

<details markdown=1>
<summary markdown="span">Output</summary>

```
lab@mxA> show ospf interface detail    
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/3.0          PtToPt  0.0.0.10        0.0.0.0         0.0.0.0            1
  Type: P2P, Address: 10.0.1.1, Mask: 255.255.255.0, MTU: 1500, Cost: 5000



lab@mxD:R3-1> show ospf interface detail 
Interface           State   Area            DR ID           BDR ID          Nbrs
ge-0/0/1.0          PtToPt  0.0.0.10        0.0.0.0         0.0.0.0            1
  Type: P2P, Address: 10.0.1.2, Mask: 255.255.255.0, MTU: 1500, Cost: 2500
...
```
</details><br />
