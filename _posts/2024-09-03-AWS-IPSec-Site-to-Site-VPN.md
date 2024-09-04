---
title: AWS Configure Site-to-Site VPN with GNS3
date: 2024-09-03 19:45:00 -0700
categories: [AWS, VPN Site-to-Site]
tags: [aws, vpn]     # TAG names should always be lowercase
---

**NOTE**: All configurations were taken from a lab environment.

This post explains step by step the configuration of a Site-to-Site VPN between AWS and a virtual Router running in GNS3 in my personal laptop.

**NOTE** For this to work, the NAT-T feature is required to be enabled on your Home modem.

Overall, these are the steps:

- Step 1 - AWS : Configure a Customer Gateway
- Step 2 - AWS : Configure a Virtual Private Gateway (VGW)
- Step 3 - AWS : Configure a Site-to-Site VPN connection
- Step 4 - AWS : Attach the VGW to a VPC
- Step 5 - AWS : Propagate the Routes in the Route Table
- Step 6 - GNS3 : Conect Router to the Internet
- Step 7 - GNS3 : Configure Router
- Step 8 - Validate connectivity using ICMP packets (ping test)

## Step 1 - AWS : Configure a Customer Gateway

![]({{ site.baseurl }}/images/services/vpc.png)

In the VPC section, go to **Virtual private network (VPN) > Customer gateways**

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/01-Customer-gateways.png)

Click on orange button **Create customer gateway** on the upper-right part of the screen.

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/02-Customer-gateway-configurations.png)

- **NOTE:** Obtain your Public IPv4 address from https://whatismyipaddress.com/

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/03-Customer-gateway-created.png)
