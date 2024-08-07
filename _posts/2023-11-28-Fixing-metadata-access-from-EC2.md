---
title: Fixing Metadata access from EC2 instance
date: 2023-11-28 19:51:00 -0700
categories: [AWS, EC2]
tags: [aws, ec2]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/ec2.png)

Today, while doing a practice lab, I tried obtaining EC2 instance's metadata information by executing the cURL command to the AWS metadata address **169.254.169.254**. However, I got an **HTTP 401 Unauthorized error** message.

```bash
[ec2-user@ip-XXX-XX-XX-XX ~]$ curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone
<?xml version="1.0" encoding="iso-8859-1"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
 <head>
  <title>401 - Unauthorized</title>
 </head>
 <body>
  <h1>401 - Unauthorized</h1>
 </body>
</html>
[ec2-user@ip-XXX-XX-XX-XX ~]$ 
```

Googling to figure out the reason about of this **401 - Unauthorized** error message.

Documentation:

- [Retrieve instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html#instance-metadata-returns)

What needs to be done is obtain a Token first by sending an **HTTP PUT** to endpoint **http://169.254.169.254/latest/api/token**

Example:

```bash
[ec2-user@ip-XXX-XX-XX-XX ~]$ TOKEN=`curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
[ec2-user@ip-XXX-XX-XX-XX ~]$ echo $TOKEN
AQAAAJDk670nmCRRpfOAlYOP0Dtjo8dCGhyN4ETA5XZenliC6J88zg==
[ec2-user@ip-XXX-XX-XX-XX ~]$
```

Include the TOKEN from the previuos command subsequent calls to the Metadata API using `-H "X-aws-ec2-metadata-token: $TOKEN"`

For example:

```bash
[ec2-user@ip-XXX-XX-XX-XX ~]$ curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/placement/availability-zone
us-east-1d
```