---
title: "Proving grounds Play: Seppuku"
layout: post
date: 2023-09-19 02:00
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
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           vsftpd 3.0.3
22/tcp   open  ssh           OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 cd55a8e40f28bcb2a67d4176bb9f71f4 (RSA)
|   256 16fa29e4e08a2e7d37d26f42b2dce922 (ECDSA)
|_  256 bb74e897fa308ddaf95c99f0d9248ad5 (ED25519)
80/tcp   open  http          nginx 1.14.2
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Restricted Content
|_http-title: 401 Authorization Required
|_http-server-header: nginx/1.14.2
139/tcp  open  netbios-ssn   Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn   Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
7080/tcp open  ssl/empowerid LiteSpeed
7601/tcp open  http          Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-title: Seppuku
8088/tcp open  http          LiteSpeed httpd
```

## Web PORT: 7601

![img](/assets/images/CTF/Proving_Grounds/Seppuku/web1706.png)

### Directory Fuzzing

![img](/assets/images/CTF/Proving_Grounds/Seppuku/dir.png)

### Folder: /Secret/

![img](/assets/images/CTF/Proving_Grounds/Seppuku/secret.png)

There is a username in the hostname file and a list of passwords which is a wordlist that allows us to bruteforce the SSH credentials.

![img](/assets/images/CTF/Proving_Grounds/Seppuku/hydra.png)

Login to the attacking machine using the SSH credentials cracked on the above step.

![img](/assets/images/CTF/Proving_Grounds/Seppuku/shell.png)

Upon checking the files in the `seppuku` user there is a file named `.passwd` which contains the password for the user `samurai`. Switch to user samurai using the password and check the executable permissions.

![img](/assets/images/CTF/Proving_Grounds/Seppuku/samurai.png)

The user samurai can run a program located in the directory `/../../../../../../home/tanto/.cgi_bin/bin` which is located in the user tanto.

In order to obtain the root we will have to put a malicious program as a bin file and execute it as samurai. 

## Privilege Escalation

During the directory fuzzing there was directory that was discovered which contains a private key for an ssh user. Upon checking the user tanto's .ssh directory there is an entry for the authorized keys.

http://192.168.166.90:7601/keys/

Download the private key to seppuku shell and apply required permissions to login to tanto.

Escape rbash using below vi bypass.

```sh
vi
:set shell=/bin/bash
:shell
```

Create a file named bin with content to just spawn a bash shell and apply `chmod 777` permission. The permission allows any users in the machine to read, write and execute the program.

![img](/assets/images/CTF/Proving_Grounds/Seppuku/tanto.png)

Now execute the command the user samurai is allowed to get root shell.

![img](/assets/images/CTF/Proving_Grounds/Seppuku/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).