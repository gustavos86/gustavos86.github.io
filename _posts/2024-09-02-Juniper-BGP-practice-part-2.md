---
title: Juniper Junos OS BGP basic configurations Part 2
date: 2024-09-02 21:45:00 -0700
categories: [JUNIPER, BGP]
tags: [juniper, bgp]     # TAG names should always be lowercase
---

This post is documents BGP configuration on Juniper Junos OS

This post documents BGP configuration on Juniper Junos OS

## Network Topology

<details markdown=1>
<summary markdown="span">Display Network Topology</summary>

![]({{ site.baseurl }}/images/2024/09-02-Juniper-BGP-practice-part-2/01-Network-topology.png)

</details><br />

## Base configurations

<details markdown=1>
<summary markdown="span">mxA</summary>

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
set interfaces ge-0/0/0 unit 0 family inet address 10.0.10.1/24
set interfaces ge-0/0/1 unit 0 family inet address 172.22.123.1/24
set interfaces ge-0/0/3 unit 0 family inet address 172.22.121.1/24
set interfaces fxp0 unit 0 family inet address 172.25.11.1/24
set interfaces lo0 unit 0 family inet address 172.16.1.1/32
set policy-options policy-statement export-aggregate term 1 from protocol aggregate
set policy-options policy-statement export-aggregate term 1 from route-filter 172.16.1.0/24 exact
set policy-options policy-statement export-aggregate term 1 then accept
set policy-options policy-statement export-aggregate term 2 from route-filter 172.16.1.0/24 longer
set policy-options policy-statement export-aggregate term 2 then reject
set policy-options policy-statement pfe-load-balance term 1 from protocol bgp
set policy-options policy-statement pfe-load-balance term 1 from route-filter 30.30.0.0/22 longer
set policy-options policy-statement pfe-load-balance term 1 then load-balance per-packet
set policy-options policy-statement redistribute-statics term 1 from protocol static
set policy-options policy-statement redistribute-statics term 1 then accept
set routing-options forwarding-table export pfe-load-balance
set routing-options autonomous-system 65001
set routing-options aggregate route 172.16.1.0/24
set protocols ospf area 0.0.0.0 interface lo0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set protocols bgp group ibgp type internal
set protocols bgp group ibgp local-address 172.16.1.1
set protocols bgp group ibgp export redistribute-statics
set protocols bgp group ibgp neighbor 172.16.1.2
set protocols bgp group P1-P2 type external
set protocols bgp group P1-P2 export redistribute-statics
set protocols bgp group P1-P2 export export-aggregate
set protocols bgp group P1-P2 peer-as 65412
set protocols bgp group P1-P2 neighbor 172.22.121.2
set protocols bgp group P1-P2 neighbor 172.22.123.2

lab@mxA>

lab@mxA> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
10.0.10.2        ge-0/0/0.0             Full      172.16.1.2       128    33

lab@mxA> 

lab@mxA> show bgp summary 
Threading mode: BGP I/O
Groups: 2 Peers: 3 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                      15          5          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.16.1.2            65001          7          6       0       0        1:33 Establ
  inet.0: 0/5/5/0
172.22.121.2          65412          9          5       0       0        1:39 Establ
  inet.0: 5/5/5/0
172.22.123.2          65412          7          5       0       0        1:35 Establ
  inet.0: 0/5/5/0

