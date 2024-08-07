---
title: Analyzing Traceroute with Packet Captures on Linux and Windows
date: 2024-08-04 19:00:00 -0700
categories: [NETWORKING, TRACEROUTE]
tags: [traceroute]     # TAG names should always be lowercase
---

## Introduction

Traceroute probes each Router in the path to the target IP address by making every Router in the path reply with a **ICMP Time Exceeded (Type 11) – time to live exceeded intransit (Code 0)** as the time-to-leave (**TTL** field in the IP header) expires in transit.

- On **Linux**, traceroute makes use of **UDP datagrams** with **Destination ranging from port 33434 to 33534**
  - The **Linux** implementation the Traceroute knows it has finally reached the target IP by receiving an **ICMP Destination Unreachable (Type 3) – PortUnreachable packet (Code 3)**
- On **Windows**, traceroute makes use of **ICMP echo request (Type 8 – Code 0)** to accomplish this
  - On **Windows** implementation the Traceroute knows it has finally reached the target IP by receiving an **ICMP echo reply (Type 0 – Code 0)implementation the Traceroute knows it has finally reached the target IP by receiving an

To see the ICMP Type and Code numbers, refer to [RFC792 - INTERNET CONTROL MESSAGE PROTOCOL](https://datatracker.ietf.org/doc/html/rfc792)

### Testbed

- VM running Linux Ubuntu 21.04 Hirsute Hippo
- VM running Windows 10 Enterprise

### ICMP messages relevant to Traceroute

- ICMP Time Exceeded (Type 11) – time to live exceeded in transit (Code 0)
- ICMP Destination Unreachable (Type 3) – Port Unreachable packet (Code 3)
- ICMP echo request (Type 8 – Code 0)
- ICMP echo reply (Type 0 – Code 0)

## Linux traceroute

- [Traceroute_Linux.pcapng]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/Traceroute_Linux.pcapng)

| ![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/01-Linux-traceroute-overview.png) | 
|:--:| 
| *Linux implementation of Traceroute receiving ICMP TTL exceeded in transit message from every Router in the path* |

| ![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/02-Linux-traceroute-completion.png) | 
|:--:| 
| *Linux implementation of Traceroute receiving ICMP Port Unreachable message from the last node in the path* |

First, let’s see the available parameters the traceroute utility has on my Linux Ubuntu. My Ubuntu fresh install was very explicit about indicating me I needed to perform `sudo apt install traceroute` to get it installed.

It seems you can get two different versions of the traceroute binary in Linux.

```bash
user@ubuntuVM01:~$ traceroute
Command 'traceroute' not found, but can be installed with:
sudo apt install inetutils-traceroute  # version 2:2.4-3ubuntu1, or
sudo apt install traceroute            # version 1:2.1.5-1
user@ubuntuVM01:~$
```

- `apt install traceroute`

<details markdown=1>
<summary markdown="span">traceroute</summary>


