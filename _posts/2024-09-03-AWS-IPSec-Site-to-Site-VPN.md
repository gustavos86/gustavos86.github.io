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

- Step 1 - AWS : Configure a Customer Gateway (CGW)
- Step 2 - AWS : Configure a Virtual Private Gateway (VGW)
- Step 3 - AWS : Attach the VGW to a VPC
- Step 4 - AWS : Configure a Site-to-Site VPN connection
- Step 5 - AWS : Propagate the Routes in the Route Table
- Step 6 - GNS3 : Conect Router to the Internet
- Step 7 - GNS3 : Configure Router
- Step 8 - Validate connectivity using ICMP packets (ping test)

## Step 1 - AWS : Configure a Customer Gateway (CGW)

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

Finally, click on **Attach to VPC**.

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/09-VGW-in-the-process-of-being-attached-to-VPC.png)

It should take a moment for the process to complete. You need to refresh the screen and see the **State** column showing **Attached**

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/10-VGW-attached-to-VPC.png)

## Step 4 - AWS : Configure a Site-to-Site VPN connection

Go next to **Virtual private network (VPN) > Site-to-Site VPN connections**

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/11-Site-to-Site-VPN-connections.png)

Click on orange button **Create VPN connection** on the upper-right part of the screen.

Select the **Virtual Private Gateway (VGW)** and **Customer Gateway (CGW)** created previously.

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/12-Create-VPN-connection-part1.png)

We are also adding an **Inside IPv4 CIDR** which is the Tunnel Point-to-Point IP addressing and **Pre-shared key**. These are optional but we are using them here to make the template configuration for the Cisco Router easier to read.

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/13-Create-VPN-connection-part2.png)

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/14-Create-VPN-connection-part3.png)

Click on the orange button **Create VPN connection**

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/15-Site-to-Site-VPN-connection-created.png)

**NOTE:** Like in several AWS configuration, the **Name** of this VPN connection is a customer friendly *tag* which we ommited in this lab.

The VPN connection takes a few minutes to get created. Once we refresh the screen, we should see the **State** column showing ****

Click on the newly created VPN. On the button section of the screen click-on **Tunnel details**. The IP addresses displayed in column **Outside IP address** are the AWS Public IP addresses to be used as the IPSec endpoints.

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/16-Site-to-Site-VPN-Tunnel-details.png)

## Step 5 - AWS : Propagate the Routes in the Route Table

Finally, go to **Virtual private cloud > Route tables**

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/17-Route-tables.png)

Select the Route Table and click on the **Rotute propagation** tab

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/18-Route-propagation.png)

Enable the Route propagation checkbox and click **Save**

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/19-Route-propagation-enable.png)

## Step 6 - GNS3 : Conect Router to the Internet

I already have a **GNS3** setup running with the the GNS3 VM (installation tutorial in the References section) and VMware® Workstation 17 Pro.

What I am using is a Cisco Router 7200 connected to a **NAT Cloud**

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/20-GNS3-Router-to-NAT.png)

The first step is configure the Router to receive an IP Address via DHCP

```
configure terminal
!
interface GigabitEthernet0/0
 ip address dhcp
 no shutdown
!
exit
```

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/21-Router-dhcp-client.png)

We must see the Routing table now having a Default Route installed installed in the Routing Table

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/22-Router-default-route.png)

The Router must be able to ping a Host on the Internet, in this case we are pinging the well-known Google DNS server **8.8.8.8**

![]({{ site.baseurl }}/images/2024/09-03-AWS-IPSec-Site-to-Site-VPN/23-Router-ping.png)

## References

- [How AWS Site-to-Site VPN works](https://docs.aws.amazon.com/vpn/latest/s2svpn/how_it_works.html)
- [AWS Advanced Networking – Part 2](https://roborndoff.com/2019/04/05/aws-advanced-networking-part-2/index.html)
- [GNS3 Setup wizard with the GNS3 VM](https://docs.gns3.com/docs/getting-started/setup-wizard-gns3-vm/)