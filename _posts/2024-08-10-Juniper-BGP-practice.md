---
title: Juniper Junos OS BGP basic configurations
date: 2024-08-10 21:15:00 -0700
categories: [JUNIPER, BGP]
tags: [juniper, bgp]     # TAG names should always be lowercase
---

This post documents BGP configuration on Juniper Junos OS

## Network Topology

<details markdown=1>
<summary markdown="span">Display Network Topology</summary>

![]({{ site.baseurl }}/images/2024/08-10-Juniper-BGP-practice/01-Network-topology.png)

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
set routing-options static route 172.16.1.0/26 reject
set routing-options static route 172.16.1.64/26 reject
set protocols ospf area 0.0.0.0 interface lo0.0
set protocols ospf area 0.0.0.0 interface ge-0/0/0.0

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
set logical-systems R3-1 routing-options static route 172.16.1.128/26 reject
set logical-systems R3-1 routing-options static route 172.16.1.192/26 reject
set logical-systems R3-2 interfaces ge-0/0/1 unit 0 family inet address 10.0.14.2/24
set logical-systems R3-2 interfaces ge-0/0/5 unit 0 family inet address 172.22.126.1/24
set logical-systems R3-2 interfaces lo0 unit 2 family inet address 172.16.2.2/32
set logical-systems R3-2 protocols ospf area 0.0.0.0 interface lo0.2
set logical-systems R3-2 protocols ospf area 0.0.0.0 interface ge-0/0/1.0
set logical-systems R3-2 protocols bgp group ibgp type internal
set logical-systems R3-2 protocols bgp group ibgp local-address 172.16.2.2
set logical-systems R3-2 protocols bgp group ibgp export redistribute-statics
set logical-systems R3-2 protocols bgp group ibgp neighbor 172.16.2.1
set logical-systems R3-2 protocols bgp group P3 type external
set logical-systems R3-2 protocols bgp group P3 multihop
set logical-systems R3-2 protocols bgp group P3 local-address 172.16.2.2
set logical-systems R3-2 protocols bgp group P3 neighbor 172.31.102.1 peer-as 65020
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
set policy-options policy-statement pfe-load-balance term 1 from protocol bgp
set policy-options policy-statement pfe-load-balance term 1 from route-filter 30.30.0.0/22 longer
set policy-options policy-statement pfe-load-balance term 1 then load-balance per-packet
set policy-options policy-statement redistribute-statics term 1 from protocol static
set policy-options policy-statement redistribute-statics term 1 then accept
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
set protocols bgp group ibgp neighbor 172.16.2.2
set protocols bgp group P1-P2 type external
set protocols bgp group P1-P2 export export-aggregate
set protocols bgp group P1-P2 peer-as 65412
set protocols bgp group P1-P2 multipath
set protocols bgp group P1-P2 neighbor 172.22.122.2
set protocols bgp group P1-P2 neighbor 172.22.124.2

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
```
</details><br />

## Show commands

<details markdown=1>
<summary markdown="span">mxC</summary>

```
lab@mxC> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
10.0.14.2        ge-0/0/0.0             Full      172.16.2.2       128    38

lab@mxC> 


lab@mxC> show bgp summary 
Threading mode: BGP I/O
Groups: 2 Peers: 3 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                      14         10          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.16.2.2            65002        186        185       0       0     1:22:02 Establ
  inet.0: 2/6/6/0
172.22.122.2          65412        175        174       0       0     1:17:30 Establ
  inet.0: 4/4/4/0
172.22.124.2          65412        180        178       0       0     1:19:19 Establ
  inet.0: 4/4/4/0

lab@mxC> 
```

</details><br />

<details markdown=1>
<summary markdown="span">mxD</summary>

```
lab@mxD> show ospf neighbor logical-system P1 
Address          Interface              State     ID               Pri  Dead
172.22.252.2     ge-0/0/0.0             Full      172.31.101.1     128    38

lab@mxD> 


lab@mxD> show bgp summary logical-system P1 
Threading mode: BGP I/O
Groups: 3 Peers: 3 Down peers: 1
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       2          1          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.22.121.1          65001          0          0       0       0     1:18:40 Active
172.22.122.1          65002        177        177       0       0     1:18:29 Establ
  inet.0: 1/1/1/0
172.31.101.1          65412        176        176       0       0     1:18:19 Establ
  inet.0: 0/1/1/0