```bash
user@ubuntuVM01:~$ traceroute
Usage:
  traceroute [ -46dFITnreAUDV ] [ -f first_ttl ] [ -g gate,... ] [ -i device ] [ -m max_ttl ] [ -N squeries ] [ -p port ] [ -t tos ] [ -l flow_label ] [ -w MAX,HERE,NEAR ] [ -q nqueries ] [ -s src_addr ] [ -z sendwait ] [ --fwmark=num ] host [ packetlen ]
Options:
  -4                          Use IPv4
  -6                          Use IPv6
  -d  --debug                 Enable socket level debugging
  -F  --dont-fragment         Do not fragment packets
  -f first_ttl  --first=first_ttl
                              Start from the first_ttl hop (instead from 1)
  -g gate,...  --gateway=gate,...
                              Route packets through the specified gateway
                              (maximum 8 for IPv4 and 127 for IPv6)
  -I  --icmp                  Use ICMP ECHO for tracerouting
  -T  --tcp                   Use TCP SYN for tracerouting (default port is 80)
  -i device  --interface=device
                              Specify a network interface to operate with
  -m max_ttl  --max-hops=max_ttl
                              Set the max number of hops (max TTL to be
                              reached). Default is 30
  -N squeries  --sim-queries=squeries
                              Set the number of probes to be tried
                              simultaneously (default is 16)
  -n                          Do not resolve IP addresses to their domain names
  -p port  --port=port        Set the destination port to use. It is either
                              initial udp port value for "default" method
                              (incremented by each probe, default is 33434), or
                              initial seq for "icmp" (incremented as well,
                              default from 1), or some constant destination
                              port for other methods (with default of 80 for
                              "tcp", 53 for "udp", etc.)
  -t tos  --tos=tos           Set the TOS (IPv4 type of service) or TC (IPv6
                              traffic class) value for outgoing packets
  -l flow_label  --flowlabel=flow_label
                              Use specified flow_label for IPv6 packets
  -w MAX,HERE,NEAR  --wait=MAX,HERE,NEAR
                              Wait for a probe no more than HERE (default 3)
                              times longer than a response from the same hop,
                              or no more than NEAR (default 10) times than some
                              next hop, or MAX (default 5.0) seconds (float
                              point values allowed too)
  -q nqueries  --queries=nqueries
                              Set the number of probes per each hop. Default is
                              3
  -r                          Bypass the normal routing and send directly to a
                              host on an attached network
  -s src_addr  --source=src_addr
                              Use source src_addr for outgoing packets
  -z sendwait  --sendwait=sendwait
                              Minimal time interval between probes (default 0).
                              If the value is more than 10, then it specifies a
                              number in milliseconds, else it is a number of
                              seconds (float point values allowed too)
  -e  --extensions            Show ICMP extensions (if present), including MPLS
  -A  --as-path-lookups       Perform AS path lookups in routing registries and
                              print results directly after the corresponding
                              addresses
  -M name  --module=name      Use specified module (either builtin or external)
                              for traceroute operations. Most methods have
                              their shortcuts (`-I' means `-M icmp' etc.)
  -O OPTS,...  --options=OPTS,...
                              Use module-specific option OPTS for the
                              traceroute module. Several OPTS allowed,
                              separated by comma. If OPTS is "help", print info
                              about available options
  --sport=num                 Use source port num for outgoing packets. Implies
                              `-N 1'
  --fwmark=num                Set firewall mark for outgoing packets
  -U  --udp                   Use UDP to particular port for tracerouting
                              (instead of increasing the port per each probe),
                              default port is 53
  -UL                         Use UDPLITE for tracerouting (default dest port
                              is 53)
  -D  --dccp                  Use DCCP Request for tracerouting (default port
                              is 33434)
  -P prot  --protocol=prot    Use raw packet of protocol prot for tracerouting
  --mtu                       Discover MTU along the path being traced. Implies
                              `-F -N 1'
  --back                      Guess the number of hops in the backward path and
                              print if it differs
  -V  --version               Print version info and exit
  --help                      Read this help and exit

Arguments:
+     host          The host to traceroute to
      packetlen     The full packet length (default is the length of an IP
                    header plus 40). Can be ignored or increased to a minimal
                    allowed value
user@ubuntuVM01:~$
```
</details><br />

- `apt install inetutils-traceroute`

<details markdown=1>
<summary markdown="span">inetutils-traceroute</summary>

```bash
user@ubuntuVM01:~$ traceroute
traceroute: missing host operand
Try 'traceroute --help' or 'traceroute --usage' for more information.
user@ubuntuVM01:~$

user@ubuntuVM01:~$ traceroute --usage
Usage: traceroute [-I?V] [-f NUM] [-g GATES] [-m NUM] [-M METHOD] [-p PORT]
            [-q NUM] [-t NUM] [-w NUM] [--first-hop=NUM] [--gateways=GATES]
            [--icmp] [--max-hop=NUM] [--type=METHOD] [--port=PORT]
            [--tries=NUM] [--resolve-hostnames] [--tos=NUM] [--wait=NUM]
            [--help] [--usage] [--version] HOST
user@ubuntuVM01:~$

