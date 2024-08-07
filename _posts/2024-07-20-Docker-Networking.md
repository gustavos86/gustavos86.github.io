---
title: Docker Networking
date: 2024-07-20 14:30:00 -0700
categories: [DOCKER, NETWORKING]
tags: [docker]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

![]({{ site.baseurl }}/images/services/docker_logo.png)

## Types of networks in Docker

1. Bridge - This is the default network the Containerd are attached to. The default one is in the `172.17.0.0/16` subnet.
To access the containers service, network port mapping with the `-p <host_port>:<container_port>` is required.

```bash
docker run ubuntu
```

2. Host - Associates the container with the Host network. The ports are shared with the Docker Host and any other containers running in this "host network" type. No port mapping is required since the Container ports are already common with the Docker Host. The downside is that Containers isolation is lost. 

```bash
docker run ubuntu --network=host
```

3.  Null - Containers are not attached to any network and do not have communication to any external network nor with the other containers.

```bash
docker run ubuntu --network=none
```

## docker network

See the networks created in the Docker Host

```sudo
docker network ls
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo docker network ls
NETWORK ID     NAME                 DRIVER    SCOPE
a2faed9e7019   bridge               bridge    local
1d6d4741adf5   cloud_user_default   bridge    local
8e9ee852c122   host                 host      local
08f4db3e6cec   none                 null      local
cloud_user@553b1e446c1c:~$
```
</details><br />

See more information about a specific network in Docker

```bash
docker network inspect <network_id> | <network_name>
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo docker network inspect a2faed9e7019
[
    {
        "Name": "bridge",
        "Id": "a2faed9e70198f53b254f8c258523a8242522227555bf7dc228af6a5d56c0c02",
        "Created": "2024-07-20T21:03:37.456106174Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "3e3b199226f57d2329f611d456b08352517b17e34fd4f5ab6c4655fa0b760b11": {
                "Name": "relaxed_cori",
                "EndpointID": "f450f628d311d7073820819acbbe0f0daa9b4adcffda60dc3e4ac1e2361dce48",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "6b03340349ea0405fd9acd887a5ebcf5133dcf37edc527b9b22606e3fa967845": {
                "Name": "gallant_euclid",
                "EndpointID": "8a0618b1e06bc1bf1f6e550c0a0a8fb408e80e64b5de76735a2b8254ebddd1cb",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
cloud_user@553b1e446c1c:~$
```
</details><br />

To create a new Docker Network use

```bash
docker network create
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ sudo docker network create \
  --driver bridge \
  --subnet 172.31.0.0/16 \
  custom-isolated-network
5286bc46318928c02bea82ad35b68d10162bde086f71d30d2d4af092e2cf3167
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ sudo docker network ls
NETWORK ID     NAME                      DRIVER    SCOPE
a2faed9e7019   bridge                    bridge    local
1d6d4741adf5   cloud_user_default        bridge    local
5286bc463189   custom-isolated-network   bridge    local
8e9ee852c122   host                      host      local
08f4db3e6cec   none                      null      local
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ sudo docker network inspect custom-isolated-network
[
    {
        "Name": "custom-isolated-network",
        "Id": "5286bc46318928c02bea82ad35b68d10162bde086f71d30d2d4af092e2cf3167",
        "Created": "2024-07-20T21:56:27.260604107Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.31.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
cloud_user@553b1e446c1c:~$
```
</details><br />

## Embedded DNS

Containers use the same DNS servers as the host by default, but you can override this with `--dns`

By default, containers inherit the DNS settings as defined in the `/etc/resolv.conf` configuration file from the Docker Host.
Containers that attach to the default bridge network receive a copy of this file.

The Docker Host is using DNS **172.31.0.2**

```bash
resolvectl status
```

<details markdown=1>
<summary markdown="span">output</summary>

```bash
cloud_user@553b1e446c1c:~$ resolvectl status
Global
       Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
resolv.conf mode: stub

Link 2 (ens5)
    Current Scopes: DNS
         Protocols: +DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 172.31.0.2
       DNS Servers: 172.31.0.2
        DNS Domain: us-east-2.compute.internal

Link 3 (br-1d6d4741adf5)
Current Scopes: none
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 4 (docker0)
Current Scopes: none
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 13 (br-5286bc463189)
Current Scopes: none
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
cloud_user@553b1e446c1c:~$
```
</details><br />