lab@mxD>
```

</details><br />

<details markdown=1>
<summary markdown="span">mxE</summary>

```
lab@mxE> show bgp summary logical-system P2 
Threading mode: BGP I/O
Groups: 3 Peers: 3 Down peers: 1
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       2          1          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.22.123.1          65001          0          0       0       0     1:19:53 Active
172.22.124.1          65002        179        179       0       0     1:19:42 Establ
  inet.0: 1/1/1/0
172.31.100.1          65412        176        174       0       0     1:17:42 Establ
  inet.0: 0/1/1/0

lab@mxE>
```

</details><br />

## Configuring iBGP Peering

<details markdown=1>
<summary markdown="span">mxA show commands</summary>

```
lab@mxA> show ospf neighbor 
Address          Interface              State     ID               Pri  Dead
10.0.10.2        ge-0/0/0.0             Full      172.16.1.2       128    32

lab@mxA> 

lab@mxA> show bgp summary 
BGP is not running

lab@mxA> 


lab@mxA> ping 172.22.121.2 count 5 
PING 172.22.121.2 (172.22.121.2): 56 data bytes
64 bytes from 172.22.121.2: icmp_seq=0 ttl=64 time=1.339 ms
64 bytes from 172.22.121.2: icmp_seq=1 ttl=64 time=1.244 ms
64 bytes from 172.22.121.2: icmp_seq=2 ttl=64 time=1.573 ms
64 bytes from 172.22.121.2: icmp_seq=3 ttl=64 time=1.176 ms
64 bytes from 172.22.121.2: icmp_seq=4 ttl=64 time=1.344 ms

--- 172.22.121.2 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/stddev = 1.176/1.335/1.573/0.134 ms

lab@mxA> 

lab@mxA> ping 172.22.123.2 count 5 
PING 172.22.123.2 (172.22.123.2): 56 data bytes
64 bytes from 172.22.123.2: icmp_seq=0 ttl=64 time=1.347 ms
64 bytes from 172.22.123.2: icmp_seq=1 ttl=64 time=1.397 ms
64 bytes from 172.22.123.2: icmp_seq=2 ttl=64 time=1.351 ms
64 bytes from 172.22.123.2: icmp_seq=3 ttl=64 time=1.150 ms
64 bytes from 172.22.123.2: icmp_seq=4 ttl=64 time=1.138 ms

--- 172.22.123.2 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/stddev = 1.138/1.277/1.397/0.110 ms

lab@mxA> 


lab@mxA> show route table inet.0 