user@ubuntuVM01:~$ traceroute --help
Usage: traceroute [OPTION...] HOST
Print the route packets trace to network host.

  -f, --first-hop=NUM        set initial hop distance, i.e., time-to-live
  -g, --gateways=GATES       list of gateways for loose source routing
  -I, --icmp                 use ICMP ECHO as probe
  -m, --max-hop=NUM          set maximal hop count (default: 64)
  -M, --type=METHOD          use METHOD (`icmp' or `udp') for traceroute
                             operations, defaulting to `udp'
  -p, --port=PORT            use destination PORT port (default: 33434)
  -q, --tries=NUM            send NUM probe packets per hop (default: 3)
      --resolve-hostnames    resolve hostnames
  -t, --tos=NUM              set type of service (TOS) to NUM
  -w, --wait=NUM             wait NUM seconds for response (default: 3)
  -?, --help                 give this help list
      --usage                give a short usage message
  -V, --version              print program version

Mandatory or optional arguments to long options are also mandatory or optional
for any corresponding short options.

Report bugs to <bug-inetutils@gnu.org>.
user@ubuntuVM01:~$
```
</details><br />

I will be using the first `traceroute` binary, the one which was installed with `apt install traceroute`.

While preparing the screenshots for this post, I quickly noticed I should perform the traceroute command adding some of the optional parameters

- `-n` makes the traceroute always show the IP address of all Routers as opposed to the domain name
- `z 1` adds delay to the probes so the Wireshark capture can show the packets in the correct order
- `-q 1` sends a single probe instead of the three ones per hop which is the default

The traceroute is to to destination **8.8.8.8** which is a well known DNS server provided by Google and a common way to confirm Internet IP connectivity is working.

| ![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/03-Traceroute-output-on-Linux.png) | 
|:--:| 
| *Screenshot of running the traceroute command as seen on my Linux machine* |

Going to the Wireshark capture I have running in parallel to the `traceroute` exeuction above, I have used the following filter **in Wireshark** to avoid other packets to disturb the flow fo the traffic created by the `traceroute` command. Doing this the capture looks “cleaner” in the screenshot.

```
((udp.port >= 33434 and udp.port <= 33534) or (icmp)) and ip.ttl < 10
```

We clearly see from the Wireshark capture that we are sending UDP datagrams from this Linux machine with private IP address of 192.168.0.234 to the destination IP on the traceroute of 8.8.8.8

| ![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/04-Traceroute-Linux-in-Wireshark.png) | 
|:--:| 
| *Screenshot of the traffic generated by the traceroute command run from my Linux machine* |

Focusing on the very first entry in the traceroute command (partial output below)

```bash
user@ubuntuVM01:~$ traceroute 8.8.8.8 -n -z 1 -q 1
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
1 192.168.0.1 4.269 ms
```

We see the UDP packet with **TTL = 1 in the IP header** and the **UDP Destination Port of 33434**

![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/05-Traceroute-Linux-in-Wireshark-more.png)

Next packet is an expected **ICMP Time Exceeded (Type 11) – time to live exceeded in transit (Code 0)** as observed in the below packet capture. This is an indication to the traceroute utility that we should add the IP Source to our traceroute output

![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/06-Traceroute-Linux-in-Wireshark-more-info.png)

The process repeats itself by increasing the **TTL by 1** (and the **UDP Destination Port** as well). I added an additional column on the Wireshark capture to clearly see this behavior

![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/07-Traceroute-Linux-in-Wireshark-even-more.png)

Interesting enough, looking at the 7th time the traceroute sends a probe, we see it does not receive an yresponse as appreciated in the Wireshark capture above. This can be correlated with the 7th entr yin the traceroute, which is missing only showing an `*`

```bash
user@ubuntuVM01:~$ traceroute 8.8.8.8 -n -z 1 -q 1
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
1 192.168.0.1 4.269 ms
2 84.116.254.140 17.070 ms
3 84.116.253.129 20.324 ms
4 84.116.133.29 21.067 ms
5 84.116.138.73 22.080 ms
6 72.14.203.234 20.972 ms
7 *
8 8.8.8.8 24.669 ms
user@ubuntuVM01:~$
```

Finally, the **ICMP Destination Unreachable (Type 3) – Port Unreachable packet (Code 3)** is letting `traceroute` know it has reached the target IP. This method makes sense as there may be some mechanisms in the network like NAT (Network Address Translations) that may blur the path to the target IP.

![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/08-Traceroute-Linux-in-Wireshark-last.png)

## Windows traceroute

- [Traceroute_Windows.pcapng]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/Traceroute_Windows.pcapng)

