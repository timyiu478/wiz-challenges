# wiz-challenges

## About

This repository contains write-ups for various cloud security challenges, including EKS Cluster Game, Kubernetes LAN Party, and the BIG IAM Challenge. The challenges are designed to learn about common misconfigurations and security issues in cloud environments, particularly in Kubernetes and AWS IAM.

## Notes

| #   | Title                                                                                                                                                                                                     | Description                                                              | Tags                                   |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | -------------------------------------- |
| 1   | [EKS Cluster Game - Registry Hunt](notes/EKS%20Cluster%20Game%20-%20Registry%20Hunt.md)                                                                                                                   | imagePullSecret is not actually a secret                                 | Kubernetes, Container-Registry         |
| 2   | [EKS Cluster Game - Secret Seeker](notes/EKS%20Cluster%20Game%20-%20Secret%20Seeker.md)                                                                                                                   | list all the secrets in the k8s cluster                                  | Kubernetes, Secret                     |
| 3   | [EKS Cluster Game - Image Inquisition](./notes/EKS%20Cluster%20Game%20-%20Image%20Inquisition.md)                                                                                                         | Accessing the Instance Metadata service from a compromised pod           | Kubernetes, Instance Metadata service  |
| 3   | [EKS Cluster Game - Pod Break](./notes/EKS%20Cluster%20Game%20-%20Pod%20Break.md)                                                                                                                         | From node to k8s API                                                     | Kubernetes, Instance Metadata service  |
| 4   | [EKS Cluster Game - Container Secrets Infrastructure](https://github.com/timyiu478/wiz-challenges/blob/main/notes/EKS%20Cluster%20Game%20-%20Container%20Secrets%20Infrastructure.md)                     | Move from the EKS to the AWS account                                     | Kubernetes, OIDC                       |
| 5   | [K8S LAN Party - DNSING with the stars](notes/K8S%20LAN%20Party%20-%20DNSING%20with%20the%20stars.md)                                                                                                     | Service discovery using dns scan                                         | Kubernetes, DNS, Service Discovery     |
| 6   | [K8S LAN Party - Hello](notes/K8S%20LAN%20Party%20-%20Hello.md)                                                                                                                                           | Capture secrets from sidecar container via packet sniffing               | Kubernetes, tcpdump                    |
| 7   | [K8S LAN Party - Exposed File Share ](notes/K8S%20LAN%20Party%20-%20Exposed%20File%20Share.md)                                                                                                            | The problem of old-school network file shares in the cloud               | Kubernetes, EFS                        |
| 8   | [K8S LAN Party - The Beauty and The Ist](notes/K8S%20LAN%20Party%20-%20The%20Beauty%20and%20The%20Ist.md)                                                                                                 | Bypass Istio envoy sidecar for ignoring the authorization policy         | Kubernetes, Istio                      |
| 9   | [K8S LAN Party - Who will guard the guardians](notes/K8S%20LAN%20Party%20-%20Who%20will%20guard%20the%20guardians.md)                                                                                     | Mistake from guardian                                                    | Kubernetes, Kyverno, Admission Webhook |
| 10  | [The BIG IAM Challenge - Buckets of Fun](notes/The%20BIG%20IAM%20Challenge%20-%20Buckets%20of%20Fun.md)                                                                                                   | Public buckets are risky                                                 | IAM, S3                                |
| 11  | [The BIG IAM Challenge - Google Analytics](notes/The%20BIG%20IAM%20Challenge%20-%20Google%20Analytics.md)                                                                                                 | Public queues are risky                                                  | IAM, SQS                               |
| 12  | [The BIG IAM Challenge - Enable Push Notifications](notes/The%20BIG%20IAM%20Challenge%20-%20Google%20Analytics.md)                                                                                        | Asterisks are risky                                                      | IAM, SNS                               |
| 13  | [THE BIG IAM Challenge - Admin Only](notes/THE%20BIG%20IAM%20Challenge%20-%20Admin%20Only.md)                                                                                                             | ForAllValues:StringLike may not be what you think                        | IAM, S3                                |
| 14  | [The BIG IAM Challenge - Do I Know You](notes/The%20BIG%20IAM%20Challenge%20-%20Do%20I%20Know%20You.md)                                                                                                   | Be aware how identity pool generate AWS credentials for the guest access | IAM, Cognito                           |
| 15  | [THE BIG IAM Challenge - One final push](notes/THE%20BIG%20IAM%20Challenge%20-%20One%20final%20push)                                                                                                      | IAM is too open to OIDC                                                  | IAM, Cognito, OIDC                     |
| 16  | [Cloud Hunting Games - Ain't no data when she's gone](notes/Cloud%20Hunting%20Games%20-%20Ain't%20no%20data%20when%20she's%20gone.md)                                                                     | Verify the attacker whether indeed obtained the secrets via logs         | S3, SQL                                |
| 17  | [Cloud Huntnig Games - Follow, follow the trail](notes/Cloud%20Huntnig%20Games%20-%20Follow%20the%20trail.md)                                                                                             | Use the role name and session name to find who assumes this role         | CloudTrail, SQL                        |
| 18  | [Cloud Hunting Games - Deeper into the trail](notes/Cloud%20Hunting%20Games%20-%20Deeper%20into%20the%20trail.md)                                                                                         | Dig deeper about compromised machine for executing malicious code        | CloudTrail, SQL                        |
| 19  | [Cloud Hunting Games - Aint no mountain high enough to keep me away from my logs](notes/Cloud%20Hunting%20Games%20-%20Aint%20no%20mountain%20high%20enough%20to%20keep%20me%20away%20from%20my%20logs.md) | Reveal the attacker ssh auth logs                                        | OverlayFS, EC2                         |
| 20  | [Cloud Hunting Games - Now you're just somebody that I used to log](notes/Cloud%20Hunting%20Games%20-%20Now%20you're%20just%20somebody%20that%20I%20used%20to%20log.md)                                   | Cloud version morris worm                                                | Bash, Cron, Self-Duplicaiton           |
| 21  | [AI Security Challenge - 1](notes/AI%20Security%20Challenge%20-%201.md)                                                                                                                                   | Overwrite LLM Pre-prompts                                                | AI, LLM                                |
| 22  | [AI Security Challenge - 2](notes/AI%20Security%20Challenge%20-%202.md)                                                                                                                                   | New verision of AI with backup ability                                   | AI, LLM                                |
| 23  | [AI Security Challenge - 3](notes/AI%20Security%20Challenge%20-%203.md)                                                                                                                                   | Backup is powerful                                                       | AI, LLM                                |


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

### The Cloud Hunting Games

> ExfilCola, a promising startup with a revolutionary soda formula that could break the duopoly of the cola giants, has received a data extortion email from a threat group known as "FizzShadows", claiming to have breached their systems. If the secret recipe is stolen, the future of the company is at immediate risk. As the chosen expert, you must prevent the formula from being exposedbefore it's too late.

Link: https://www.cloudhuntinggames.com/

### AI Security Challenge

> In this challenge, you'll interact with a chatbot acting as the customer service for the fictional Prompt Airlines. Your goal is to find and exploit vulnerabilities in the AI's logic to **trick it into giving you a free flight ticket**. The entire challenge is conducted through chatbot interaction - no coding or technical skills required! Each stage of the challenge highlights a different AI security risk that could impact real businesses. It's a hands-on opportunity to learn about securing AI systems against malicious attacks.

 Link: https://promptairlines.com/
