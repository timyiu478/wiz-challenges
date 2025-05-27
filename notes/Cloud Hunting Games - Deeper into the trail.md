---
tags:
  - SQL
  - CloudTrail
title: Cloud Hunting Games - Deeper into the trail
description: Dig deeper about compromised machine for executing malicious code
reference: https://www.cloudhuntinggames.com/challenge/3
---


## Challenge

Bingo — you've tracked down the compromised IAM user: Moe.Jito. Keep digging through the CloudTrail logs.

Follow the attacker's footsteps and find the machine that was compromised and leveraged for lateral movement.

## Solution

### 1. get the logs where user name is the attacker

query:

```
SELECT * FROM cloudtrail WHERE userIdentity_userName = "Moe.Jito";
```

logs:

- they all came from the same IP - `109.236.81.188`
- IP Country - `Netherlands`

```
|   |   |   |   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
|2025-05-05T12:40:26Z|ListAttachedUserPolicies|109.236.81.188|iam.amazonaws.com|IAMUser|arn:aws:iam::509843726190:user/Moe.Jito|Moe.Jito|||Netherlands|TRUE|{"userName":"Moe.Jito"}||
|2025-05-05T12:41:39Z|ListRoles|109.236.81.188|iam.amazonaws.com|IAMUser|arn:aws:iam::509843726190:user/Moe.Jito|Moe.Jito|||Netherlands|TRUE|||
|2025-05-05T12:42:32Z|ListRoles|109.236.81.188|iam.amazonaws.com|IAMUser|arn:aws:iam::509843726190:user/Moe.Jito|Moe.Jito|||Netherlands|TRUE|||
|2025-05-05T12:42:43Z|ListRoles|109.236.81.188|iam.amazonaws.com|IAMUser|arn:aws:iam::509843726190:user/Moe.Jito|Moe.Jito|||Netherlands|TRUE|||
|2025-05-05T12:43:21Z|AssumeRole|109.236.81.188|sts.amazonaws.com|IAMUser|arn:aws:iam::509843726190:user/Moe.Jito|Moe.Jito|||Netherlands|TRUE|{"roleArn":"arn:aws:iam::509843726190:role/S3Reader","roleSessionName":"drinks"}|{"credentials":{"accessKeyId":"REDACTED","sessionToken":"REDACTED"REDACTED},"assumedRoleUser":{"arn":"arn:aws:sts::509843726190:assumed-role/S3Reader/drinks","assumedRoleId":"AROA6ODU25RADQW4NKKE4:drinks"}}|
```

### 2. filter the logs with one or more of the columns container `Moe.Joe`

found some lambda functions were used to create access key `Moe.Joe`

```
|   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|
|2025-05-03T15:41:13Z|DeleteAccessKey|aws-sdk-python/1.26.163 Python/3.9.11 Linux/5.10.192-162.733.amzn2.x86_64 exec-env/AWS_Lambda_python3.9 botocore/1.29.163|52.94.76.114|iam.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/credsrotator-role-g67nmnfk/credsrotator||||
|2025-05-03T15:41:13Z|CreateAccessKey|aws-sdk-python/1.26.163 Python/3.9.11 Linux/5.10.192-162.733.amzn2.x86_64 exec-env/AWS_Lambda_python3.9 botocore/1.29.163|52.94.76.114|iam.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/credsrotator-role-g67nmnfk/credsrotator||||
|2025-05-03T17:41:12Z|CreateAccessKey|aws-sdk-python/1.26.163 Python/3.9.11 Linux/5.10.192-162.733.amzn2.x86_64 exec-env/AWS_Lambda_python3.9 botocore/1.29.163|52.94.76.114|iam.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/credsrotator-role-g67nmnfk/credsrotator||||
|2025-05-03T17:41:12Z|DeleteAccessKey|aws-sdk-python/1.26.163 Python/3.9.11 Linux/5.10.192-162.733.amzn2.x86_64 exec-env/AWS_Lambda_python3.9 botocore/1.29.163|52.94.76.114|iam.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/credsrotator-role-g67nmnfk/credsrotator||||
|2025-05-03T19:41:11Z|CreateAccessKey|aws-sdk-python/1.26.163 Python/3.9.11 Linux/5.10.192-162.733.amzn2.x86_64 exec-env/AWS_Lambda_python3.9 botocore/1.29.163|52.94.76.114|iam.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/credsrotator-role-g67nmnfk/credsrotator||||
|2025-05-03T19:41:11Z|DeleteAccessKey|aws-sdk-python/1.26.163 Python/3.9.11 Linux/5.10.192-162.733.amzn2.x86_64 exec-env/AWS_Lambda_python3.9 botocore/1.29.163|52.94.76.114|iam.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/credsrotator-role-g67nmnfk/credsrotator||||
|2025-05-03T21:41:11Z|CreateAccessKey|aws-sdk-python/1.26.163 Python/3.9.11 Linux/5.10.192-162.733.amzn2.x86_64 exec-env/AWS_Lambda_python3.9 botocore/1.29.163|52.94.76.114|iam.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/credsrotator-role-g67nmnfk/credsrotator||||
|2025-05-03T21:41:12Z|DeleteAccessKey|aws-sdk-python/1.26.163 Python/3.9.11 Linux/5.10.192-162.733.amzn2.x86_64 exec-env/AWS_Lambda_python3.9 botocore/1.29.163|52.94.76.114|iam.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/credsrotator-role-g67nmnfk/credsrotator||||
|2025-05-03T23:41:13Z|CreateAccessKey|aws-sdk-python/1.26.163 Python/3.9.11 Linux/5.10.192-162.733.amzn2.x86_64 exec-env/AWS_Lambda_python3.9 botocore/1.29.163|52.94.76.114|iam.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/credsrotator-role-g67nmnfk/credsrotator||||
|2025-05-03T23:41:13Z|DeleteAccessKey|aws-sdk-python/1.26.163 Python/3.9.11 Linux/5.10.192-162.733.amzn2.x86_64 exec-env/AWS_Lambda_python3.9 botocore/1.29.163|52.94.76.114|iam.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/credsrotator-role-g67nmnfk/credsrotator||||
|2025-05-04T01:41:12Z|DeleteAccessKey|aws-sdk-python/1.26.163 Python/3.9.11 Linux/5.10.192-162.733.amzn2.x86_64 exec-env/AWS_Lambda_python3.9 botocore/1.29.163|52.94.76.114|iam.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/credsrotator-role-g67nmnfk/credsrotator||||
|2025-05-04T01:41:12Z|CreateAccessKey|aws-sdk-python/1.26.163 Python/3.9.11 Linux/5.10.192-162.733.amzn2.x86_64 exec-env/AWS_Lambda_python3.9 botocore/1.29.163|52.94.76.114|iam.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/credsrotator-role-g67nmnfk/credsrotator||||
```


