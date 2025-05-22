---
tags:
  - Kubernetes
  - Sidecar-Container
  - tcpdump
title: K8S LAN Party - Hello
description: Capture secrets from sidecar container via packet sniffing
reference: https://k8slanparty.com/challenge/2
---

## Challenge

Sometimes, it seems we are the only ones around, but we should always be on guard against invisible [sidecars](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) reporting sensitive secrets.
## Solution

![](/attachments/capture_sidecar_container_data_from_the_network.png)

### 1. discover other services

```
player@wiz-k8s-lan-party:~$ dnscan -subnet 10.100.0.0/16
43684 / 65536 [------------------------------------------------------------------------------->_______________________________________] 66.66% 994 p/s10.100.171.123 reporting-service.k8s-lan-party.svc.cluster.local.
65339 / 65536 [---------------------------------------------------------------------------------------------------------------------->] 99.70% 993 p/s10.100.171.123 -> reporting-service.k8s-lan-party.svc.cluster.local.
```

### 2. curl the reporting-service

- The `report-service`'s response is from its sidecar proxy `istio-envoy`. We know this by looking at the HTTP header `server: istio-envoy`.
- `transfer-encoding: chunked`: I am new to this, but I guess the server response payload will be chunked and sent back to the client with separate HTTP responses.

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

### 4. check if any other processes in the same host sending something to the `reporting-service` by `tcpdump`

Yes.

```
03:26:05.757521 ns-33c1e9 Out IP 192.168.15.111.36450 > reporting-service.k8s-lan-party.svc.cluster.local.http: Flags [.], ack 206, win 501, options [nop,nop,TS val 4050682731 ecr 2793517166], length 0
E..4..@...Y....o
d.{.b.P.u.0..Y'...........
.p.k...n
03:26:05.757621 ns-33c1e9 Out IP 192.168.15.111.36450 > reporting-service.k8s-lan-party.svc.cluster.local.http: Flags [F.], seq 215, ack 206, win 501, options [nop,nop,TS val 4050682731 ecr 2793517166], length 0
E..4..@...Y....o
d.{.b.P.u.0..Y'...........
.p.k...n
03:42:28.914336 ns-33c1e9 In  IP reporting-service.k8s-lan-party.svc.cluster.local.http > 192.168.15.111.44558: Flags [F.], seq 206, ack 216, win 508, options [nop,nop,TS val 2794500323 ecr 4051665888], length 0
E..4..@...."
d.{...o.P......h.ma...........
........
^C
```

### 5. Try to combine the encoded chunk together

- each chunk start with `4500` 

```
player@wiz-k8s-lan-party:~$ tcpdump -i any src port 80 -X -A -s 0
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
03:48:55.143753 ns-33c1e9 In  IP reporting-service.k8s-lan-party.svc.cluster.local.http > 192.168.15.111.59636: Flags [S.], seq 2264893675, ack 4098796295, win 65160, options [mss 1460,sackOK,TS val 2794886552 ecr 4052052117,nop,wscale 7], length 0
        0x0000:  4500 003c 0000 4000 7f06 75c5 0a64 ab7b  E..<..@...u..d.{
        0x0010:  c0a8 0f6f 0050 e8f4 86ff 88eb f44e ab07  ...o.P.......N..
        0x0020:  a012 fe88 8625 0000 0204 05b4 0402 080a  .....%..........
        0x0030:  a696 9598 f185 6895 0103 0307            ......h.....
03:48:55.144322 ns-33c1e9 In  IP reporting-service.k8s-lan-party.svc.cluster.local.http > 192.168.15.111.59636: Flags [.], ack 215, win 508, options [nop,nop,TS val 2794886553 ecr 4052052118], length 0
        0x0000:  4500 0034 ac57 4000 7f06 c975 0a64 ab7b  E..4.W@....u.d.{
        0x0010:  c0a8 0f6f 0050 e8f4 86ff 88ec f44e abdd  ...o.P.......N..
        0x0020:  8010 01fc 861d 0000 0101 080a a696 9599  ................
        0x0030:  f185 6896                                ..h.
03:48:55.147881 ns-33c1e9 In  IP reporting-service.k8s-lan-party.svc.cluster.local.http > 192.168.15.111.59636: Flags [P.], seq 1:206, ack 215, win 508, options [nop,nop,TS val 2794886556 ecr 4052052118], length 205: HTTP: HTTP/1.1 200 OK
        0x0000:  4500 0101 ac58 4000 7f06 c8a7 0a64 ab7b  E....X@......d.{
        0x0010:  c0a8 0f6f 0050 e8f4 86ff 88ec f44e abdd  ...o.P.......N..
        0x0020:  8018 01fc 86ea 0000 0101 080a a696 959c  ................
        0x0030:  f185 6896 4854 5450 2f31 2e31 2032 3030  ..h.HTTP/1.1.200
        0x0040:  204f 4b0d 0a73 6572 7665 723a 2069 7374  .OK..server:.ist
        0x0050:  696f 2d65 6e76 6f79 0d0a 6461 7465 3a20  io-envoy..date:.
        0x0060:  5468 752c 2032 3220 4d61 7920 3230 3235  Thu,.22.May.2025
        0x0070:  2030 333a 3438 3a35 3520 474d 540d 0a63  .03:48:55.GMT..c
        0x0080:  6f6e 7465 6e74 2d74 7970 653a 2074 6578  ontent-type:.tex
        0x0090:  742f 706c 6169 6e0d 0a78 2d65 6e76 6f79  t/plain..x-envoy
        0x00a0:  2d75 7073 7472 6561 6d2d 7365 7276 6963  -upstream-servic
        0x00b0:  652d 7469 6d65 3a20 320d 0a78 2d65 6e76  e-time:.2..x-env
        0x00c0:  6f79 2d64 6563 6f72 6174 6f72 2d6f 7065  oy-decorator-ope
        0x00d0:  7261 7469 6f6e 3a20 3a30 2f2a 0d0a 7472  ration:.:0/*..tr
        0x00e0:  616e 7366 6572 2d65 6e63 6f64 696e 673a  ansfer-encoding:
        0x00f0:  2063 6875 6e6b 6564 0d0a 0d0a 300d 0a0d  .chunked....0...
        0x0100:  0a          
```

