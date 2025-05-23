---
tags:
  - Kubernetes
  - Istio
  - Service-Mesh
title: K8S LAN Party - The Beauty and The Ist
description: 
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