### 3. find the logs that the user identity arn is `lambda`

query:

```
SELECT * FROM cloudtrail where userIdentity_ARN LIKE "%lambda%";
```

logs:

- found a log about `lambda.update-function-code` and the machine id is `i-0a44002eec2f16c25`

```
|   |   |   |   |   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|2025-05-05T12:28:23Z|ListFunctions20150331|aws-cli/2.25.11 md/awscrt#0.25.4 ua/2.1 os/linux#6.8.0-1024-aws md/arch#x86_64 lang/python#3.12.9 md/pyimpl#CPython cfg/retry-mode#standard md/installer#exe md/distrib#ubuntu.24 md/prompt#off md/command#lambda.List-functions|54.227.18.169|lambda.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/lambdaWorker/i-0a44002eec2f16c25||||United States|TRUE|||
|2025-05-05T12:32:42Z|UpdateFunctionCode20150331v2|aws-cli/2.25.11 md/awscrt#0.25.4 ua/2.1 os/linux#6.8.0-1024-aws md/arch#x86_64 lang/python#3.12.9 md/pyimpl#CPython cfg/retry-mode#standard md/installer#exe md/distrib#ubuntu.24 md/prompt#off md/command#lambda.update-function-code|54.227.18.169|lambda.amazonaws.com|AssumedRole|arn:aws:sts::509843726190:assumed-role/lambdaWorker/i-0a44002eec2f16c25||||United States|FALSE|{"functionName":"credsrotator","dryRun":false,"publish":false}|{"state":"Active","version":"$LATEST","description":"","role":"arn:aws:iam::509843726190:role/service-role/credsrotator-role-g67nmnfk","functionName":"credsrotator","functionArn":"arn:aws:lambda:us-east-1:509843726190:function:credsrotator","runtime":"python3.12","handler":"lambda_function.lambda_handler","codeSize":1506,"timeout":30,"memorySize":128,"lastModified":"2025-04-09T12:32:42.0000000Z","codeSha256":"xMWbzIHcmJhPRW+la07ozhSi4CDvyW3QyxgXDIa5VZw=","environment":{},"tracingConfig":{"mode":"PassThrough"},"revisionId":"9c041553-2fbe-40ea-a533-6da6d0d39ee1","lastUpdateStatus":"InProgress","lastUpdateStatusReason":"The function is being created.","lastUpdateStatusReasonCode":"Creating","packageType":"Zip","architectures":["x86_64"],"ephemeralStorage":{"size":512},"snapStart":{"applyOn":"None","optimizationStatus":"Off"},"runtimeVersionConfig":{"runtimeVersionArn":"arn:aws:lambda:us-east-1::runtime:40d617d2be93996819b6c44f30965daf9adf07d68603c070365dbdc7e1d8db16"},"loggingConfig":{"logFormat":"Text","logGroup":"/aws/lambda/credsrotator"}}|
```