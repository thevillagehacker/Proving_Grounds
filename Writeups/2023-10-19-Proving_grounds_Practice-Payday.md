---
title: "Proving grounds Play: Payday"
layout: post
date: 2023-10-19 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- RCE
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```text
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 4.6p1 Debian 5build1 (protocol 2.0)
80/tcp  open  http        Apache httpd 2.2.4 ((Ubuntu) PHP/5.2.3-1ubuntu6)
110/tcp open  pop3        Dovecot pop3d
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MSHOME)
143/tcp open  imap        Dovecot imapd
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MSHOME)
993/tcp open  ssl/imap    Dovecot imapd
995/tcp open  ssl/pop3    Dovecot pop3d
```

## 80/tcp  open http Apache httpd 2.2.4 ((Ubuntu) PHP/5.2.3-1ubuntu6)

### Directory Search

![img](/assets/images/CTF/Proving_Grounds/PayDay/dir.png)

**Admin login**

http://192.168.195.39/admin

> Credential: `admin:admin`

CS-cart software is vulnerable to [Remote Code Execution](https://www.exploit-db.com/exploits/48891).

Rename the pentest monkey php reverse shell to `.phtml` file and uplaod it to the template editor menu as new template. Direct the below url to trigger the reverse shell.

http://192.168.195.39/skins/shell.phtml

![img](/assets/images/CTF/Proving_Grounds/PayDay/shell.png)

**Initial Foothold Obtained**

## Privilege Escalation

Upon recon it was found that there are 2 users in the system which are `patrick` and `root`.

Brute forcing the SSH credentials for the user patrick using hydra have failed and as an alternative used `ncrack` to crack the password.

> Credential: `patrick:patrick`

![img](/assets/images/CTF/Proving_Grounds/PayDay/ncrack.png)

> Note: Always try to login to SSH using password as same as username.

Upon recon it was found that the user patrick can run ALL, run `sudo su` to obtain root.

![img](/assets/images/CTF/Proving_Grounds/PayDay/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).