### 6. oh, the flag is sent out from the client 

- The client is the invisble(different pid namespace) sidecar container. 
- But the containers in the same pod share the same network namespace!

```
04:17:30.663575 ns-33c1e9 Out IP 192.168.15.111.57396 > reporting-service.k8s-lan-party.svc.cluster.local.http: Flags [P.], seq 1:215, ack 1, win 502, options [nop,nop,TS val 4053767637 ecr 2796602072], length 214: HTTP: POST / HTTP/1.1
E..
..@....8...o
d.{.4.Ph(..*.9 ...........
........POST / HTTP/1.1
Host: reporting-service
User-Agent: curl/7.64.0
Accept: */*
Content-Length: 63
Content-Type: application/x-www-form-urlencoded

wiz_k8s_lan_party{good-crime-comes-with-a-partner-in-a-sidecar}
04:17:30.664014 ns-33c1e9 In  IP reporting-service.k8s-lan-party.svc.cluster.local.http > 192.168.15.111.57396: Flags [.], ack 215, win 508, options [nop,nop,TS val 2796602072 ecr 4053767637], length 0
E..4I.@...,.
d.{...o.P.4*.9 h(.............
........
04:17:30.667516 ns-33c1e9 In  IP reporting-service.k8s-lan-party.svc.cluster.local.http > 192.168.15.111.57396: Flags [P.], seq 1:206, ack 215, win 508, options [nop,nop,TS val 2796602076 ecr 4053767637], length 205: HTTP: HTTP/1.1 200 OK
E...I.@...+.
d.{...o.P.4*.9 h(.............
........HTTP/1.1 200 OK
server: istio-envoy
date: Thu, 22 May 2025 04:17:30 GMT
content-type: text/plain
x-envoy-upstream-service-time: 2
x-envoy-decorator-operation: :0/*
transfer-encoding: chunked

0
```

verifiy by only capture the packets from src `192.168.15.111`:

```
player@wiz-k8s-lan-party:~$ sudo tcpdump -i any src 192.168.15.111 and port 80 -A -s 0
sudo: The "no new privileges" flag is set, which prevents sudo from running as root.
sudo: If sudo is running in a container, you may need to adjust the container configuration to disable the flag.
player@wiz-k8s-lan-party:~$ tcpdump -i any src 192.168.15.111 and port 80 -A -s 0
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
04:30:05.040935 ns-33c1e9 Out IP 192.168.15.111.39962 > reporting-service.k8s-lan-party.svc.cluster.local.http: Flags [S], seq 2521709429, win 64240, options [mss 1460,sackOK,TS val 4054522015 ecr 0,nop,wscale 7], length 0
E..<._@....e...o
d.{...P.N;u.........%.........
............
04:30:05.041767 ns-33c1e9 Out IP 192.168.15.111.39962 > reporting-service.k8s-lan-party.svc.cluster.local.http: Flags [.], ack 2221787437, win 502, options [nop,nop,TS val 4054522016 ecr 2797356450], length 0
E..4.`@....l...o
d.{...P.N;v.m.-...........
......E.
04:30:05.041814 ns-33c1e9 Out IP 192.168.15.111.39962 > reporting-service.k8s-lan-party.svc.cluster.local.http: Flags [P.], seq 0:214, ack 1, win 502, options [nop,nop,TS val 4054522016 ecr 2797356450], length 214: HTTP: POST / HTTP/1.1
E..
.a@........o
d.{...P.N;v.m.-...........
......E.POST / HTTP/1.1
Host: reporting-service
User-Agent: curl/7.64.0
Accept: */*
Content-Length: 63
Content-Type: application/x-www-form-urlencoded

wiz_k8s_lan_party{good-crime-comes-with-a-partner-in-a-sidecar}
```
