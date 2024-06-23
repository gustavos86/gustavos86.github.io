---
title: VPC Private Connectivity
date: 2023-10-22 18:00:00 -0700
categories: [AWS, VPC Peering, VPC Endpoints, VPC PrivateLink]
tags: [aws, vpc peering, vpc endpoints, vpc privatelink]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/vpc.png)

## VPC Peering

VPC Peering connects two VPCs privately using AWS network

- It makes them behave as if they were in the same network
- Peered VPCs can be in the same AWS region or across AWS regions
- You can do VPC peering with another AWS account

Important:

- VPC CIDRs should not overlap!
- You must update route tables in each VPC's subnets to ensure instances can communicate across VPCs

One VPC is the **Requester VPC** and the other VPC is the **Accepter VPC**.

Note: Enabling DNS support for VPC peers in the peering connection allows the private IP usage to be forced, if applications always use the instance public DNS name.

VPC peering limitations:

- Must not have overlapping CIDRs
- VPC Peering connection is not transitive (must be established for each VPC that need to communicate with one another)
- You can setup only 1 VPC peering connection between 2 VPCs
- Maximum 125 VPC peering connections per VPC

- VPC-A <Peering connection> VPC-B <VPN/DX> Corporate. Traffic from VPC-A cannot cross VPC-B to reach Corporate in this scenario.
- VPC <Peering connection> VPC-B (IGW). Traffic from VPC-A cannot use VPC-B's IGW. Same if we replace IGW with NAT GW.
- VPC <Peering connection> VPC-B (Endpoints). Traffic from VPC-A cannot use VPC-B's endpoints to services such as S3 & DynamoDB

## VPC Endpoints

VPC Endpoints provide connectivity between VPC and AWS services.

- Endpoints allow you to connect to AWS Services using a private network instead of the public network
- They remove the need of IGW or NAT GW to access AWS Services
- Endpoint devices are horizontally scaled, redundant and highly available without any bandwidth constraint on your network traffic

  - **Gateway Endpoint:** provisions a target and must be used in a route table. Only **S3 and DynamoDB**
  - **Interface Endoint:** provisions an ENI (private IP) as an entry point - most other AWS services

### Gateway Endpoints

- Enables private connection between VPC and S3 or DynamoDB
- Need to modify the route tables and add an entry to route the traffic to S3 or DnamoDB through the gateway VPC endpoint
- When we create an Amazon S3 endpoint, a *prefix list* is created in VPC
- The prefix list is the collection of IP addresses that Amazon S3 uses
- The prefix list is formatted as *pl-xxxxxxxx* and becomes and available option in both subnet routing tables and Security Groups
- Prefix list should be added in Security Group Outbound rule (if Security group outbound rules do not have default "Allow All" rule)

VPC Endpoint Security

- Endpoint allows more granular access to VPC resources as compared to broad access through VPC peering connection
- Access to S3 through VPC endpoint can be secured using bucket policies and endpoint policies
- VPC Endpoint policy:
  - An IAM policy which is attached to VPC endpoint
  - Default policy allows full control to the AWS service

### Interface Endpoints

- Interface endpoints create local IP addresses (using ENI) in the VPC
- You create one interface endpoint per Availability Zone for high availability
- There is per hour cost (~$0.01/hr per AZ) and data processing cost (~$0.01 GB)
- Uses Security Groups - inbound rules
- For interface endpoints, AWS creates Regional and zonal DNS entries that resolves to private IP addresses of interface endpoints
- Interface endpoints support only IPv4 traffic
- Interface VPC endpoints support traffic only over TCP

## VPC PrivateLInk