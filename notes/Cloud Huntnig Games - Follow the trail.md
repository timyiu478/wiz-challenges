---
tags:
  - CloudTrail
  - SQL
title: Cloud Huntnig Games - Follow, follow the trail
description: Use the role name and session name to find who assumes this role
reference: https://www.cloudhuntinggames.com/challenge/2
---

## Challenge

So you have managed to validate FizzShadows' claim and track the IAM role that has exfiltrated ExfilCola's recipe. You've been granted access to the cloudtrail table.

Follow the trail of the S3Reader. Who used it?

## Solution

### 1. find the logs that the event name is assume role and the role request contains the role name `S3Reader` and the session name `drinks`

The query:

```
SELECT * FROM cloudtrail WHERE EventName = "AssumeRole" and requestParameters LIKE "%S3Reader%drinks%"; 
```

Result:

```
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|2025-05-05T12:43:21Z|AssumeRole|aws-cli/2.23.2 md/awscrt#0.23.4 ua/2.0 os/macos#23.6.0 md/arch#arm64 lang/python#3.12.8 md/pyimpl#CPython cfg/retry-mode#standard md/installer#source md/prompt#off md/command#sts.assume-role|109.236.81.188|sts.amazonaws.com|IAMUser|arn:aws:iam::509843726190:user/Moe.Jito|Moe.Jito|||Netherlands|TRUE|{"roleArn":"arn:aws:iam::509843726190:role/S3Reader","roleSessionName":"drinks"}|{"credentials":{"accessKeyId":"REDACTED","sessionToken":"REDACTED"REDACTED},"assumedRoleUser":{"arn":"arn:aws:sts::509843726190:assumed-role/S3Reader/drinks","assumedRoleId":"AROA6ODU25RADQW4NKKE4:drinks"}}|
```