lab@mxA>
```
</details><br />

<details markdown=1>
<summary markdown="span">mxB</summary>

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
set logical-systems P3 interfaces ge-0/0/3 unit 0 family inet address 172.22.125.2/24
set logical-systems P3 interfaces ge-0/0/4 unit 0 family inet address 172.22.126.2/24
set logical-systems P3 interfaces lo0 unit 0 family inet address 172.31.102.1/32
set logical-systems P3 protocols bgp group R3-1 type external
set logical-systems P3 protocols bgp group R3-1 multihop
set logical-systems P3 protocols bgp group R3-1 local-address 172.31.102.1
set logical-systems P3 protocols bgp group R3-1 export ajspr-bgp-export-p3
set logical-systems P3 protocols bgp group R3-1 neighbor 172.16.1.2 peer-as 65001
set logical-systems P3 protocols bgp group R3-2 type external
set logical-systems P3 protocols bgp group R3-2 multihop
set logical-systems P3 protocols bgp group R3-2 local-address 172.31.102.1
set logical-systems P3 protocols bgp group R3-2 export ajspr-bgp-export-p3
set logical-systems P3 protocols bgp group R3-2 neighbor 172.16.2.2 peer-as 65002
set logical-systems P3 policy-options policy-statement ajspr-bgp-export-p3 term 1 from protocol static
set logical-systems P3 policy-options policy-statement ajspr-bgp-export-p3 term 1 from route-filter 40.40.0.0/22 orlonger
set logical-systems P3 policy-options policy-statement ajspr-bgp-export-p3 term 1 then accept
set logical-systems P3 policy-options policy-statement ajspr-bgp-export-p3 term 2 from protocol bgp
set logical-systems P3 policy-options policy-statement ajspr-bgp-export-p3 term 2 from route-filter 30.30.0.0/22 longer
set logical-systems P3 policy-options policy-statement ajspr-bgp-export-p3 term 2 then reject
set logical-systems P3 routing-options static route 20.20.0.0/24 reject
set logical-systems P3 routing-options static route 20.20.1.0/24 reject
set logical-systems P3 routing-options static route 20.20.2.0/24 reject
set logical-systems P3 routing-options static route 20.20.3.0/24 reject
set logical-systems P3 routing-options static route 20.20.4.0/25 reject
set logical-systems P3 routing-options static route 20.20.4.128/25 reject
set logical-systems P3 routing-options static route 20.20.5.0/26 reject
set logical-systems P3 routing-options static route 20.20.5.64/26 reject
set logical-systems P3 routing-options static route 20.20.5.128/26 reject
set logical-systems P3 routing-options static route 20.20.5.192/26 reject
set logical-systems P3 routing-options static route 40.40.0.0/24 reject
set logical-systems P3 routing-options static route 40.40.1.0/24 reject
set logical-systems P3 routing-options static route 40.40.2.0/24 reject
set logical-systems P3 routing-options static route 40.40.3.0/24 reject
set logical-systems P3 routing-options static route 172.16.1.2/32 next-hop 172.22.125.1
set logical-systems P3 routing-options static route 172.16.2.2/32 next-hop 172.22.126.1
set logical-systems P3 routing-options static route 172.16.1.4/32 next-hop 172.22.125.1
set logical-systems P3 routing-options static route 172.16.2.4/32 next-hop 172.22.126.1
set logical-systems P3 routing-options static route 10.0.20.0/22 next-hop 172.22.125.1
set logical-systems P3 routing-options static route 10.0.24.0/22 next-hop 172.22.126.1
set logical-systems P3 routing-options autonomous-system 65020
set logical-systems P3 routing-options aggregate route 20.20.0.0/21
set logical-systems R3-1 interfaces ge-0/0/0 unit 0 family inet address 10.0.10.2/24
set logical-systems R3-1 interfaces ge-0/0/2 unit 0 family inet address 172.22.125.1/24
set logical-systems R3-1 interfaces lo0 unit 1 family inet address 172.16.1.2/32
set logical-systems R3-1 protocols ospf area 0.0.0.0 interface lo0.1
set logical-systems R3-1 protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set logical-systems R3-1 protocols bgp group ibgp type internal
set logical-systems R3-1 protocols bgp group ibgp local-address 172.16.1.2
set logical-systems R3-1 protocols bgp group ibgp export redistribute-statics
set logical-systems R3-1 protocols bgp group ibgp neighbor 172.16.1.1
set logical-systems R3-1 protocols bgp group P3 type external
set logical-systems R3-1 protocols bgp group P3 multihop
set logical-systems R3-1 protocols bgp group P3 local-address 172.16.1.2
set logical-systems R3-1 protocols bgp group P3 export export-aggregate
set logical-systems R3-1 protocols bgp group P3 peer-as 65020
set logical-systems R3-1 protocols bgp group P3 neighbor 172.31.102.1
set logical-systems R3-1 policy-options policy-statement export-aggregate term 1 from protocol aggregate
set logical-systems R3-1 policy-options policy-statement export-aggregate term 1 from route-filter 172.16.1.0/24 exact
set logical-systems R3-1 policy-options policy-statement export-aggregate term 1 then accept
set logical-systems R3-1 policy-options policy-statement export-aggregate term 2 from route-filter 172.16.1.0/24 longer
set logical-systems R3-1 policy-options policy-statement export-aggregate term 2 then reject
set logical-systems R3-1 policy-options policy-statement redistribute-statics term 1 from protocol static
set logical-systems R3-1 policy-options policy-statement redistribute-statics term 1 then accept
set logical-systems R3-1 routing-options static route 172.31.102.1/32 next-hop 172.22.125.2
set logical-systems R3-1 routing-options static route 172.31.102.1/32 no-readvertise
set logical-systems R3-1 routing-options autonomous-system 65001
set logical-systems R3-1 routing-options aggregate route 172.16.1.0/24
set logical-systems R3-2 interfaces ge-0/0/1 unit 0 family inet address 10.0.14.2/24
set logical-systems R3-2 interfaces ge-0/0/5 unit 0 family inet address 172.22.126.1/24
set logical-systems R3-2 interfaces lo0 unit 2 family inet address 172.16.2.2/32
set logical-systems R3-2 protocols ospf area 0.0.0.0 interface lo0.2
set logical-systems R3-2 protocols ospf area 0.0.0.0 interface ge-0/0/1.0
set logical-systems R3-2 protocols bgp group ibgp type internal
set logical-systems R3-2 protocols bgp group ibgp local-address 172.16.2.2
set logical-systems R3-2 protocols bgp group ibgp export redistribute-statics
set logical-systems R3-2 protocols bgp group ibgp export next-hop-self
set logical-systems R3-2 protocols bgp group ibgp neighbor 172.16.2.1
set logical-systems R3-2 protocols bgp group P3 type external
set logical-systems R3-2 protocols bgp group P3 multihop
set logical-systems R3-2 protocols bgp group P3 local-address 172.16.2.2
set logical-systems R3-2 protocols bgp group P3 export export-aggregate
set logical-systems R3-2 protocols bgp group P3 neighbor 172.31.102.1 peer-as 65020
set logical-systems R3-2 policy-options policy-statement export-aggregate term 1 from protocol aggregate
set logical-systems R3-2 policy-options policy-statement export-aggregate term 1 from route-filter 172.16.2.0/24 exact
set logical-systems R3-2 policy-options policy-statement export-aggregate term 1 then accept
set logical-systems R3-2 policy-options policy-statement export-aggregate term 2 from route-filter 172.16.2.0/24 longer
set logical-systems R3-2 policy-options policy-statement export-aggregate term 2 then reject
set logical-systems R3-2 policy-options policy-statement next-hop-self term 1 from protocol bgp
set logical-systems R3-2 policy-options policy-statement next-hop-self term 1 from route-type external
set logical-systems R3-2 policy-options policy-statement next-hop-self term 1 then next-hop self
set logical-systems R3-2 policy-options policy-statement redistribute-statics term 1 from protocol static
set logical-systems R3-2 policy-options policy-statement redistribute-statics term 1 then accept
set logical-systems R3-2 routing-options static route 172.16.2.128/26 reject
set logical-systems R3-2 routing-options static route 172.16.2.192/26 reject
set logical-systems R3-2 routing-options static route 172.31.102.1/32 next-hop 172.22.126.2
set logical-systems R3-2 routing-options static route 172.31.102.1/32 no-readvertise
set logical-systems R3-2 routing-options autonomous-system 65002
set logical-systems R3-2 routing-options aggregate route 172.16.2.0/24
set interfaces fxp0 unit 0 family inet address 172.25.11.2/24
                                        
lab@mxB>

lab@mxB> show bgp summary logical-system P3 
Threading mode: BGP I/O
Groups: 2 Peers: 2 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       2          2          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.16.1.2            65001          3          4       0       0          26 Establ
  inet.0: 1/1/1/0
172.16.2.2            65002          3          4       0       0          24 Establ
  inet.0: 1/1/1/0

lab@mxB>
```
</details><br />

<details markdown=1>
<summary markdown="span">mxC</summary>

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
set interfaces ge-0/0/0 unit 0 family inet address 10.0.14.1/24
set interfaces ge-0/0/1 unit 0 family inet address 172.22.124.1/24
set interfaces ge-0/0/3 unit 0 family inet address 172.22.122.1/24
set interfaces fxp0 unit 0 family inet address 172.25.11.3/24
set interfaces lo0 unit 0 family inet address 172.16.2.1/32
set policy-options policy-statement export-aggregate term 1 from protocol aggregate
set policy-options policy-statement export-aggregate term 1 from route-filter 172.16.2.0/24 exact
set policy-options policy-statement export-aggregate term 1 then accept
set policy-options policy-statement export-aggregate term 2 from route-filter 172.16.2.0/24 longer
set policy-options policy-statement export-aggregate term 2 then reject
set policy-options policy-statement import-P1 term 1 from protocol bgp
set policy-options policy-statement import-P1 term 1 from as-path partner-as
set policy-options policy-statement import-P1 term 1 then accept
set policy-options policy-statement import-P1 term 2 then reject
set policy-options policy-statement next-hop-self term 1 from protocol bgp
set policy-options policy-statement next-hop-self term 1 from route-type external
set policy-options policy-statement next-hop-self term 1 then next-hop self
set policy-options policy-statement pfe-load-balance term 1 from protocol bgp
set policy-options policy-statement pfe-load-balance term 1 from route-filter 30.30.0.0/22 longer
set policy-options policy-statement pfe-load-balance term 1 then load-balance per-packet
set policy-options policy-statement redistribute-statics term 1 from protocol static
set policy-options policy-statement redistribute-statics term 1 then accept
set policy-options as-path partner-as ".* 65001"
set policy-options as-path internal-as "()"
set routing-options static route 172.16.2.0/26 reject
set routing-options static route 172.16.2.64/26 reject
set routing-options forwarding-table export pfe-load-balance
set routing-options autonomous-system 65002
set routing-options aggregate route 172.16.2.0/24
set protocols ospf area 0.0.0.0 interface lo0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set protocols bgp group ibgp type internal
set protocols bgp group ibgp local-address 172.16.2.1
set protocols bgp group ibgp export redistribute-statics
set protocols bgp group ibgp export next-hop-self
set protocols bgp group ibgp neighbor 172.16.2.2
set protocols bgp group P1-P2 type external
set protocols bgp group P1-P2 export export-aggregate
set protocols bgp group P1-P2 peer-as 65412
set protocols bgp group P1-P2 neighbor 172.22.122.2 import import-P1
set protocols bgp group P1-P2 neighbor 172.22.124.2

