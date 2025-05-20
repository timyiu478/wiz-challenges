---
tags:
  - Kubernetes
  - Sidecar-Container
  - Istio
title: K8S LAN Party - Hello
description: TODO
reference: https://k8slanparty.com/challenge/2
---
## Challenge

Sometimes, it seems we are the only ones around, but we should always be on guard against invisible [sidecars](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) reporting sensitive secrets.
## Solution

### 1. discover other services

```
player@wiz-k8s-lan-party:~$ dnscan -subnet 10.100.0.0/16
43684 / 65536 [------------------------------------------------------------------------------->_______________________________________] 66.66% 994 p/s10.100.171.123 reporting-service.k8s-lan-party.svc.cluster.local.
65339 / 65536 [---------------------------------------------------------------------------------------------------------------------->] 99.70% 993 p/s10.100.171.123 -> reporting-service.k8s-lan-party.svc.cluster.local.
```

### 2. curl the reporting-service

The `report-service`'s response is from its sidecar proxy `istio-envoy`. We know this by looking at the HTTP header `server: istio-envoy`.

```
player@wiz-k8s-lan-party:/$ curl -v reporting-service.k8s-lan-party.svc.cluster.local/               
*   Trying 10.100.171.123:80...
* Connected to reporting-service.k8s-lan-party.svc.cluster.local (10.100.171.123) port 80 (#0)
> GET / HTTP/1.1
> Host: reporting-service.k8s-lan-party.svc.cluster.local
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< server: istio-envoy
< date: Tue, 20 May 2025 11:38:28 GMT
< content-type: text/plain
< x-envoy-upstream-service-time: 2
< x-envoy-decorator-operation: :0/*
< transfer-encoding: chunked
< 
* Connection #0 to host reporting-service.k8s-lan-party.svc.cluster.local left intact
```

### 3. port scanning the report-service

what is `bo2k`? https://www.f-secure.com/v-descs/bo2k.shtml

```
player@wiz-k8s-lan-party:~$ cat scan_report | grep 15151
15151/tcp open     bo2k
```