| ![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/09-Windows-traceroute-overview.png) | 
|:--:| 
| *Windows implementation of Traceroute receiving ICMP TTL exceeded in transit message from every Router in the path* |

| ![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/10-Windows-traceroute-completion.png) | 
|:--:| 
| *Windows implementation of Traceroute receiving ICMP echp reply message from the lastnode in the path* |

In a similar way, let’s first look at the traceroute parameters available on the Windows platform. We must call the binary `tracert` as opposed to `traceroute`…

```
C:\Users\user>traceroute
'traceroute' is not recognized as an internal or external command,
operable program or batch file.

C:\Users\user>
```

```
C:\Users\user>tracert

Usage: tracert [-d] [-h maximum_hops] [-j host-list] [-w timeout]
               [-R] [-S srcaddr] [-4] [-6] target_name

Options:
    -d                 Do not resolve addresses to hostnames.
    -h maximum_hops    Maximum number of hops to search for target.
    -j host-list       Loose source route along host-list (IPv4-only).
    -w timeout         Wait timeout milliseconds for each reply.
    -R                 Trace round-trip path (IPv6-only).
    -S srcaddr         Source address to use (IPv6-only).
    -4                 Force using IPv4.
    -6                 Force using IPv6.

C:\Users\user>
```

There is no parameter on the Windows traceroute command to send only a single probe, but I included the `-d` parameter to make traceroute always show the IP address of all Routers in the path as opposed to the domain name

```
>tracert -d 8.8.8.8
```

| ![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/11-Windows-tracert-output.png) | 
|:--:| 
| *Screenshot of running the traceroute command as seen on my Windows machine* |

In this case, we see from the Wireshark capture that we are sending ICMP echo requests from the Windows machine with **source IP** address 192.168.0.24 to the **destination IP** 8.8.8.8

In this case the filter used in Wireshark is simply:

```
icmp
```

| ![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/12-Traceroute-Windows-in-Wireshark.png) | 
|:--:| 
| *Screenshot of the traffic generated by the tracert command run from my Windows machine* |

The very first entry in the traceroute command (partial output below) shows 3 probes sent with **TTL = 1** which were replied by 192.168.0.1

```
C:\Users\hecserra>tracert -d 8.8.8.8
Tracing route to 8.8.8.8 over a maximum of 30 hops

1 4 ms 3 ms 4 ms 192.168.0.1
...
```

```
C:\Users\hecserra>tracert -d 8.8.8.8
Tracing route to 8.8.8.8 over a maximum of 30 hops
1 4 ms 3 ms 4 ms 192.168.0.1
2 15 ms 18 ms 17 ms 84.116.254.140
3 19 ms 27 ms 22 ms 84.116.253.129
4 21 ms 20 ms 21 ms 84.116.133.29
5 20 ms 19 ms 24 ms 84.116.138.73
6 21 ms 23 ms 20 ms 72.14.203.234
7 29 ms 24 ms 25 ms 142.250.227.17
8 28 ms 20 ms 20 ms 209.85.252.117
9 26 ms 27 ms 28 ms 8.8.8.8
Trace complete.
```

We can see in the following screenshot the **ICMP echo request (Type 8 – Code 0)** generated by Windows and initiating the **traceroute**

![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/13-Traceroute-Windows-echo-request.png)

In the next screenshot, we the **ICMP Time Exceeded (Type 11) – time to live exceeded in transit (Code 0)** generate by the nodes in the path

![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/14-Traceroute-Windows-exceeded-in-transit.png)

Lastly, in **Windows**, the traceroute completes when receiving **ICMP echo reply (Type 0 – Code 0)** from the destionation. This behavior differs from Linux.

![]({{ site.baseurl }}/images/2024/08-04-Analyzing-Traceroutes-with-Packet-Captures/15-Traceroute-Windows-in-Wireshark-last.png)

## References

- [RFC792 - INTERNET CONTROL MESSAGE PROTOCOL](https://datatracker.ietf.org/doc/html/rfc792)