lab@mxC>

lab@mxC> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
10.0.14.2        ge-0/0/0.0             Full      172.16.2.2       128    33

lab@mxC> 

lab@mxC> show bgp summary 
Threading mode: BGP I/O
Groups: 2 Peers: 3 Down peers: 2
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       7          7          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.16.2.2            65002         51         49       0       0       21:06 Establ
  inet.0: 7/7/7/0
172.22.122.2          65412          0          0       0       0       21:18 Connect
172.22.124.2          65412          0          0       0       0       21:18 Connect

lab@mxC>
```
</details><br />

<details markdown=1>
<summary markdown="span">mxD</summary>

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
set logical-systems P1 interfaces ge-0/0/0 unit 0 family inet address 172.22.252.1/30
set logical-systems P1 interfaces ge-0/0/1 unit 0 family inet address 172.22.121.2/24
set logical-systems P1 interfaces ge-0/0/4 unit 0 family inet address 172.22.122.2/24
set logical-systems P1 interfaces lo0 unit 0 family inet address 172.31.100.1/32
set logical-systems P1 protocols ospf area 0.0.0.0 interface lo0.0
set logical-systems P1 protocols ospf area 0.0.0.0 interface ge-0/0/0.0
set logical-systems P1 protocols bgp group ibgp type internal
set logical-systems P1 protocols bgp group ibgp local-address 172.31.100.1
set logical-systems P1 protocols bgp group ibgp export ajspr-bgp-nhs
set logical-systems P1 protocols bgp group ibgp neighbor 172.31.101.1
set logical-systems P1 protocols bgp group mxA type external
set logical-systems P1 protocols bgp group mxA export ajspr-bgp-export
set logical-systems P1 protocols bgp group mxA neighbor 172.22.121.1 peer-as 65001
set logical-systems P1 protocols bgp group mxC type external
set logical-systems P1 protocols bgp group mxC export ajspr-bgp-export
set logical-systems P1 protocols bgp group mxC neighbor 172.22.122.1 peer-as 65002
set logical-systems P1 policy-options policy-statement ajspr-bgp-export term 1 from protocol static
set logical-systems P1 policy-options policy-statement ajspr-bgp-export term 1 from route-filter 30.30.0.0/22 orlonger
set logical-systems P1 policy-options policy-statement ajspr-bgp-export term 1 then accept
set logical-systems P1 policy-options policy-statement ajspr-bgp-export term 2 from protocol bgp
set logical-systems P1 policy-options policy-statement ajspr-bgp-export term 2 from route-filter 40.40.0.0/22 longer
set logical-systems P1 policy-options policy-statement ajspr-bgp-export term 2 then reject
set logical-systems P1 policy-options policy-statement ajspr-bgp-nhs term 1 from protocol bgp
set logical-systems P1 policy-options policy-statement ajspr-bgp-nhs term 1 from route-type external
set logical-systems P1 policy-options policy-statement ajspr-bgp-nhs term 1 then next-hop self
set logical-systems P1 routing-options static route 30.30.0.0/24 reject
set logical-systems P1 routing-options static route 30.30.1.0/24 reject
set logical-systems P1 routing-options static route 30.30.2.0/24 reject
set logical-systems P1 routing-options static route 30.30.3.0/24 reject
set logical-systems P1 routing-options autonomous-system 65412
set interfaces fxp0 unit 0 family inet address 172.25.11.4/24

lab@mxD> 

lab@mxD> show ospf neighbor logical-system P1 
Address          Interface              State     ID               Pri  Dead
172.22.252.2     ge-0/0/0.0             Full      172.31.101.1     128    36

lab@mxD> 

lab@mxD> show bgp summary logical-system P1 
Threading mode: BGP I/O
Groups: 3 Peers: 3 Down peers: 1
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                      12          6          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.22.121.1          65001          0          0       0       0        2:27 Connect
172.22.122.1          65002         10          7       0       0        2:15 Establ
  inet.0: 6/6/6/0
172.31.101.1          65412          9          8       0       0        2:05 Establ
  inet.0: 0/6/6/0

lab@mxD>
```
</details><br />

<details markdown=1>
<summary markdown="span">mxE</summary>

```
lab@mxE> show configuration | display set 
set version 20190829.221548_builder.r1052644
set system host-name mxE
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
set logical-systems P2 interfaces ge-0/0/2 unit 0 family inet address 172.22.124.2/24
set logical-systems P2 interfaces ge-0/0/5 unit 0 family inet address 172.22.252.2/30
set logical-systems P2 interfaces ge-0/0/7 unit 0 family inet address 172.22.123.2/24
set logical-systems P2 interfaces lo0 unit 0 family inet address 172.31.101.1/32
set logical-systems P2 protocols ospf area 0.0.0.0 interface lo0.0
set logical-systems P2 protocols ospf area 0.0.0.0 interface ge-0/0/5.0
set logical-systems P2 protocols bgp group ibgp type internal
set logical-systems P2 protocols bgp group ibgp local-address 172.31.101.1
set logical-systems P2 protocols bgp group ibgp export ajspr-bgp-nhs
set logical-systems P2 protocols bgp group ibgp neighbor 172.31.100.1
set logical-systems P2 protocols bgp group mxA type external
set logical-systems P2 protocols bgp group mxA export ajspr-bgp-export
set logical-systems P2 protocols bgp group mxA neighbor 172.22.123.1 peer-as 65001
set logical-systems P2 protocols bgp group mxC type external
set logical-systems P2 protocols bgp group mxC export ajspr-bgp-export
set logical-systems P2 protocols bgp group mxC neighbor 172.22.124.1 peer-as 65002
set logical-systems P2 policy-options policy-statement ajspr-bgp-export term 1 from protocol static
set logical-systems P2 policy-options policy-statement ajspr-bgp-export term 1 from route-filter 30.30.0.0/22 orlonger
set logical-systems P2 policy-options policy-statement ajspr-bgp-export term 1 then accept
set logical-systems P2 policy-options policy-statement ajspr-bgp-export term 2 from protocol bgp
set logical-systems P2 policy-options policy-statement ajspr-bgp-export term 2 from route-filter 40.40.0.0/22 longer
set logical-systems P2 policy-options policy-statement ajspr-bgp-export term 2 then reject
set logical-systems P2 policy-options policy-statement ajspr-bgp-nhs term 1 from protocol bgp
set logical-systems P2 policy-options policy-statement ajspr-bgp-nhs term 1 from route-type external
set logical-systems P2 policy-options policy-statement ajspr-bgp-nhs term 1 then next-hop self
set logical-systems P2 routing-options static route 30.30.0.0/24 reject
set logical-systems P2 routing-options static route 30.30.1.0/24 reject
set logical-systems P2 routing-options static route 30.30.2.0/24 reject
set logical-systems P2 routing-options static route 30.30.3.0/24 reject
set logical-systems P2 routing-options autonomous-system 65412
set interfaces fxp0 unit 0 family inet address 172.25.11.5/24

lab@mxE>

lab@mxE> show bgp summary logical-system P2 
Threading mode: BGP I/O
Groups: 3 Peers: 3 Down peers: 2
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       6          6          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.22.123.1          65001          0          0       0       0        1:14 Connect
172.22.124.1          65002          7          4       0       0        1:04 Establ
  inet.0: 6/6/6/0
172.31.100.1          65412          0          0       0       0        1:14 Active

lab@mxE> 
```
</details><br />