Containers that attach to a custom network use Docker's embedded DNS server. The embedded DNS server forwards external DNS lookups to the DNS servers configured on the host.

**NOTE:** The embedded DNS is **127.0.0.11**

Quick lab to confirm this. `custom-isolated-network` was created by me.

```bash
cloud_user@553b1e446c1c:~$ sudo docker network ls
NETWORK ID     NAME                      DRIVER    SCOPE
a2faed9e7019   bridge                    bridge    local
1d6d4741adf5   cloud_user_default        bridge    local
5286bc463189   custom-isolated-network   bridge    local
8e9ee852c122   host                      host      local
08f4db3e6cec   none                      null      local
cloud_user@553b1e446c1c:~$
```

Starting 2 Containers based un **ubuntu** image.

```bash
cloud_user@553b1e446c1c:~$ sudo docker run -d --name ubuntu1 --network custom-isolated-network ubuntu sleep 3000
ca6c02496ef2744785b87f5e4da43017ad93ffcae0173dfb6155496ed135e8c2
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ sudo docker run -d --name ubuntu2 --network custom-isolated-network ubuntu sleep 3000
baa34ef8e76c5838aa39648693b9131c8b02a37295675bc689c4e7ebf489ff14
cloud_user@553b1e446c1c:~$

cloud_user@553b1e446c1c:~$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND        CREATED          STATUS          PORTS     NAMES
baa34ef8e76c   ubuntu    "sleep 3000"   13 seconds ago   Up 12 seconds             ubuntu2
ca6c02496ef2   ubuntu    "sleep 3000"   22 seconds ago   Up 21 seconds             ubuntu1
cloud_user@553b1e446c1c:~$
```

Log in to the Container to see what DNS server they are using:

```bash
cloud_user@553b1e446c1c:~$ sudo docker exec -it ubuntu1 bash
root@ca6c02496ef2:/# cat /etc/resolv.conf
# Generated by Docker Engine.
# This file can be edited; Docker Engine will not make further changes once it
# has been modified.

nameserver 127.0.0.11
search us-east-2.compute.internal
options edns0 trust-ad ndots:0

# Based on host file: '/etc/resolv.conf' (internal resolver)
# ExtServers: [host(127.0.0.53)]
# Overrides: []
# Option ndots from: internal
root@ca6c02496ef2:/#

cloud_user@553b1e446c1c:~$ sudo docker exec -it ubuntu2 bash
root@baa34ef8e76c:/# cat /etc/resolv.conf
# Generated by Docker Engine.
# This file can be edited; Docker Engine will not make further changes once it
# has been modified.

nameserver 127.0.0.11
search us-east-2.compute.internal
options edns0 trust-ad ndots:0

# Based on host file: '/etc/resolv.conf' (internal resolver)
# ExtServers: [host(127.0.0.53)]
# Overrides: []
# Option ndots from: internal
root@baa34ef8e76c:/#
```

Install ping & nslookup on Ubuntu container

```bash
apt update
apt install inetutils-ping -y
apt install dnsutils -y
```

Validate ping & nslookup between containers

From **ubuntu2** container

```bash
root@baa34ef8e76c:/# nslookup ubuntu1
Server:		127.0.0.11
Address:	127.0.0.11#53

Non-authoritative answer:
Name:	ubuntu1
Address: 172.31.0.2

root@baa34ef8e76c:/#
root@baa34ef8e76c:/# ping ubuntu1 -c 5
PING ubuntu1 (172.31.0.2): 56 data bytes
64 bytes from 172.31.0.2: icmp_seq=0 ttl=64 time=0.088 ms
64 bytes from 172.31.0.2: icmp_seq=1 ttl=64 time=0.092 ms
64 bytes from 172.31.0.2: icmp_seq=2 ttl=64 time=0.088 ms
64 bytes from 172.31.0.2: icmp_seq=3 ttl=64 time=0.093 ms
64 bytes from 172.31.0.2: icmp_seq=4 ttl=64 time=0.076 ms
--- ubuntu1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.076/0.087/0.093/0.000 ms
root@baa34ef8e76c:/#
```

## References

- [Networking overview](https://docs.docker.com/network/)