---
tags:
  - S3
  - SQL
title: Cloud Hunting Games - Ain't no data when she's gone
description: Verify the attacker whether indeed obtained the secrets via logs
reference: https://www.cloudhuntinggames.com/challenge/1
---

## Challenge

FizzShadows claim that they were able to exfiltrate ExfilCola's secret recipes. You have to validate this claim before considering any further steps.

All ExfilCola's secret recipes are stored in a secured S3 bucket. Luckily, their security team was responsible enough to make sure that S3 data events were collected. You've been granted access to these logs, which are available in the s3_data_events table.

Go ahead and see if there are any traces of exfiltration: find the IAM role involved in the attack.

## Solution

### 1. run `SELECT * FROM s3_data_events WHERE PATH LIKE '%exfilcola%';` SQL query

found a log that the path is `ExfilCola-Top-Secret.txt` and the error code is success.

```
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|2025-05-05T12:48:00Z|GetObject|[Boto3/1.34.69 md/Botocore#1.34.69 ua/2.0 os/macos#23.6.0 md/arch#arm64 lang/python#3.12.2 md/pyimpl#CPython cfg/retry-mode#legacy Botocore/1.34.69]|37.19.199.135|arn:aws:sts::509843726190:assumed-role/S3Reader/drinks|s3.amazonaws.com|AssumedRole|||United States|{"bucketName":"soda-vault","Host":"soda-vault.s3.amazonaws.com","key":"Pwnsi.txt"}|["/ExfilCola-Top-Secret.txt"]||3395|
```