## Repairing Unusable Routes

<details markdown=1>
<summary markdown="span">mxA</summary>

```
lab@mxA> show route table inet.0 

inet.0: 21 destinations, 27 routes (17 active, 0 holddown, 5 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.10.0/24       *[Direct/0] 00:11:05
                    >  via ge-0/0/0.0
10.0.10.1/32       *[Local/0] 00:11:05
                       Local via ge-0/0/0.0
30.30.0.0/24       *[BGP/170] 00:10:59, localpref 100
                      AS path: 65412 I, validation-state: unverified
                    >  to 172.22.121.2 via ge-0/0/3.0
                    [BGP/170] 00:10:55, localpref 100
                      AS path: 65412 I, validation-state: unverified
                    >  to 172.22.123.2 via ge-0/0/1.0
30.30.1.0/24       *[BGP/170] 00:10:59, localpref 100
                      AS path: 65412 I, validation-state: unverified
                    >  to 172.22.121.2 via ge-0/0/3.0
                    [BGP/170] 00:10:55, localpref 100
                      AS path: 65412 I, validation-state: unverified
                    >  to 172.22.123.2 via ge-0/0/1.0
30.30.2.0/24       *[BGP/170] 00:10:59, localpref 100
                      AS path: 65412 I, validation-state: unverified
                    >  to 172.22.121.2 via ge-0/0/3.0
                    [BGP/170] 00:10:55, localpref 100
                      AS path: 65412 I, validation-state: unverified
                    >  to 172.22.123.2 via ge-0/0/1.0
30.30.3.0/24       *[BGP/170] 00:10:59, localpref 100
                      AS path: 65412 I, validation-state: unverified
                    >  to 172.22.121.2 via ge-0/0/3.0
                    [BGP/170] 00:10:55, localpref 100
                      AS path: 65412 I, validation-state: unverified
                    >  to 172.22.123.2 via ge-0/0/1.0
172.16.1.0/24      *[Aggregate/130] 00:11:05
                       Reject
172.16.1.1/32      *[Direct/0] 04:22:15
                    >  via lo0.0
172.16.1.2/32      *[OSPF/10] 00:10:55, metric 1
                    >  to 10.0.10.2 via ge-0/0/0.0
172.16.2.0/24      *[BGP/170] 00:10:59, localpref 100
                      AS path: 65412 65002 I, validation-state: unverified
                    >  to 172.22.121.2 via ge-0/0/3.0
                    [BGP/170] 00:10:55, localpref 100
                      AS path: 65412 65002 I, validation-state: unverified
                    >  to 172.22.123.2 via ge-0/0/1.0
172.22.121.0/24    *[Direct/0] 00:11:05
                    >  via ge-0/0/3.0
172.22.121.1/32    *[Local/0] 00:11:05
                       Local via ge-0/0/3.0
172.22.123.0/24    *[Direct/0] 00:11:05
                    >  via ge-0/0/1.0
172.22.123.1/32    *[Local/0] 00:11:05
                       Local via ge-0/0/1.0
172.25.11.0/24     *[Direct/0] 04:22:15
                    >  via fxp0.0
172.25.11.1/32     *[Local/0] 04:22:15
                       Local via fxp0.0
224.0.0.5/32       *[OSPF/10] 00:11:05, metric 1
                       MultiRecv

lab@mxA> 

lab@mxA> show route hidden table inet.0 

inet.0: 21 destinations, 27 routes (17 active, 0 holddown, 5 hidden)
+ = Active Route, - = Last Active, * = Both

40.40.0.0/24        [BGP/170] 00:12:45, localpref 100, from 172.16.1.2
                      AS path: 65020 I, validation-state: unverified
                       Unusable
40.40.1.0/24        [BGP/170] 00:12:45, localpref 100, from 172.16.1.2
                      AS path: 65020 I, validation-state: unverified
                       Unusable
40.40.2.0/24        [BGP/170] 00:12:45, localpref 100, from 172.16.1.2
                      AS path: 65020 I, validation-state: unverified
                       Unusable
40.40.3.0/24        [BGP/170] 00:12:45, localpref 100, from 172.16.1.2
                      AS path: 65020 I, validation-state: unverified
                       Unusable
172.16.2.0/24       [BGP/170] 00:12:45, localpref 100, from 172.16.1.2
                      AS path: 65020 65002 I, validation-state: unverified
                       Unusable

lab@mxA> 


lab@mxA> show route 40.40.0.0/24 hidden extensive 

inet.0: 21 destinations, 27 routes (17 active, 0 holddown, 5 hidden)
40.40.0.0/24 (1 entry, 0 announced)
         BGP    Preference: 170/-101
                Next hop type: Unusable, Next hop index: 0
                Address: 0xc0f12d0
                Next-hop reference count: 5
                State: <Hidden Int Ext Changed>
                Local AS: 65001 Peer AS: 65001
                Age: 13:14 
                Validation State: unverified 
                Task: BGP_65001.172.16.1.2
                AS path: 65020 I 
                Accepted
                Localpref: 100
                Router ID: 172.16.1.2
                Indirect next hops: 1
                        Protocol next hop: 172.31.102.1
                        Indirect next hop: 0x0 - INH Session ID: 0x0

lab@mxA> 

lab@mxA> show route 172.31.102.1 table inet.0 

lab@mxA> 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxB:R3-1</summary>

```
lab@mxB> set cli logical-system R3-1 
Logical system: R3-1

lab@mxB:R3-1> configure 
Entering configuration mode

[edit]
lab@mxB:R3-1# edit policy-options policy-statement next-hop-self 

