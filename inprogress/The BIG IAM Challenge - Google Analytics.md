---
tags:
  - IAM
  - Google-Analytics
title: The BIG IAM Challenge - Google Analytics
description: 
reference: https://thebigiamchallenge.com/challenge/2
---

# Challenge

We created our own analytics system specifically for this challenge. We think it's so good that we even used it on this page. What could go wrong?

Join our queue and get the secret flag.

# Solution
## 1. View the provided IAM Policy

```json
{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Principal": "*", "Action": [ "sqs:SendMessage", "sqs:ReceiveMessage" ], "Resource": "arn:aws:sqs:us-east-1:092297851374:wiz-tbic-analytics-sqs-queue-ca7a1b2" } ] }
```


