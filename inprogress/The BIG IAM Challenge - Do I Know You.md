---
tags:
  - IAM
  - Cognito
  - S3
title: The BIG IAM Challenge - Do I Know You
description: 
reference: https://thebigiamchallenge.com/challenge/5
---

## Challenge

We configured AWS Cognito as our main identity provider. Let's hope we didn't make any mistakes.

## Solution

### 1. View the provided IAM Policy

```json
{ "Version": "2012-10-17", "Statement": [ { "Sid": "VisualEditor0", "Effect": "Allow", "Action": [ "mobileanalytics:PutEvents", "cognito-sync:*" ], "Resource": "*" }, { "Sid": "VisualEditor1", "Effect": "Allow", "Action": [ "s3:GetObject", "s3:ListBucket" ], "Resource": [ "arn:aws:s3:::wiz-privatefiles", "arn:aws:s3:::wiz-privatefiles/*" ] } ] }
```


