---
title: Fixing Metadata access from EC2 instance
date: 2023-11-28 19:51:00 -0700
categories: [AWS, EC2]
tags: [aws, ec2]     # TAG names should always be lowercase
---

Today, while doing a practice lab I tried obtaining metadata information for a EC2 instance by executing CURL to the AWS metadata address 169.254.169.254. However, I got an **HTTP 401 error**.

```
[ec2-user@ip-172-31-30-20 ~]$ curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone
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
[ec2-user@ip-172-31-30-20 ~]$ 
```

A google search helped me to get the rason about of this.

Documentation:

- [Retrieve instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html#instance-metadata-returns)

It seems I have to obtain a Token first. Example:

```
[ec2-user@ip-172-31-30-20 ~]$ TOKEN=`curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
[ec2-user@ip-172-31-30-20 ~]$ echo $TOKEN
AQAAAJDk670nmCRRpfOAlYOP0Dtjo8dCGhyN4ETA5XZenliC6J88zg==
[ec2-user@ip-172-31-30-20 ~]$
```

I have to include the TOKEN to calls to the Metadata API using `-H "X-aws-ec2-metadata-token: $TOKEN"`

For example:

```
[ec2-user@ip-172-31-30-20 ~]$ curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/placement/availability-zone
us-east-1d
```