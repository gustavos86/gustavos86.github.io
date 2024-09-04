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
- Step 3 - AWS : Attach the VGW to a VPC
- Step 4 - AWS : Configure a Site-to-Site VPN connection
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

## Step 2 - AWS : Configure a Virtual Private Gateway (VGW)

![]({{ site.baseurl }}/images/services/vpc.png)

Still in the VPC section, go to **Virtual private network (VPN) > Virtual private gateways**

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/04-Virtual-private-gateways.png)

Click on orange button **Create virtual private gateway** on the upper-right part of the screen.

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/05-Virtual-private-gateways-configuration.png)

- **NOTE:** The Amazon default ASN is **64512**

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/06-Virtual-private-gateway-created.png)

## Step 3 - AWS : Attach the VGW to a VPC

While still in **Virtual private network (VPN) > Virtual private gateways**, click on the newly created VGW

On the upper-right side of the screen, click on **Actions > Attach to VPC**

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/07-VGW-Attach-to-VPC.png)

And select the desired VPC from the list of Available VPCs. In this case I am choosing the **Default VPC**

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/08-VGW-Attach-to-VPC-select-the-VPC.png)

## References

- [How AWS Site-to-Site VPN works](https://docs.aws.amazon.com/vpn/latest/s2svpn/how_it_works.html)
- [AWS Advanced Networking – Part 2](https://roborndoff.com/2019/04/05/aws-advanced-networking-part-2/index.html)