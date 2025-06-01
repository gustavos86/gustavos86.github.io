---
title: Starting with Juniper SRX Firewalls
date: 2025-06-01 15:00:00 -0700
categories: [JUNIPER, JUNOS OS, SRX]
tags: [juniper]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/juniper_logo.png)

## Packet Mode Processing

The SRX basically operates like a Router.

```
set security forwarding-options family mpls mode packet-based
```

- [[SRX] How to change forwarding mode for IPv4 from 'flow based' to 'packet based'](https://supportportal.juniper.net/s/article/SRX-How-to-change-forwarding-mode-for-IPv4-from-flow-based-to-packet-based?language=en_US)

## Logical Packet Flow

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/logical-packet-flow-diagram.png)

