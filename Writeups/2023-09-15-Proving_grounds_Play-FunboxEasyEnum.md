---
title: "Proving grounds Play: FunboxEasyEnum"
layout: post
date: 2023-09-15 01:00
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
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9c52325b8bf638c77fa1b704854954f3 (RSA)
|   256 d6135606153624ad655e7aa18ce564f4 (ECDSA)
|_  256 1ba9f35ad05183183a23ddc4a9be59f0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web PORT: 80

```text
http://192.168.177.132/mini.php
```

## Upload reverse shell

![img](/assets/images/CTF/Proving_Grounds/FunboxEasyEnum/web.png)

Trigger reverse shell by visiting `192.168.177.132/shell.php` URL.

**Initial Foothold obtained**

![img](/assets/images/CTF/Proving_Grounds/FunboxEasyEnum/shell.png)

## User Enumeration

```text
goat
harry
karla
oracle
sally
```

The above are the list of users who have home directory in the machine. The user `goat` has the password configured as same as username `goat`.

SSH to the user `goat` or switch user.

## Privilege Escalation

Enumerate executable permissions for the user goat.

```sh
t@funbox7:~$ sudo -l
Matching Defaults entries for goat on funbox7:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User goat may run the following commands on funbox7:
    (root) NOPASSWD: /usr/bin/mysql
```

Search for sudo exploit on GTFOBins for `mysql`.

![img](/assets/images/CTF/Proving_Grounds/FunboxEasyEnum/gtfo.png)

![img](/assets/images/CTF/Proving_Grounds/FunboxEasyEnum/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).