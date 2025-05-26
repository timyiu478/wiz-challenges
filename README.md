# wiz-challenges

## About

This repository contains write-ups for various cloud security challenges, including EKS Cluster Game, Kubernetes LAN Party, and the BIG IAM Challenge. The challenges are designed to learn about common misconfigurations and security issues in cloud environments, particularly in Kubernetes and AWS IAM.

## Notes

| Title                                                                                                                                                                                 | Description                                                              | Tags                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | -------------------------------------- |
| [EKS Cluster Game - Registry Hunt](notes/EKS%20Cluster%20Game%20-%20Registry%20Hunt.md)                                                                                               | imagePullSecret is not actually a secret                                 | Kubernetes, Container-Registry         |
| [EKS Cluster Game - Secret Seeker](notes/EKS%20Cluster%20Game%20-%20Secret%20Seeker.md)                                                                                               | list all the secrets in the k8s cluster                                  | Kubernetes, Secret                     |
| [EKS Cluster Game - Image Inquisition](./notes/EKS%20Cluster%20Game%20-%20Image%20Inquisition.md)                                                                                     | Accessing the Instance Metadata service from a compromised pod           | Kubernetes, Instance Metadata service  |
| [EKS Cluster Game - Pod Break](./notes/EKS%20Cluster%20Game%20-%20Pod%20Break.md)                                                                                                     | From node to k8s API                                                     | Kubernetes, Instance Metadata service  |
| [EKS Cluster Game - Container Secrets Infrastructure](https://github.com/timyiu478/wiz-challenges/blob/main/notes/EKS%20Cluster%20Game%20-%20Container%20Secrets%20Infrastructure.md) | Move from the EKS to the AWS account                                     | Kubernetes, OIDC                       |
| [K8S LAN Party - DNSING with the stars](notes/K8S%20LAN%20Party%20-%20DNSING%20with%20the%20stars.md)                                                                                 | Service discovery using dns scan                                         | Kubernetes, DNS, Service Discovery     |
| [K8S LAN Party - Hello](notes/K8S%20LAN%20Party%20-%20Hello.md)                                                                                                                       | Capture secrets from sidecar container via packet sniffing               | Kubernetes, tcpdump                    |
| [K8S LAN Party - Exposed File Share ](notes/K8S%20LAN%20Party%20-%20Exposed%20File%20Share.md)                                                                                        | The problem of old-school network file shares in the cloud               | Kubernetes, EFS                        |
| [K8S LAN Party - The Beauty and The Ist](notes/K8S%20LAN%20Party%20-%20The%20Beauty%20and%20The%20Ist.md)                                                                             | Bypass Istio envoy sidecar for ignoring the authorization policy         | Kubernetes, Istio                      |
| [K8S LAN Party - Who will guard the guardians](notes/K8S%20LAN%20Party%20-%20Who%20will%20guard%20the%20guardians.md)                                                                 | Mistake from guardian                                                    | Kubernetes, Kyverno, Admission Webhook |
| [The BIG IAM Challenge - Buckets of Fun](notes/The%20BIG%20IAM%20Challenge%20-%20Buckets%20of%20Fun.md)                                                                               | Public buckets are risky                                                 | IAM, S3                                |
| [The BIG IAM Challenge - Google Analytics](notes/The%20BIG%20IAM%20Challenge%20-%20Google%20Analytics.md)                                                                             | Public queues are risky                                                  | IAM, SQS                               |
| [The BIG IAM Challenge - Enable Push Notifications](notes/The%20BIG%20IAM%20Challenge%20-%20Google%20Analytics.md)                                                                    | Asterisks are risky                                                      | IAM, SNS                               |
| [THE BIG IAM Challenge - Admin Only](notes/THE%20BIG%20IAM%20Challenge%20-%20Admin%20Only.md)                                                                                         | ForAllValues:StringLike may not be what you think                        | IAM, S3                                |
| [The BIG IAM Challenge - Do I Know You](notes/The%20BIG%20IAM%20Challenge%20-%20Do%20I%20Know%20You.md)                                                                               | Be aware how identity pool generate AWS credentials for the guest access | IAM, Cognito                           |
| [THE BIG IAM Challenge - One final push](notes/THE%20BIG%20IAM%20Challenge%20-%20One%20final%20push)                                                                                  | IAM is too open to OIDC                                                  | IAM, Cognito, OIDC                     |

## Resources

### EKS Cluster Game

> The challenge consists of five different scenarios, each one focusing on a possible Amazon EKS issue — and we’ve already directly observed some of them in various research engagements. Participants will play as the attacker, learn about these misconfigurations and security issues, and then exploit them in a controlled environment.

Link: https://eksclustergames.com/

### Kubernetes Lan Party

> A CTF designed to challenge your Kubernetes hacking skills through a series of critical network vulnerabilities and misconfigurations. Challenge yourself, boost your skills, and stay ahead in the cloud security game.

Link: https://k8slanparty.com/

### BIG IAM Challenge

> This challenge is open to everyone - from beginners seeking to learn more about [IAM security](https://www.wiz.io/academy/iam-security) configurations to experienced professionals wanting to brush up on their skills. No special software, no complex set-ups - all you need is the AWS Command Line Interface (CLI), which is already integrated into the challenge's website. The challenge consists of 6 steps, with each one focusing on a common IAM configuration mistake in various AWS services. You will have the opportunity to identify and exploit these errors while applying your knowledge in real-world scenarios.

Link: https://bigiamchallenge.com/challenge/
### Others

- https://www.cloudhuntinggames.com/
- https://promptairlines.com/
