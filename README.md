# wiz-challenges

## About

This repository contains various challenges and resources related to Kubernetes, cloud security, and container registries. Use these challenges to practice and improve cloud security skills.

## Notes

| Title                                                                                                                                                                                 | Description                                                      | Tags                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- | -------------------------------------- |
| [EKS Cluster Game - Registry Hunt](notes/EKS%20Cluster%20Game%20-%20Registry%20Hunt.md)                                                                                               | imagePullSecret is not actually a secret                         | Kubernetes, Container-Registry         |
| [EKS Cluster Game - Secret Seeker](notes/EKS%20Cluster%20Game%20-%20Secret%20Seeker.md)                                                                                               | list all the secrets in the k8s cluster                          | Kubernetes, Secret                     |
| [EKS Cluster Game - Image Inquisition](./notes/EKS%20Cluster%20Game%20-%20Image%20Inquisition.md)                                                                                     | Accessing the Instance Metadata service from a compromised pod   | Kubernetes, Instance Metadata service  |
| [EKS Cluster Game - Pod Break](./notes/EKS%20Cluster%20Game%20-%20Pod%20Break.md)                                                                                                     | From node to k8s API                                             | Kubernetes, Instance Metadata service  |
| [EKS Cluster Game - Container Secrets Infrastructure](https://github.com/timyiu478/wiz-challenges/blob/main/notes/EKS%20Cluster%20Game%20-%20Container%20Secrets%20Infrastructure.md) | Move from the EKS to the AWS account                             | Kubernetes, OIDC                       |
| [K8S LAN Party - DNSING with the stars](notes/K8S%20LAN%20Party%20-%20DNSING%20with%20the%20stars.md)                                                                                 | Service discovery using dns scan                                 | Kubernetes, DNS, Service Discovery     |
| [K8S LAN Party - Hello](notes/K8S%20LAN%20Party%20-%20Hello.md)                                                                                                                       | Capture secrets from sidecar container via packet sniffing       | Kubernetes, tcpdump                    |
| [K8S LAN Party - Exposed File Share ](notes/K8S%20LAN%20Party%20-%20Exposed%20File%20Share.md)                                                                                        | The problem of old-school network file shares in the cloud       | Kubernetes, EFS                        |
| [K8S LAN Party - The Beauty and The Ist](notes/K8S%20LAN%20Party%20-%20The%20Beauty%20and%20The%20Ist.md)                                                                             | Bypass Istio envoy sidecar for ignoring the authorization policy | Kubernetes, Istio                      |
| [K8S LAN Party - Who will guard the guardians](notes/K8S%20LAN%20Party%20-%20Who%20will%20guard%20the%20guardians.md)                                                                 | Mistake from guardian                                            | Kubernetes, Kyverno, Admission Webhook |
| [The BIG IAM Challenge - Buckets of Fun](notes/The%20BIG%20IAM%20Challenge%20-%20Buckets%20of%20Fun.md)                                                                               | Public buckets are risky                                         | IAM, S3                                |
| [The BIG IAM Challenge - Google Analytics](notes/The%20BIG%20IAM%20Challenge%20-%20Google%20Analytics.md)                                                                             | Public queues are risky                                          | IAM, SQS                               |
| [The BIG IAM Challenge - Enable Push Notifications](notes/The%20BIG%20IAM%20Challenge%20-%20Google%20Analytics.md)                                                                    | Asterisks are risky                                              | IAM, SNS                               |


## Resources

### EKS Cluster Game

> The challenge consists of five different scenarios, each one focusing on a possible Amazon EKS issue — and we’ve already directly observed some of them in various research engagements. Participants will play as the attacker, learn about these misconfigurations and security issues, and then exploit them in a controlled environment.

Link: https://eksclustergames.com/

### Kubernetes Lan Party

> A CTF designed to challenge your Kubernetes hacking skills through a series of critical network vulnerabilities and misconfigurations. Challenge yourself, boost your skills, and stay ahead in the cloud security game.

Link: https://k8slanparty.com/

### Others

- https://www.cloudhuntinggames.com/
- https://promptairlines.com/
- https://bigiamchallenge.com/challenge/1
