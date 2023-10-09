---
title: "Proving grounds Play: DriftingBlues6"
layout: post
date: 2023-09-10 01:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- Pg-Play
- Dirtyc0w
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds play linux machine writeup"
---

## Nmap

```sh
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.22 ((Debian))
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
|_http-title: driftingblues
|_http-server-header: Apache/2.2.22 (Debian)
| http-robots.txt: 1 disallowed entry 
|_/textpattern/textpattern
```

## Web PORT: 80

![img](/assets/images/CTF/Proving_Grounds/DriftingBlues6/web.png)

## Fuzzing for files

/robots.txt

**Robots File**

```text
User-agent: *
Disallow: /textpattern/textpattern

dont forget to add .zip extension to your dir-brute
;)
```

### Login

http://192.168.151.219/textpattern/textpattern/

![img](/assets/images/CTF/Proving_Grounds/DriftingBlues6/login.png)

Zip file found at `http://192.168.151.219/spammer.zip`. File is password protected and which can be easliy cracked using `fcrackzip` tool.

```sh
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt spammer.zip
```

Extracted password: `myspace4`

Extracted `creds.txt` has login credentials for the CMS application.

```text
mayer:lionheart
```

Login to the application and upload reverse shell.

![img](/assets/images/CTF/Proving_Grounds/DriftingBlues6/upload.png)

Checked the document root configuration and triggered the reverse shell file.

**Initial foothold obtained**

![img](/assets/images/CTF/Proving_Grounds/DriftingBlues6/shell.png)

## Privilege Escalation

Check the kernel version to escalate privileges.

```sh
uname -a
Linux driftingblues 3.2.0-4-amd64 #1 SMP Debian 3.2.78-1 x86_64 GNU/Linux
```

Linux kernel <3.2.0-4-amd64 is vulnerable to [Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege Escalation (/etc/passwd Method)](https://www.exploit-db.com/exploits/40839).

Download the exploit into the attacking machine and compile the code as mentioned in the exploit.

Run the exploit as follow:

```sh
gcc -pthread dirty.c -o dirty -lcrypt
./dirty password #password is the password for the user firefart created by the exploit
```

Switch user to `firefart` and use the password `password`.

![img](/assets/images/CTF/Proving_Grounds/DriftingBlues6/root.png)

**Root shell obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).