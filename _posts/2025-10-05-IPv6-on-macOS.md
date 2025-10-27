---
title: IPv6 on macOS
date: 2025-10-05 18:30:00 -0700
categories: [IPv6, macOS]
tags: [ipv6, macos]     # TAG names should always be lowercase
---

macOS gets three IPv6 addresses:

|   Address Type   |                    Purpose                    |    Lifespan    | Defined By |
|------------------|-----------------------------------------------|----------------|------------|
| Link-local       | Local network communication                   | Always present | RFC 4291   |
| Stable global    | Persistent IPv6 identity (e.g., DNS, inbound) | Long-lived     | RFC 7217   |
| Temporary global | Privacy-protected for outbound use            | Short-lived    | RFC 4941   |

Troubleshooting commands:

- `ifconfig -v en0`

The stable global address (`secured`) has a long lifetime.

The `temporary` one has a shorter lifetime (rotated automatically by macOS).

Example:

```
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500 index 11
	eflags=452008d0<ADV_REPORT,ACCEPT_RTADV,TXSTART,ARPLL,NOACKPRI,ECN_ENABLE,CHANNEL_DRV,FASTLN_ON>
	options=6460<TSO4,TSO6,CHANNEL_IO,PARTIAL_CSUM,ZEROINVERT_CSUM>
	hwassist=703000<CSUM_PARTIAL,CSUM_ZERO_INVERT,MULTIPAGES,TSO_V4,TSO_V6>
	ether a0:9a:8e:90:a6:62
	inet6 fe80::10d9:bcb0:601f:16fa%en0 prefixlen 64 secured scopeid 0xb
	inet6 2602:ae:15a4:8001:424:837a:870:6095 prefixlen 64 autoconf secured
	inet6 2602:ae:15a4:8001:edf7:1b86:b5fc:9085 prefixlen 64 autoconf temporary
	inet 192.168.0.106 netmask 0xffffff00 broadcast 192.168.0.255
	netif: EAC9C25E-FC35-4485-A08E-34485A15664D
	flowswitch: 1F20873E-7382-4A87-9768-EED8C7BCAB99
	nd6 options=201<PERFORMNUD,DAD>
	media: autoselect
	status: active
	generation id: 11
	type: Wi-Fi
	agent domain:Skywalk type:NetIf flags:0xa443 desc:"Userspace Networking"
	agent domain:Skywalk type:FlowSwitch flags:0x4483 desc:"Userspace Networking"
	agent domain:WiFiManager type:CallInProgress flags:0x3d desc:"WiFi"
	link quality: 100 (good)
	state availability: 0 (true)
	scheduler: FQ_CODEL (driver managed)
	uplink rate: 88.31 Mbps [eff] / 146.08 Mbps
	downlink rate: 88.31 Mbps [eff] / 146.08 Mbps [max]
	qosmarking enabled: yes mode: none
	low power mode: disabled
	multi layer packet logging (mpklog): disabled
	routermode4: disabled
	routermode6: disabled
```

- Global IPv6 privacy settings `sysctl net.inet6.ip6.use_tempaddr`

Example:

```
net.inet6.ip6.use_tempaddr: 1
```

Output meanings:

0 → Privacy extensions disabled (only stable addresses)

1 → Enabled (temporary addresses used)

2 → Enabled and always preferred for outgoing connections (default on macOS)

- This shows neighbor cache `ndp -a`

```
Neighbor                                Linklayer Address  Netif Expire    St Flgs Prbs
2602:ae:15a4:8001::1                    c:83:cc:fb:a1:7      en0 17s       R  R
2602:ae:15a4:8001:424:837a:870:6095     a0:9a:8e:90:a6:62    en0 permanent R
2602:ae:15a4:8001:edf7:1b86:b5fc:9085   a0:9a:8e:90:a6:62    en0 permanent R
fe80::e83:ccff:fefb:a107%en0            c:83:cc:fb:a1:7      en0 22s       R  R
```

- `scutil --nwi`

This shows the Network Interface Info (NWI) table and which IPv6 addresses are considered active/preferred for routing.


```
Network information

IPv4 network interface information
     en0 : flags      : 0x7 (IPv4,IPv6,DNS)
           address    : 192.168.0.106
           reach      : 0x00000002 (Reachable)

   REACH : flags 0x00000002 (Reachable)

IPv6 network interface information
     en0 : flags      : 0x7 (IPv4,IPv6,DNS)
           address    : 2602:ae:15a4:8001:424:837a:870:6095
           reach      : 0x00000002 (Reachable)

   REACH : flags 0x00000002 (Reachable)

Network interfaces: en0
```

- `scutil`

Using the network database to inspect the full interface configuration as known to networkd:

```
scutil
> show State:/Network/Interface/en0/IPv6
```

Example:

```
> show State:/Network/Interface/en0/IPv6
<dictionary> {
  Addresses : <array> {
    0 : fe80::10d9:bcb0:601f:16fa
    1 : 2602:ae:15a4:8001:424:837a:870:6095
    2 : 2602:ae:15a4:8001:edf7:1b86:b5fc:9085
  }
  Flags : <array> {
    0 : 1024
    1 : 1088
    2 : 192
  }
  PrefixLength : <array> {
    0 : 64
    1 : 64
    2 : 64
  }
}
>
```

- To see the macOS Routing Table

```
netstat -nr -f inet6
```

- To see only the default route

```
netstat -nr -f inet6 | grep default | grep en0
```