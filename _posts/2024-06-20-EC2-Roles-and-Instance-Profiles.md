---
title: EC2 Roles and Instance Profiles
date: 2024-06-20 20:42:00 -0700
categories: [AWS, IAM, EC2]
tags: [aws, iam, ec2]     # TAG names should always be lowercase
---

# Introduction

Amazon EC2 instances should be able to securely access other AWS services. Credentials need to be rotated regularly to minimize the adverse impact of a security breach.

- **AWS IAM roles** provide the ability to automatically grant instances temporary credentials without the need for manual management.
- **IAM instance profiles** provide the mechanism to attach IAM roles to EC2 instances.

# Configuration via AWS CLI

## 1. Create IAM Policy

```
```

## 2. Associate the IAM Policy with an IAM Role

```
```

## 3. Attach the IAM Role to the EC2 instance

```
```

## 4. Validation

```
```


# Configuration using the AWS Management Console

## 1. Create IAM Policy

```
```

## 2. Associate the IAM Policy with an IAM Role

```
```

## 3. Attach the IAM Role to the EC2 instance

```
```

## 4. Validation

```
```