[edit policy-options policy-statement next-hop-self]
lab@mxB:R3-1# set term 1 from protocol bgp 

[edit policy-options policy-statement next-hop-self]
lab@mxB:R3-1# set term 1 from route-type external 

[edit policy-options policy-statement next-hop-self]
lab@mxB:R3-1# set term 1 then next-hop self 

[edit policy-options policy-statement next-hop-self]
lab@mxB:R3-1# top 

[edit]
lab@mxB:R3-1# show | compare 
[edit logical-systems R3-1 policy-options]
+   policy-statement next-hop-self {
+       term 1 {
+           from {
+               protocol bgp;
+               route-type external;
+           }
+           then {
+               next-hop self;
+           }
+       }
+   }

[edit]
lab@mxB:R3-1# 

[edit]
lab@mxB:R3-1# top edit protocols bgp 

[edit protocols bgp]
lab@mxB:R3-1# set group ibgp export next-hop-self 

[edit protocols bgp]
lab@mxB:R3-1# top 

[edit]
lab@mxB:R3-1# show | compare 
[edit logical-systems R3-1 protocols bgp group ibgp]
-    export redistribute-statics;
+    export [ redistribute-statics next-hop-self ];
[edit logical-systems R3-1 policy-options]
+   policy-statement next-hop-self {
+       term 1 {
+           from {
+               protocol bgp;
+               route-type external;
+           }
+           then {
+               next-hop self;
+           }
+       }
+   }

[edit]
lab@mxB:R3-1# 

[edit]
lab@mxB:R3-1# commit and-quit 
commit complete
Exiting configuration mode

lab@mxB:R3-1> 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxA</summary>

```
lab@mxA> show route hidden 

inet.0: 21 destinations, 27 routes (21 active, 0 holddown, 0 hidden)

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

lab@mxA> 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxB:R3-1</summary>

```
lab@mxB:R3-1> show route hidden 

inet.0: 18 destinations, 19 routes (14 active, 0 holddown, 5 hidden)
+ = Active Route, - = Last Active, * = Both

30.30.0.0/24        [BGP/170] 00:35:18, localpref 100, from 172.16.1.1
                      AS path: 65412 I, validation-state: unverified
                       Unusable
30.30.1.0/24        [BGP/170] 00:35:18, localpref 100, from 172.16.1.1
                      AS path: 65412 I, validation-state: unverified
                       Unusable
30.30.2.0/24        [BGP/170] 00:35:18, localpref 100, from 172.16.1.1
                      AS path: 65412 I, validation-state: unverified
                       Unusable
30.30.3.0/24        [BGP/170] 00:35:18, localpref 100, from 172.16.1.1
                      AS path: 65412 I, validation-state: unverified
                       Unusable
172.16.2.0/24       [BGP/170] 00:35:18, localpref 100, from 172.16.1.1
                      AS path: 65412 65002 I, validation-state: unverified
                       Unusable

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

lab@mxB:R3-1>


lab@mxB:R3-1> show route 30.30.0.0/24 hidden extensive 

inet.0: 18 destinations, 19 routes (14 active, 0 holddown, 5 hidden)
30.30.0.0/24 (1 entry, 0 announced)
         BGP    Preference: 170/-101
                Next hop type: Unusable, Next hop index: 0
                Address: 0xc0d82d0
                Next-hop reference count: 5
                State: <Hidden Int Ext Changed>
                Local AS: 65001 Peer AS: 65001
                Age: 36:21 
                Validation State: unverified 
                Task: BGP_65001.172.16.1.1
                AS path: 65412 I 
                Accepted
                Localpref: 100
                Router ID: 172.16.1.1
                Indirect next hops: 1
                        Protocol next hop: 172.22.121.2
                        Indirect next hop: 0x0 - INH Session ID: 0x0

lab@mxB:R3-1> 

lab@mxB:R3-1> show route 172.22.121.2 table inet.0 

lab@mxB:R3-1> 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxA</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# edit policy-options policy-statement next-hop-self 

[edit policy-options policy-statement next-hop-self]
lab@mxA# set term 1 from protocol bgp 

[edit policy-options policy-statement next-hop-self]
lab@mxA# set term 1 from route-type external 

[edit policy-options policy-statement next-hop-self]
lab@mxA# set term 1 then next-hop self 

[edit policy-options policy-statement next-hop-self]
lab@mxA# top 

[edit]
lab@mxA# show | compare 
[edit policy-options]
+   policy-statement next-hop-self {
+       term 1 {
+           from {
+               protocol bgp;
+               route-type external;
+           }
+           then {
+               next-hop self;
+           }
+       }
+   }

[edit]
lab@mxA# 

[edit]
lab@mxA# top edit protocols bgp 

[edit protocols bgp]
lab@mxA# set group ibgp export next-hop-self 

[edit protocols bgp]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA>
```
</details><br />

<details markdown=1>
<summary markdown="span">mxB:R3-1</summary>

```
lab@mxB:R3-1> show route hidden 

inet.0: 18 destinations, 19 routes (18 active, 0 holddown, 0 hidden)

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

lab@mxB:R3-1> 
```
</details><br />

## Modifying the Local-Preference Attribute

<details markdown=1>
<summary markdown="span">mxA</summary>

```
lab@mxA> show route 172.16.2.0/24 detail 

inet.0: 21 destinations, 27 routes (21 active, 0 holddown, 0 hidden)
172.16.2.0/24 (3 entries, 1 announced)
        *BGP    Preference: 170/-101
                Next hop type: Router, Next hop index: 578
                Address: 0xc0f1bcc
                Next-hop reference count: 10
                Source: 172.22.121.2
                Next hop: 172.22.121.2 via ge-0/0/3.0, selected
                Session Id: 0x140
                State: <Active Ext>
                Local AS: 65001 Peer AS: 65412
                Age: 41:25 
                Validation State: unverified 
                Task: BGP_65412.172.22.121.2
                Announcement bits (3): 0-KRT 1-BGP_RT_Background 5-Resolve tree 1 
                AS path: 65412 65002 I 
                Aggregator: 65002 172.16.2.1
                Accepted
                Localpref: 100
                Router ID: 172.31.100.1
         BGP    Preference: 170/-101
                Next hop type: Router, Next hop index: 0
                Address: 0xc0f1d5c
                Next-hop reference count: 5
                Source: 172.22.123.2
                Next hop: 172.22.123.2 via ge-0/0/1.0, selected
                Session Id: 0x0
                State: <NotBest Ext Changed>
                Inactive reason: Not Best in its group - Active preferred
                Local AS: 65001 Peer AS: 65412
                Age: 41:21 
                Validation State: unverified 
                Task: BGP_65412.172.22.123.2
                AS path: 65412 65002 I 
                Aggregator: 65002 172.16.2.1
                Accepted
                Localpref: 100
                Router ID: 172.31.101.1
         BGP    Preference: 170/-101    
                Next hop type: Indirect, Next hop index: 0
                Address: 0xc0f1f50
                Next-hop reference count: 9
                Source: 172.16.1.2
                Next hop type: Router, Next hop index: 601
                Next hop: 10.0.10.2 via ge-0/0/0.0, selected
                Session Id: 0x141
                Protocol next hop: 172.16.1.2
                Indirect next hop: 0xc298a84 1048574 INH Session ID: 0x142
                State: <Int Ext Changed>
                Inactive reason: Interior > Exterior > Exterior via Interior
                Local AS: 65001 Peer AS: 65001
                Age: 7:10 	Metric2: 1 
                Validation State: unverified 
                Task: BGP_65001.172.16.1.2
                AS path: 65020 65002 I 
                Aggregator: 65002 172.16.2.2
                Accepted
                Localpref: 100
                Router ID: 172.16.1.2

