---
tags:
  - IAM
  - Cognito
title: THE BIG IAM Challenge - One final push
description: 
reference: https://thebigiamchallenge.com/challenge/6
---
## Challenge

Anonymous access no more. Let's see what can you do now.

Now try it with the authenticated role: _arn:aws:iam::092297851374:role/Cognito_s3accessAuth_Role_

## Solution

### 1. View the provided IAM Policy

```json
{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Principal": { "Federated": "cognito-identity.amazonaws.com" }, "Action": "sts:AssumeRoleWithWebIdentity", "Condition": { "StringEquals": { "cognito-identity.amazonaws.com:aud": "us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b" } } } ] }
```



