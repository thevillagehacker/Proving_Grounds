---
title: "Proving grounds Play: Election1"
layout: post
date: 2023-07-18 12:00
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
# Walkthrough on Youtube

[![youtube](/assets/images/CTF/Proving_Grounds/Election1/youtube.png)](https://youtu.be/4ls30YSlfAM)

## NMAP

```sh
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 20d1ed84cc68a5a786f0dab8923fd967 (RSA)
|   256 7889b3a2751276922af98d27c108a7b9 (ECDSA)
|_  256 b8f4d661cf1690c5071899b07c70fdc0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web Discovery

### Fuzzing for Directories

http://192.168.176.211/FUZZ/

![web_fuzz](/assets/images/CTF/Proving_Grounds/Election1/dir_wfuzz.png)

Landing webpage

![web_fuzz](/assets/images/CTF/Proving_Grounds/Election1/webpage_home.png)

http://192.168.176.211/election/FUZZ/

![web_fuzz](/assets/images/CTF/Proving_Grounds/Election1/dir_wfuzz2.png)

More directory fuzzing...🏃

![web_fuzz](/assets/images/CTF/Proving_Grounds/Election1/dir_wfuzz3.png)

Found a log file at `http://192.168.176.211/election/admin/logs/`

![log](/assets/images/CTF/Proving_Grounds/Election1/log_file.png)

#### Contents of the log file
```log
[2020-01-01 00:00:00] Assigned Password for the user love: P@$$w0rd@123
[2020-04-03 00:13:53] Love added candidate 'Love'.
[2020-04-08 19:26:34] Love has been logged in from Unknown IP on Firefox (Linux).
```

### Credentials
```text
love: P@$$w0rd@123
```

## SSH Login using extracted credentials

![ssh](/assets/images/CTF/Proving_Grounds/Election1/ssh01.png)

![local](/assets/images/CTF/Proving_Grounds/Election1/local.png)

## Privilege Escalation

### SUIDs

![SUI](/assets/images/CTF/Proving_Grounds/Election1/SUID.png)

**Searchsploit search**

```sh
------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                    |  Path
------------------------------------------------------------------ ---------------------------------
RhinoSoft Serv-U FTP Server 3.x < 5.x - Local Privilege Escalatio | windows/local/381.c
Serv-U FTP Server - prepareinstallation Privilege Escalation (Met | linux/local/47072.rb
Serv-U FTP Server - prepareinstallation Privilege Escalation (Met | linux/local/47072.rb
Serv-U FTP Server < 15.1.7 - Local Privilege Escalation (1)       | linux/local/47009.c
Serv-U FTP Server < 15.1.7 - Local Privilege Escalation (2)       | multiple/local/47173.sh
------------------------------------------------------------------ ---------------------------------
```

- Copy the exploit code to the attacking machine usig `wget`.
- Apply required permission `chmod 755 exploit.sh`.
- run the `exploit.sh` file.

## Root Shell

**Root user access and proof.txt obtained**

![root](/assets/images/CTF/Proving_Grounds/Election1/root.png)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).