lab@mxA> 

lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# edit policy-options policy-statement import-localpref 

[edit policy-options policy-statement import-localpref]
lab@mxA# set term 1 from protocol bgp 

[edit policy-options policy-statement import-localpref]
lab@mxA# set term 1 from neighbor 172.22.121.2 

[edit policy-options policy-statement import-localpref]
lab@mxA# set term 1 from route-filter 172.16.2.0/24 exact 

[edit policy-options policy-statement import-localpref]
lab@mxA# set term 1 then local-preference 110 

[edit policy-options policy-statement import-localpref]
lab@mxA# show 
term 1 {
    from {
        protocol bgp;
        neighbor 172.22.121.2;
        route-filter 172.16.2.0/24 exact;
    }
    then {
        local-preference 110;
    }
}

[edit policy-options policy-statement import-localpref]
lab@mxA# 

[edit policy-options policy-statement import-localpref]
lab@mxA# top 

[edit]
lab@mxA# 

[edit]
lab@mxA# 

[edit]
lab@mxA# top edit protocols bgp group P1-P2 

[edit protocols bgp group P1-P2]
lab@mxA# set import import-local-pref 

[edit protocols bgp group P1-P2]
lab@mxA# top 

[edit]
lab@mxA# show | compare 
[edit policy-options]
+   policy-statement import-localpref {
+       term 1 {
+           from {
+               protocol bgp;
+               neighbor 172.22.121.2;
+               route-filter 172.16.2.0/24 exact;
+           }
+           then {
+               local-preference 110;
+           }
+       }
+   }
[edit protocols bgp group P1-P2]
+    import import-localpref;

[edit]
lab@mxA# 

[edit]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 


lab@mxA> show route 172.16.2.0/24 detail 

inet.0: 21 destinations, 26 routes (21 active, 0 holddown, 0 hidden)
172.16.2.0/24 (2 entries, 1 announced)
        *BGP    Preference: 170/-111
                Next hop type: Router, Next hop index: 578
                Address: 0xc0f1bcc
                Next-hop reference count: 10
                Source: 172.22.121.2
                Next hop: 172.22.121.2 via ge-0/0/3.0, selected
                Session Id: 0x140
                State: <Active Ext>
                Local AS: 65001 Peer AS: 65412
                Age: 1:01 
                Validation State: unverified 
                Task: BGP_65412.172.22.121.2
                Announcement bits (3): 0-KRT 1-BGP_RT_Background 5-Resolve tree 1 
                AS path: 65412 65002 I 
                Aggregator: 65002 172.16.2.1
                Accepted
                Localpref: 110
                Router ID: 172.31.100.1
         BGP    Preference: 170/-101
                Next hop type: Router, Next hop index: 0
                Address: 0xc0f1d5c
                Next-hop reference count: 5
                Source: 172.22.123.2
                Next hop: 172.22.123.2 via ge-0/0/1.0, selected
                Session Id: 0x0
                State: <Ext>
                Inactive reason: Local Preference
                Local AS: 65001 Peer AS: 65412
                Age: 48:56 
                Validation State: unverified 
                Task: BGP_65412.172.22.123.2
                AS path: 65412 65002 I 
                Aggregator: 65002 172.16.2.1
                Accepted
                Localpref: 100
                Router ID: 172.31.101.1
                                        
lab@mxA>
```
</details><br />

<details markdown=1>
<summary markdown="span">mxB:R3-1</summary>

```
lab@mxB:R3-1> show route 172.16.2.0/24 detail 

inet.0: 18 destinations, 19 routes (18 active, 0 holddown, 0 hidden)
172.16.2.0/24 (2 entries, 1 announced)
        *BGP    Preference: 170/-111
                Next hop type: Indirect, Next hop index: 0
                Address: 0xc0d8c30
                Next-hop reference count: 10
                Source: 172.16.1.1
                Next hop type: Router, Next hop index: 768
                Next hop: 10.0.10.1 via ge-0/0/0.0, selected
                Session Id: 0x14a
                Protocol next hop: 172.16.1.1
                Indirect next hop: 0xc2df804 1048579 INH Session ID: 0x14b
                State: <Active Int Ext>
                Local AS: 65001 Peer AS: 65001
                Age: 3:14 	Metric2: 1 
                Validation State: unverified 
                Task: BGP_65001.172.16.1.1
                Announcement bits (3): 3-KRT 5-BGP_RT_Background 6-Resolve tree 1 
                AS path: 65412 65002 I 
                Aggregator: 65002 172.16.2.1
                Accepted
                Localpref: 110
                Router ID: 172.16.1.1
         BGP    Preference: 170/-101
                Next hop type: Indirect, Next hop index: 0
                Address: 0xc0d89d8
                Next-hop reference count: 9
                Source: 172.31.102.1
                Next hop type: Router, Next hop index: 745
                Next hop: 172.22.125.2 via ge-0/0/2.0, selected
                Session Id: 0x142
                Protocol next hop: 172.31.102.1
                Indirect next hop: 0xc2dfb04 1048575 INH Session ID: 0x145
                State: <Ext>
                Inactive reason: Local Preference
                Local AS: 65001 Peer AS: 65020
                Age: 1:23:36 	Metric2: 0 
                Validation State: unverified 
                Task: BGP_65020.172.31.102.1
                AS path: 65020 65002 I 
                Aggregator: 65002 172.16.2.2
                Accepted
                Localpref: 100
                Router ID: 172.31.102.1

lab@mxB:R3-1> 

lab@mxB:R3-1> configure 
Entering configuration mode

[edit]
lab@mxB:R3-1# edit policy-options policy-statement import-p3 

[edit policy-options policy-statement import-p3]
lab@mxB:R3-1# set term 1 from protocol bgp 

[edit policy-options policy-statement import-p3]
lab@mxB:R3-1# set term 1 from neighbor 172.31.102.1 

[edit policy-options policy-statement import-p3]
lab@mxB:R3-1# set term 1 from route-filter 172.16.2.0/24 exact 

[edit policy-options policy-statement import-p3]
lab@mxB:R3-1# set term 1 then local-preference 120 

[edit policy-options policy-statement import-p3]
lab@mxB:R3-1# show 
term 1 {
    from {
        protocol bgp;
        neighbor 172.31.102.1;
        route-filter 172.16.2.0/24 exact;
    }
    then {
        local-preference 120;
    }
}

