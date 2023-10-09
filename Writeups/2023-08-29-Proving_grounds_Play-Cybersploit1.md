---
title: "Proving grounds Play: CyberSploit1"
layout: post
date: 2023-08-29 01:00
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

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 011bc8fe18712860846a9f303511663d (DSA)
|   2048 d95314a37f9951403f49efef7f8b35de (RSA)
|_  256 ef435bd0c0ebee3e76615c6dce15fe7e (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Hello Pentester!
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.22 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web Port : 80

![img](/assets/images/CTF/Proving_Grounds/Cybersploit1/web.png)

## Fuzzing

![img](/assets/images/CTF/Proving_Grounds/Cybersploit1/files.png)

**SSH Password found on robots.txt file**

![img](/assets/images/CTF/Proving_Grounds/Cybersploit1/flag1.png)

Decode the base64 text to get the SSH password.

## Initial Foothold

Since we have the username, password and PORT 22 `SSH` is open, SSH to the attacking machine using the same.

![img](/assets/images/CTF/Proving_Grounds/Cybersploit1/ssh.png)

## Privilege Escalation

![img](/assets/images/CTF/Proving_Grounds/Cybersploit1/kernel.png)

The linux kernel in the attacking machine is vulnerable to [Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation](https://www.exploit-db.com/exploits/37292) vulnerability.

Download the exploit code to the local machine and use python http server to download the exploit to the remote machine.

Run the following to obtain root shell.

```sh
gcc exploit.c
./a.out
```

![img](/assets/images/CTF/Proving_Grounds/Cybersploit1/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).