inet.0: 13 destinations, 13 routes (13 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.10.0/24       *[Direct/0] 00:04:56
                    >  via ge-0/0/0.0
10.0.10.1/32       *[Local/0] 00:04:56
                       Local via ge-0/0/0.0
172.16.1.0/26      *[Static/5] 00:04:56
                       Reject
172.16.1.1/32      *[Direct/0] 3d 07:52:04
                    >  via lo0.0
172.16.1.2/32      *[OSPF/10] 00:04:46, metric 1
                    >  to 10.0.10.2 via ge-0/0/0.0
172.16.1.64/26     *[Static/5] 00:04:56
                       Reject
172.22.121.0/24    *[Direct/0] 00:04:56
                    >  via ge-0/0/3.0
172.22.121.1/32    *[Local/0] 00:04:56
                       Local via ge-0/0/3.0
172.22.123.0/24    *[Direct/0] 00:04:56
                    >  via ge-0/0/1.0
172.22.123.1/32    *[Local/0] 00:04:56
                       Local via ge-0/0/1.0
172.25.11.0/24     *[Direct/0] 3d 07:52:04
                    >  via fxp0.0
172.25.11.1/32     *[Local/0] 3d 07:52:04
                       Local via fxp0.0
224.0.0.5/32       *[OSPF/10] 00:04:56, metric 1
                       MultiRecv

lab@mxA> 
```

</details><br />

<details markdown=1>
<summary markdown="span">mxB show commands</summary>

```
lab@mxB> show bgp summary logical-system P3 
Threading mode: BGP I/O
Groups: 2 Peers: 2 Down peers: 1
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       0          0          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.16.1.2            65001          0          0       0       0        1:30 Connect
172.16.2.2            65002          4          5       0       0        1:20 Establ
  inet.0: 0/0/0/0

lab@mxB> 
```

</details><br />

<details markdown=1>
<summary markdown="span">mxA iBGP peering configuration</summary>

```
lab@mxA> configure 
Entering configuration mode

[edit]
lab@mxA# edit routing-options 

[edit routing-options]
lab@mxA# set autonomous-system 65001 

[edit routing-options]
lab@mxA# show | compare 
[edit routing-options]
+  autonomous-system 65001;

[edit routing-options]
lab@mxA# commit 
commit complete

[edit routing-options]
lab@mxA# 






[edit routing-options]
lab@mxA# top edit protocols bgp 

[edit protocols bgp]
lab@mxA# set group ibgp type internal 

[edit protocols bgp]
lab@mxA# set group ibgp local-address 172.16.1.1 

[edit protocols bgp]
lab@mxA# set group ibgp neighbor 172.16.1.2 

[edit protocols bgp]
lab@mxA# show | compare 
[edit protocols bgp]
+ group ibgp {
+     type internal;
+     local-address 172.16.1.1;
+     neighbor 172.16.1.2;
+ }

[edit protocols bgp]
lab@mxA# commit 
commit complete

[edit protocols bgp]
lab@mxA# 



lab@mxA# run show bgp summary 
Threading mode: BGP I/O
Groups: 1 Peers: 1 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       2          2          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.16.1.2            65001         10          8       0       0        2:48 Establ
  inet.0: 2/2/2/0

[edit protocols bgp]
lab@mxA# 
```

</details><br />

<details markdown=1>
<summary markdown="span">mxB iBGP peering configuration</summary>

```
lab@mxB> set cli logical-system R3-1 
Logical system: R3-1

lab@mxB:R3-1> 




lab@mxB:R3-1> configure 
Entering configuration mode

[edit]
lab@mxB:R3-1# edit routing-options 

[edit routing-options]
lab@mxB:R3-1# set autonomous-system 65001 

[edit routing-options]
lab@mxB:R3-1# 





[edit routing-options]
lab@mxB:R3-1# top edit protocols bgp 

[edit protocols bgp]
lab@mxB:R3-1# set group ibgp type internal 

[edit protocols bgp]
lab@mxB:R3-1# set group ibgp local-address 172.16.1.2 

[edit protocols bgp]
lab@mxB:R3-1# set group ibgp neighbor 172.16.1.1 

[edit protocols bgp]
lab@mxB:R3-1# commit 
commit complete

[edit protocols bgp]
lab@mxB:R3-1# 




lab@mxB:R3-1# run show bgp summary 
Threading mode: BGP I/O
Groups: 1 Peers: 1 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       0          0          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.16.1.1            65001          2          2       0       0          12 Establ
  inet.0: 0/0/0/0

[edit protocols bgp]
lab@mxB:R3-1# 
```

</details><br />

## Configuring eBGP Peering

<details markdown=1>
<summary markdown="span">mxA eBGP peering configuration</summary>

```
[edit protocols bgp]
lab@mxA# run show route protocol static 

inet.0: 15 destinations, 15 routes (15 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.0/26      *[Static/5] 00:20:25
                       Reject
172.16.1.64/26     *[Static/5] 00:20:25
                       Reject

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

[edit protocols bgp]
lab@mxA# 






[edit protocols bgp]
lab@mxA# top edit policy-options policy-statement redistribute-statics 

[edit policy-options policy-statement redistribute-statics]
lab@mxA# set term 1 from protocol static 

[edit policy-options policy-statement redistribute-statics]
lab@mxA# set term 1 then accept 

[edit policy-options policy-statement redistribute-statics]
lab@mxA# 





[edit policy-options policy-statement redistribute-statics]
lab@mxA# top edit protocols bgp 

[edit protocols bgp]
lab@mxA# set group ibgp export redistribute-statics 

[edit protocols bgp]
lab@mxA# commit 
commit complete

[edit protocols bgp]
lab@mxA# 





lab@mxA> show route advertising-protocol bgp 172.16.1.2 

inet.0: 15 destinations, 15 routes (15 active, 0 holddown, 0 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
* 172.16.1.0/26           Self                         100        I
* 172.16.1.64/26          Self                         100        I

lab@mxA> 





[edit]
lab@mxA# edit protocols bgp 

[edit protocols bgp]
lab@mxA# set group P1-P2 type external 

[edit protocols bgp]
lab@mxA# set group P1-P2 neighbor 172.22.121.2 

[edit protocols bgp]
lab@mxA# set group P1-P2 neighbor 172.22.123.2 

[edit protocols bgp]
lab@mxA# set group P1-P2 peer-as 65412 

[edit protocols bgp]
lab@mxA# commit 
commit complete

[edit protocols bgp]
lab@mxA# 





lab@mxA# run show bgp summary 
Threading mode: BGP I/O
Groups: 2 Peers: 3 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                      12          7          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.16.1.2            65001         20         21       0       0        7:20 Establ
  inet.0: 2/2/2/0
172.22.121.2          65412          4          3       0       0          13 Establ
  inet.0: 5/5/5/0
172.22.123.2          65412          4          3       0       0           9 Establ
  inet.0: 0/5/5/0

[edit protocols bgp]
lab@mxA# 





[edit protocols bgp]
lab@mxA# run show bgp neighbor 172.22.121.2 
Peer: 172.22.121.2+179 AS 65412 Local: 172.22.121.1+58903 AS 65001
  Group: P1-P2                 Routing-Instance: master
  Forwarding routing-instance: master  
  Type: External    State: Established    Flags: <Sync>
  Last State: OpenConfirm   Last Event: RecvKeepAlive
  Last Error: None
  Options: <Preference PeerAS Refresh>
  Options: <GracefulShutdownRcv>
  Holdtime: 90 Preference: 170
  Graceful Shutdown Receiver local-preference: 0
  Number of flaps: 0
  Peer ID: 172.31.100.1    Local ID: 172.16.1.1        Active Holdtime: 90
  Keepalive Interval: 30         Group index: 1    Peer index: 0    SNMP index: 1     
  I/O Session Thread: bgpio-0 State: Enabled
  BFD: disabled, down
  Local Interface: ge-0/0/3.0                       
  NLRI for restart configured on peer: inet-unicast
  NLRI advertised by peer: inet-unicast
  NLRI for this session: inet-unicast
  Peer supports Refresh capability (2)
  Stale routes from peer are kept for: 300
  Peer does not support Restarter functionality
  Restart flag received from the peer: Notification
  NLRI that restart is negotiated for: inet-unicast
  NLRI of received end-of-rib markers: inet-unicast
  NLRI of all end-of-rib markers sent: inet-unicast
  Peer does not support LLGR Restarter functionality
  Peer supports 4 byte AS extension (peer-as 65412)
  Peer does not support Addpath
  Table inet.0 Bit: 20001
    RIB State: BGP restart is complete
    Send state: in sync
    Active prefixes:              5
    Received prefixes:            5
    Accepted prefixes:            5
    Suppressed due to damping:    0
    Advertised prefixes:          2
  Last traffic (seconds): Received 15   Sent 12   Checked 42  
  Input messages:  Total 5	Updates 3 	Refreshes 0 	Octets 182
  Output messages: Total 3      Updates 1 	Refreshes 0 	Octets 95
  Output Queue[1]: 0            (inet.0, inet-unicast)

[edit protocols bgp]
lab@mxA# 






[edit protocols bgp]
lab@mxA# run show route receive-protocol bgp 172.22.121.2 

inet.0: 20 destinations, 25 routes (20 active, 0 holddown, 0 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
* 30.30.0.0/24            172.22.121.2                            65412 I
* 30.30.1.0/24            172.22.121.2                            65412 I
* 30.30.2.0/24            172.22.121.2                            65412 I
* 30.30.3.0/24            172.22.121.2                            65412 I
* 172.16.2.0/24           172.22.121.2                            65412 65002 I

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

[edit protocols bgp]
lab@mxA# 





[edit protocols bgp]
lab@mxA# run show route receive-protocol bgp 172.22.123.2  

inet.0: 20 destinations, 25 routes (20 active, 0 holddown, 0 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
  30.30.0.0/24            172.22.123.2                            65412 I
  30.30.1.0/24            172.22.123.2                            65412 I
  30.30.2.0/24            172.22.123.2                            65412 I
  30.30.3.0/24            172.22.123.2                            65412 I
  172.16.2.0/24           172.22.123.2                            65412 65002 I

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

[edit protocols bgp]
lab@mxA# 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxB eBGP peering configuration</summary>

```
[edit protocols bgp]
lab@mxB:R3-1# run show route protocol static 

inet.0: 9 destinations, 9 routes (9 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.16.1.128/26    *[Static/5] 01:42:17
                       Reject
172.16.1.192/26    *[Static/5] 01:42:17
                       Reject

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

[edit protocols bgp]
lab@mxB:R3-1# 




[edit protocols bgp]
lab@mxB:R3-1# top edit policy-options policy-statement redistribute-statics 

[edit policy-options policy-statement redistribute-statics]
lab@mxB:R3-1# set term 1 from protocol static 

[edit policy-options policy-statement redistribute-statics]
lab@mxB:R3-1# set term 1 then accept 

[edit policy-options policy-statement redistribute-statics]
lab@mxB:R3-1# 







[edit policy-options policy-statement redistribute-statics]
lab@mxB:R3-1# top edit protocols bgp 

[edit protocols bgp]
lab@mxB:R3-1# set group ibgp export redistribute-statics 

[edit protocols bgp]
lab@mxB:R3-1# show | compare 
[edit logical-systems R3-1 protocols bgp group ibgp]
+  export redistribute-statics;

[edit protocols bgp]
lab@mxB:R3-1# commit 
commit complete

[edit protocols bgp]
lab@mxB:R3-1# 









[edit protocols bgp]
lab@mxB:R3-1# top edit routing-options 

[edit routing-options]
lab@mxB:R3-1# show 
static {
    route 172.16.1.128/26 reject;
    route 172.16.1.192/26 reject;
}
autonomous-system 65001;

[edit routing-options]
lab@mxB:R3-1# set static route 172.31.102.1 next-hop 172.22.125.2 

[edit routing-options]
lab@mxB:R3-1# set static route 172.31.102.1 no-readvertise 

[edit routing-options]
lab@mxB:R3-1# show | compare 
[edit logical-systems R3-1 routing-options static]
    route 172.16.1.192/26 { ... }
+   route 172.31.102.1/32 {
+       next-hop 172.22.125.2;
+       no-readvertise;
+   }

[edit routing-options]
lab@mxB:R3-1# commit 
commit complete

[edit routing-options]
lab@mxB:R3-1# 





[edit routing-options]
lab@mxB:R3-1# run ping 172.31.102.1 source 172.16.1.2 count 5 
PING 172.31.102.1 (172.31.102.1): 56 data bytes
64 bytes from 172.31.102.1: icmp_seq=0 ttl=64 time=1.353 ms
64 bytes from 172.31.102.1: icmp_seq=1 ttl=64 time=1.261 ms
64 bytes from 172.31.102.1: icmp_seq=2 ttl=64 time=1.157 ms
64 bytes from 172.31.102.1: icmp_seq=3 ttl=64 time=1.366 ms
64 bytes from 172.31.102.1: icmp_seq=4 ttl=64 time=1.222 ms

--- 172.31.102.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/stddev = 1.157/1.272/1.366/0.079 ms

[edit routing-options]
lab@mxB:R3-1# 






[edit routing-options]
lab@mxB:R3-1# top edit protocols bgp 

[edit protocols bgp]
lab@mxB:R3-1# set group P3 type external 

[edit protocols bgp]
lab@mxB:R3-1# set group P3 local-address 172.16.1.2 

[edit protocols bgp]
lab@mxB:R3-1# set group P3 neighbor 172.31.102.1 

[edit protocols bgp]
lab@mxB:R3-1# set group P3 peer-as 65020 

[edit protocols bgp]
lab@mxB:R3-1# commit 
commit complete

[edit protocols bgp]
lab@mxB:R3-1# 







[edit protocols bgp]
lab@mxB:R3-1# run show bgp summary 
Threading mode: BGP I/O
Groups: 2 Peers: 2 Down peers: 1
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       7          2          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.16.1.1            65001         39         37       0       0       15:44 Establ
  inet.0: 2/7/7/0
172.31.102.1          65020          0          0       0       0          23 Idle  

[edit protocols bgp]
lab@mxB:R3-1# 
```
</details><br />

## BGP multipath configuration

<details markdown=1>
<summary markdown="span">mxA BGP multipath configuration</summary>

```
[edit protocols bgp]
lab@mxA# run show route 30.30/24 detail 

inet.0: 20 destinations, 25 routes (20 active, 0 holddown, 0 hidden)
30.30.0.0/24 (2 entries, 1 announced)
        *BGP    Preference: 170/-101
                Next hop type: Router, Next hop index: 586
                Address: 0xc0f1dc0
                Next-hop reference count: 10
                Source: 172.22.121.2
                Next hop: 172.22.121.2 via ge-0/0/3.0, selected
                Session Id: 0x142
                State: <Active Ext>
                Local AS: 65001 Peer AS: 65412
                Age: 1:36 
                Validation State: unverified 
                Task: BGP_65412.172.22.121.2
                Announcement bits (3): 0-KRT 4-BGP_RT_Background 5-Resolve tree 1 
                AS path: 65412 I 
                Accepted
                Localpref: 100
                Router ID: 172.31.100.1
         BGP    Preference: 170/-101
                Next hop type: Router, Next hop index: 0
                Address: 0xc0f1eec
                Next-hop reference count: 5
                Source: 172.22.123.2
                Next hop: 172.22.123.2 via ge-0/0/1.0, selected
                Session Id: 0x0
                State: <NotBest Ext Changed>
                Inactive reason: Not Best in its group - Active preferred
                Local AS: 65001 Peer AS: 65412
                Age: 1:32 
                Validation State: unverified 
                Task: BGP_65412.172.22.123.2
                AS path: 65412 I 
                Accepted
                Localpref: 100
                Router ID: 172.31.101.1

[edit protocols bgp]
lab@mxA# 



[edit protocols bgp]
lab@mxA# set group P1-P2 multipath 

[edit protocols bgp]
lab@mxA# show | compare 
[edit protocols bgp group P1-P2]
+   multipath;

[edit protocols bgp]
lab@mxA# commit 
commit complete

[edit protocols bgp]
lab@mxA# 




[edit protocols bgp]
lab@mxA# run show route 30.30/24 detail 

inet.0: 20 destinations, 25 routes (20 active, 0 holddown, 0 hidden)
30.30.0.0/24 (2 entries, 1 announced)
        *BGP    Preference: 170/-101
                Next hop type: Router, Next hop index: 0
                Address: 0xcad0b04
                Next-hop reference count: 6
                Source: 172.22.121.2
                Next hop: 172.22.121.2 via ge-0/0/3.0, selected
                Session Id: 0x0
                Next hop: 172.22.123.2 via ge-0/0/1.0
                Session Id: 0x0
                State: <Active Ext>
                Local AS: 65001 Peer AS: 65412
                Age: 13 
                Validation State: unverified 
                Task: BGP_65412.172.22.121.2
                Announcement bits (4): 0-KRT 4-BGP_RT_Background 5-Resolve tree 1 7-BGP_RT_Background 
                AS path: 65412 I 
                Accepted Multipath
                Localpref: 100
                Router ID: 172.31.100.1
         BGP    Preference: 170/-101
                Next hop type: Router, Next hop index: 585
                Address: 0xc0f1eec
                Next-hop reference count: 6
                Source: 172.22.123.2
                Next hop: 172.22.123.2 via ge-0/0/1.0, selected
                Session Id: 0x143
                State: <NotBest Ext>
                Inactive reason: Not Best in its group - Active preferred
                Local AS: 65001 Peer AS: 65412
                Age: 2:31 
                Validation State: unverified 
                Task: BGP_65412.172.22.123.2
                AS path: 65412 I 
                Accepted MultipathContrib
                Localpref: 100
                Router ID: 172.31.101.1
                                        
[edit protocols bgp]
lab@mxA#





[edit protocols bgp]
lab@mxA# run show route forwarding-table destination 30.30.0.0/24 
Routing table: default.inet
Internet:
Enabled protocols: Bridging, 
Destination        Type RtRef Next hop           Type Index    NhRef Netif
30.30.0.0/24       user     0 172.22.121.2       ucst      586     7 ge-0/0/3.0

Routing table: __pfe_private__.inet
Internet:
Enabled protocols: Bridging, 
Destination        Type RtRef Next hop           Type Index    NhRef Netif
default            perm     0                    dscd      514     2

Routing table: __juniper_services__.inet
Internet:
Enabled protocols: Bridging, 
Destination        Type RtRef Next hop           Type Index    NhRef Netif
default            perm     0                    dscd      523     2

Routing table: __master.anon__.inet
Internet:
Enabled protocols: Bridging, Dual VLAN, 
Destination        Type RtRef Next hop           Type Index    NhRef Netif
default            perm     0                    rjct      538     1

[edit protocols bgp]
lab@mxA# 






[edit protocols bgp]
lab@mxA# top edit policy-options policy-statement pfe-load-balance 

[edit policy-options policy-statement pfe-load-balance]
lab@mxA# set term 1 from protocol bgp 

[edit policy-options policy-statement pfe-load-balance]
lab@mxA# set term 1 from route-filter 30.30/22 longer 

[edit policy-options policy-statement pfe-load-balance]
lab@mxA# set term 1 then load-balance per-packet  

[edit policy-options policy-statement pfe-load-balance]
lab@mxA# show | compare 
[edit policy-options policy-statement pfe-load-balance]
+ term 1 {
+     from {
+         protocol bgp;
+         route-filter 30.30.0.0/22 longer;
+     }
+     then {
+         load-balance per-packet;
+     }
+ }

[edit policy-options policy-statement pfe-load-balance]
lab@mxA# 







[edit policy-options policy-statement pfe-load-balance]
lab@mxA# top edit routing-options forwarding-table 

[edit routing-options forwarding-table]
lab@mxA# set export pfe-load-balance 

[edit routing-options forwarding-table]
lab@mxA# commit 
commit complete

[edit routing-options forwarding-table]
lab@mxA# 







[edit routing-options forwarding-table]
lab@mxA# run show route forwarding-table destination 30.30./24 
Routing table: default.inet
Internet:
Enabled protocols: Bridging, 
Destination        Type RtRef Next hop           Type Index    NhRef Netif
30.30.0.0/24       user     0                    ulst  1048575     5
                              172.22.121.2       ucst      586     4 ge-0/0/3.0
                              172.22.123.2       ucst      585     5 ge-0/0/1.0

Routing table: __pfe_private__.inet
Internet:
Enabled protocols: Bridging, 
Destination        Type RtRef Next hop           Type Index    NhRef Netif
default            perm     0                    dscd      514     2

Routing table: __juniper_services__.inet
Internet:
Enabled protocols: Bridging, 
Destination        Type RtRef Next hop           Type Index    NhRef Netif
default            perm     0                    dscd      523     2

Routing table: __master.anon__.inet
Internet:
Enabled protocols: Bridging, Dual VLAN, 
Destination        Type RtRef Next hop           Type Index    NhRef Netif
default            perm     0                    rjct      538     1

[edit routing-options forwarding-table]
lab@mxA# 
```

</details><br />


<details markdown=1>
<summary markdown="span">mxB BGP multipath configuration</summary>

```
[edit protocols bgp]
lab@mxB:R3-1# set group P3 multihop 

[edit protocols bgp]
lab@mxB:R3-1# commit 
commit complete

[edit protocols bgp]
lab@mxB:R3-1# 
```
</details><br />

## BGP summarization

<details markdown=1>
<summary markdown="span">mxA BGP summarization configuration</summary>

```
[edit routing-options forwarding-table]
lab@mxA# up 

[edit routing-options]
lab@mxA# set aggregate route 172.16.1.0/24 

[edit routing-options]
lab@mxA# 






[edit routing-options]
lab@mxA# top edit policy-options policy-statement export-aggregate 

[edit policy-options policy-statement export-aggregate]
lab@mxA# set term 1 from protocol aggregate 

[edit policy-options policy-statement export-aggregate]
lab@mxA# set term 1 from route-filter 172.16.1.0/24 exact 

[edit policy-options policy-statement export-aggregate]
lab@mxA# set term 1 then accept 

[edit policy-options policy-statement export-aggregate]
lab@mxA# set term 2 from route-filter 172.16.1.0/24 longer 

[edit policy-options policy-statement export-aggregate]
lab@mxA# set term 2 then reject 

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
lab@mxA# 





[edit policy-options policy-statement export-aggregate]
lab@mxA# top edit protocols bgp group P1-P2 

[edit protocols bgp group P1-P2]
lab@mxA# set export export-aggregate 

[edit protocols bgp group P1-P2]
lab@mxA# commit and-quit 
commit complete
Exiting configuration mode

lab@mxA> 





lab@mxA> show route advertising-protocol bgp 172.22.121.2 

inet.0: 27 destinations, 32 routes (21 active, 0 holddown, 6 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
* 172.16.1.0/24           Self                                    I

lab@mxA> show route advertising-protocol bgp 172.22.123.2 

inet.0: 27 destinations, 32 routes (21 active, 0 holddown, 6 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
* 172.16.1.0/24           Self                                    I

lab@mxA> 
```
</details><br />

<details markdown=1>
<summary markdown="span">mxB BGP summarization configuration</summary>

```
[edit protocols bgp]
lab@mxB:R3-1# run show bgp summary 
Threading mode: BGP I/O
Groups: 2 Peers: 2 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                      13          8          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.16.1.1            65001         41         41       0       0       16:18 Establ
  inet.0: 2/7/7/0
172.31.102.1          65020          4          3       0       0           9 Establ
  inet.0: 6/6/6/0

[edit protocols bgp]
lab@mxB:R3-1# 




[edit protocols bgp]
lab@mxB:R3-1# run show route receive-protocol bgp 172.31.102.1 

inet.0: 23 destinations, 23 routes (18 active, 0 holddown, 5 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
* 40.40.0.0/24            172.31.102.1                            65020 I
* 40.40.1.0/24            172.31.102.1                            65020 I
* 40.40.2.0/24            172.31.102.1                            65020 I
* 40.40.3.0/24            172.31.102.1                            65020 I
* 172.16.2.0/26           172.31.102.1                            65020 65002 I
* 172.16.2.64/26          172.31.102.1                            65020 65002 I

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

[edit protocols bgp]
lab@mxB:R3-1# 







[edit protocols bgp]
lab@mxB:R3-1# top edit routing-options 

[edit routing-options]
lab@mxB:R3-1# set aggregate route 172.16.1.0/24 

[edit routing-options]
lab@mxB:R3-1# show 
static {
    route 172.16.1.128/26 reject;
    route 172.16.1.192/26 reject;
    route 172.31.102.1/32 {
        next-hop 172.22.125.2;
        no-readvertise;
    }
}
autonomous-system 65001;
aggregate {
    route 172.16.1.0/24;
}

[edit routing-options]
lab@mxB:R3-1# 







[edit routing-options]
lab@mxB:R3-1# top edit policy-options policy-statement export-aggragate 

[edit policy-options policy-statement export-aggragate]
lab@mxB:R3-1# set term 1 from protocol aggregate 

[edit policy-options policy-statement export-aggragate]
lab@mxB:R3-1# set term 1 from route-filter 172.16.1.0/24 exact 

[edit policy-options policy-statement export-aggragate]
lab@mxB:R3-1# set term 1 then accept 

[edit policy-options policy-statement export-aggragate]
lab@mxB:R3-1# set term 2 from route-filter 172.16.1.0/24 longer 

[edit policy-options policy-statement export-aggragate]
lab@mxB:R3-1# set term 2 then reject 

[edit policy-options policy-statement export-aggragate]
lab@mxB:R3-1# show 
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

[edit policy-options policy-statement export-aggragate]
lab@mxB:R3-1#   




[edit policy-options]
lab@mxB:R3-1# rename policy-statement export-aggragate to policy-statement export-aggregate 

[edit policy-options]
lab@mxB:R3-1# top         

[edit]
lab@mxB:R3-1# show | compare 
[edit logical-systems R3-1 protocols bgp group P3]
+    export export-aggregate;
[edit logical-systems R3-1 policy-options]
+   policy-statement export-aggregate {
+       term 1 {
+           from {
+               protocol aggregate;
+               route-filter 172.16.1.0/24 exact;
+           }
+           then accept;
+       }
+       term 2 {
+           from {
+               route-filter 172.16.1.0/24 longer;
+           }
+           then reject;
+       }
+   }
[edit logical-systems R3-1 routing-options]
+   aggregate {
+       route 172.16.1.0/24;
+   }

[edit]
lab@mxB:R3-1# commit 
commit complete

[edit]
lab@mxB:R3-1# 





[edit]
lab@mxB:R3-1# run show route advertising-protocol bgp 172.31.102.1 

inet.0: 24 destinations, 24 routes (19 active, 0 holddown, 5 hidden)
  Prefix		  Nexthop	       MED     Lclpref    AS path
* 172.16.1.0/24           Self                                    I

[edit]
lab@mxB:R3-1# 
```
</details><br />