[edit policy-options policy-statement import-p3]
lab@mxB:R3-1# top 

[edit]
lab@mxB:R3-1# edit protocols bgp group P3 

[edit protocols bgp group P3]
lab@mxB:R3-1# set import import-p3 

[edit protocols bgp group P3]
lab@mxB:R3-1# top 

[edit]
lab@mxB:R3-1# show | compare 
[edit logical-systems R3-1 protocols bgp group P3]
+    import import-p3;
[edit logical-systems R3-1 policy-options]
+   policy-statement import-p3 {
+       term 1 {
+           from {
+               protocol bgp;
+               neighbor 172.31.102.1;
+               route-filter 172.16.2.0/24 exact;
+           }
+           then {
+               local-preference 120;
+           }
+       }
+   }

[edit]
lab@mxB:R3-1# commit and-quit 
commit complete
Exiting configuration mode

lab@mxB:R3-1> 

lab@mxB:R3-1> show route 172.16.2.0/24 detail 

inet.0: 18 destinations, 18 routes (18 active, 0 holddown, 0 hidden)
172.16.2.0/24 (1 entry, 1 announced)
        *BGP    Preference: 170/-121
                Next hop type: Indirect, Next hop index: 0
                Address: 0xc0d89d8
                Next-hop reference count: 10
                Source: 172.31.102.1
                Next hop type: Router, Next hop index: 745
                Next hop: 172.22.125.2 via ge-0/0/2.0, selected
                Session Id: 0x142
                Protocol next hop: 172.31.102.1
                Indirect next hop: 0xc2dfb04 1048575 INH Session ID: 0x145
                State: <Active Ext>
                Local AS: 65001 Peer AS: 65020
                Age: 43 	Metric2: 0 
                Validation State: unverified 
                Task: BGP_65020.172.31.102.1
                Announcement bits (3): 3-KRT 5-BGP_RT_Background 6-Resolve tree 1 
                AS path: 65020 65002 I 
                Aggregator: 65002 172.16.2.2
                Accepted
                Localpref: 120
                Router ID: 172.31.102.1

lab@mxB:R3-1> 


lab@mxB:R3-1> configure 
Entering configuration mode

[edit]
lab@mxB:R3-1# delete protocols bgp group P3 import 

[edit]
lab@mxB:R3-1# show | compare 
[edit logical-systems R3-1 protocols bgp group P3]
-    import import-p3;

[edit]
lab@mxB:R3-1# commit and-quit 
commit complete
Exiting configuration mode

lab@mxB:R3-1> 
```
</details><br />

## Modifying the AS Path Attribute

<details markdown=1>
<summary markdown="span">mxC</summary>

```
lab@mxC> show route 172.16.1.0/24 

inet.0: 25 destinations, 31 routes (25 active, 0 holddown, 4 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.0/24      *[BGP/170] 00:57:12, localpref 100
                      AS path: 65412 65001 I, validation-state: unverified
                    >  to 172.22.122.2 via ge-0/0/3.0
                    [BGP/170] 00:57:11, localpref 100
                      AS path: 65412 65001 I, validation-state: unverified
                    >  to 172.22.124.2 via ge-0/0/1.0
                    [BGP/170] 01:26:49, localpref 100, from 172.16.2.2
                      AS path: 65020 65001 I, validation-state: unverified
                    >  to 10.0.14.2 via ge-0/0/0.0

lab@mxC> 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxB:R3-1</summary>

```
lab@mxB:R3-1> configure 
Entering configuration mode

[edit]
lab@mxB:R3-1# top edit policy-options 

[edit policy-options]
lab@mxB:R3-1# copy policy-statement export-aggregate to policy-statement export-p3 

[edit policy-options]
lab@mxB:R3-1# show policy-statement export-p3 
term 1 {
    from {
        protocol aggregate;
        route-filter 172.16.1.0/24 exact;
    }
    then accept;
}
term 2 {
    from {
        route-filter 172.16.1.0/24 longer;
    }
    then reject;
}

[edit policy-options]
lab@mxB:R3-1# 



[edit policy-options]
lab@mxB:R3-1# edit policy-statement export-p3 

[edit policy-options policy-statement export-p3]
lab@mxB:R3-1# set term 1 then as-path-prepend 65002 

[edit policy-options policy-statement export-p3]
lab@mxB:R3-1# top 

[edit]
lab@mxB:R3-1# edit protocols bgp 

[edit protocols bgp]
lab@mxB:R3-1# set group P3 neighbor 172.31.102.1 export export-p3 

[edit protocols bgp]
lab@mxB:R3-1# commit and-quit 
commit complete
Exiting configuration mode

lab@mxB:R3-1> 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxC</summary>

```
lab@mxC> show route 172.16.1.0/24    

inet.0: 25 destinations, 30 routes (25 active, 0 holddown, 4 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.0/24      *[BGP/170] 00:59:27, localpref 100
                      AS path: 65412 65001 I, validation-state: unverified
                    >  to 172.22.122.2 via ge-0/0/3.0
                    [BGP/170] 00:59:26, localpref 100
                      AS path: 65412 65001 I, validation-state: unverified
                    >  to 172.22.124.2 via ge-0/0/1.0

lab@mxC> 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxA</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# top edit policy-options policy-statement export-aggregate 

[edit policy-options policy-statement export-aggregate]
lab@mxA# show 
term 1 {
    from {
        protocol aggregate;
        route-filter 172.16.1.0/24 exact;
    }
    then accept;
}
term 2 {
    from {
        route-filter 172.16.1.0/24 longer;
    }
    then reject;
}

[edit policy-options policy-statement export-aggregate]
lab@mxA# set term 1 then as-path-prepend "65001 65001 65001" 

[edit policy-options policy-statement export-aggregate]
lab@mxA# top     

[edit]
lab@mxA# show | compare 
[edit policy-options policy-statement export-aggregate term 1 then]
+     as-path-prepend "65001 65001 65001";

[edit]
lab@mxA# 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxC</summary>

```
lab@mxC> show route 172.16.1.0/24 

inet.0: 25 destinations, 30 routes (25 active, 0 holddown, 4 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.0/24      *[BGP/170] 01:02:23, localpref 100
                      AS path: 65412 65001 I, validation-state: unverified
                    >  to 172.22.122.2 via ge-0/0/3.0
                    [BGP/170] 01:02:22, localpref 100
                      AS path: 65412 65001 I, validation-state: unverified
                    >  to 172.22.124.2 via ge-0/0/1.0

lab@mxC> 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxB:R3-1</summary>

```
lab@mxB:R3-1> configure 
Entering configuration mode

[edit]
lab@mxB:R3-1# top edit policy-options policy-statement export-p3 

[edit policy-options policy-statement export-p3]
lab@mxB:R3-1# show 
term 1 {
    from {
        protocol aggregate;
        route-filter 172.16.1.0/24 exact;
    }
    then {
        as-path-prepend 65002;
        accept;
    }
}
term 2 {
    from {
        route-filter 172.16.1.0/24 longer;
    }
    then reject;
}

[edit policy-options policy-statement export-p3]
lab@mxB:R3-1# delete term 1 then as-path-prepend 

[edit policy-options policy-statement export-p3]
lab@mxB:R3-1# top 

[edit]
lab@mxB:R3-1# show | compare 
[edit logical-systems R3-1 policy-options policy-statement export-p3 term 1 then]
-     as-path-prepend 65002;

[edit]
lab@mxB:R3-1# 


[edit]
lab@mxB:R3-1# commit and-quit 
commit complete
Exiting configuration mode

lab@mxB:R3-1> 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxC</summary>

```
lab@mxC> show route 172.16.1.0/24    

inet.0: 25 destinations, 31 routes (25 active, 0 holddown, 4 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.0/24      *[BGP/170] 01:04:20, localpref 100
                      AS path: 65412 65001 I, validation-state: unverified
                    >  to 172.22.122.2 via ge-0/0/3.0
                    [BGP/170] 01:04:19, localpref 100
                      AS path: 65412 65001 I, validation-state: unverified
                    >  to 172.22.124.2 via ge-0/0/1.0
                    [BGP/170] 00:00:21, localpref 100, from 172.16.2.2
                      AS path: 65020 65001 I, validation-state: unverified
                    >  to 10.0.14.2 via ge-0/0/0.0

lab@mxC> 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxA</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# edit policy-options policy-statement export-aggregate 

[edit policy-options policy-statement export-aggregate]
lab@mxA# show 
term 1 {
    from {
        protocol aggregate;
        route-filter 172.16.1.0/24 exact;
    }
    then {
        as-path-prepend "65001 65001 65001";
        accept;
    }
}
term 2 {
    from {
        route-filter 172.16.1.0/24 longer;
    }
    then reject;
}

[edit policy-options policy-statement export-aggregate]
lab@mxA# delete term 1 then as-path-prepend 

[edit policy-options policy-statement export-aggregate]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 


lab@mxA> show route receive-protocol bgp 172.22.121.2 

inet.0: 21 destinations, 26 routes (21 active, 0 holddown, 0 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
* 30.30.0.0/24            172.22.121.2                            65412 I
* 30.30.1.0/24            172.22.121.2                            65412 I
* 30.30.2.0/24            172.22.121.2                            65412 I
* 30.30.3.0/24            172.22.121.2                            65412 I
* 172.16.2.0/24           172.22.121.2                            65412 65002 I

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

lab@mxA> 


lab@mxA> show route receive-protocol bgp 172.22.121.2 aspath-regex ".* 65002"   

inet.0: 21 destinations, 26 routes (21 active, 0 holddown, 0 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
* 172.16.2.0/24           172.22.121.2                            65412 65002 I

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

lab@mxA> 



lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# top edit policy-options 

[edit policy-options]
lab@mxA# set as-path remote-as ".* 65002"

[edit policy-options]
lab@mxA# edit policy-statement import-P1 

[edit policy-options policy-statement import-P1]
lab@mxA# set term 1 from protocol bgp 

[edit policy-options policy-statement import-P1]
lab@mxA# set term 1 from as-path remote-as 

[edit policy-options policy-statement import-P1]
lab@mxA# set term 1 then accept 

[edit policy-options policy-statement import-P1]
lab@mxA# set term 2 then reject 

[edit policy-options policy-statement import-P1]
lab@mxA# top             

[edit]
lab@mxA# edit protocols bgp group P1-P2 

[edit protocols bgp group P1-P2]
lab@mxA# set neighbor 172.22.121.2 import import-P1 

[edit protocols bgp group P1-P2]
lab@mxA# top 

[edit]
lab@mxA# show | compare 
[edit policy-options]
+   policy-statement import-P1 {
+       term 1 {
+           from {
+               protocol bgp;
+               as-path remote-as;
+           }
+           then accept;
+       }
+       term 2 {
+           then reject;
+       }
+   }
[edit policy-options]
+   as-path remote-as ".* 65002";
[edit protocols bgp group P1-P2 neighbor 172.22.121.2]
+     import import-P1;

[edit]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 



lab@mxA> show route receive-protocol bgp 172.22.121.2 

inet.0: 21 destinations, 27 routes (21 active, 0 holddown, 4 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
* 172.16.2.0/24           172.22.121.2                            65412 65002 I

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

lab@mxA> 



lab@mxA> show route advertising-protocol bgp 172.22.121.2 

inet.0: 21 destinations, 27 routes (21 active, 0 holddown, 4 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
* 40.40.0.0/24            Self                                    65020 I
* 40.40.1.0/24            Self                                    65020 I
* 40.40.2.0/24            Self                                    65020 I
* 40.40.3.0/24            Self                                    65020 I
* 172.16.1.0/24           Self                                    I

lab@mxA> 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxB:R3-1</summary>

```
lab@mxB:R3-1> configure 
Entering configuration mode

[edit]
lab@mxB:R3-1# top edit routing-options 

[edit routing-options]
lab@mxB:R3-1# set static route 172.16.10.0/24 reject 

[edit routing-options]
lab@mxB:R3-1# commit 
commit complete

[edit routing-options]
lab@mxB:R3-1# 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxA</summary>

```
lab@mxA> show route advertising-protocol bgp 172.22.121.2 aspath-regex "()"  

inet.0: 22 destinations, 28 routes (22 active, 0 holddown, 4 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
* 172.16.1.0/24           Self                                    I
* 172.16.10.0/24          Self                                    I

lab@mxA> 

lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# edit policy-options 

[edit policy-options]
lab@mxA# set as-path internal-as "()"

[edit policy-options]
lab@mxA# edit policy-statement export-aggregate 

[edit policy-options policy-statement export-aggregate]
lab@mxA# set term 3 from protocol bgp 

[edit policy-options policy-statement export-aggregate]
lab@mxA# set term 3 from as-path internal-as 

[edit policy-options policy-statement export-aggregate]
lab@mxA# set term 3 then reject 

[edit policy-options policy-statement export-aggregate]
lab@mxA# top 

[edit]
lab@mxA# show | compare 
[edit policy-options policy-statement export-aggregate]
     term 2 { ... }
+    term 3 {
+        from {
+            protocol bgp;
+            as-path internal-as;
+        }
+        then reject;
+    }
[edit policy-options]
    as-path remote-as { ... }
+   as-path internal-as "()";

[edit]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 

lab@mxA> show route advertising-protocol bgp 172.22.121.2 aspath-regex "()" 

inet.0: 22 destinations, 28 routes (22 active, 0 holddown, 4 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
* 172.16.1.0/24           Self                                    I

lab@mxA> 
```
</details><br />

## References:

- [[Tutorials & Fixes] VNC Viewer Copy Paste on Windows, Linux, & Mac](https://www.anyviewer.com/how-to/vnc-viewer-copy-paste-2578.html)
