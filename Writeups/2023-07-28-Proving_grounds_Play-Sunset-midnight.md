---
title: "Proving grounds Play: SunsetMidnight"
layout: post
date: 2023-07-28 10:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Pg-Play
- Linux
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds play linux machine writeup"
---
## NMAP
```sh
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Did not follow redirect to http://sunset-midnight/
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
3306/tcp open  mysql   MySQL 5.5.5-10.3.22-MariaDB-0+deb10u1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.22-MariaDB-0+deb10u1
```

## Fuzzing
## Files
```sh
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://sunset-midnight/FUZZ
Total requests: 37050

=====================================================================
ID           Response   Lines    Word       Chars       Payload                     
=====================================================================

000000005:   405        0 L      6 W        42 Ch       "xmlrpc.php"                
000000036:   200        87 L     300 W      4869 Ch     "wp-login.php"              
000000130:   200        97 L     823 W      7278 Ch     "readme.html"               
000000206:   200        384 L    3177 W     19915 Ch    "license.txt"               
000000248:   200        3 L      6 W        67 Ch       "robots.txt"                
000000263:   200        0 L      0 W        0 Ch        "wp-config.php"                            
000000413:   200        0 L      0 W        0 Ch        "wp-cron.php"                             
000000462:   200        11 L     24 W       228 Ch      "wp-links-opml.php"                           
000000838:   200        0 L      0 W        0 Ch        "wp-load.php"               
```

## Brute Forcing Mysql Credentials

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/hydra.png)

## Logging into Mysql DB

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/db1.png)

Get user credentials.

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/db2.png)

Generate new password MD5  hash.

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/md51.png)

Update user password.

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/db3.png)

Sucessfuly logged into wordpress admin portal.

## Uploading Reverse Shell in themes

Uploading revershell in the themes resulted in failure.

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/rev_upload_failed.png)

### Generate Malicious wordpress plugin

[GitHub](https://github.com/wetw0rk/malicious-wordpress-plugin)

The python code allows to create malicious reverse shell payload and write it to the zip file.

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/malicious_plugin_gen.png)

Upload and install the malicious plugin

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/upload_plugin.png)

### Trigger reverse shell

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/shell_trigger.png)

Shell obtained

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/stable_shell0.png)

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/stable_shell.png)

Post obtaining shell, hardcoded user credentials were found in the wordpress config files.

Found credentials for user `jose`

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/wp-config.png)

**SSH to user jose**

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/jose.png)

## Privilege Escalation

### SUIDs

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/SUID.png)

The status binary in the SUID runs services.

- Create a service
- Apply executable permission
- Run `/usr/bin/status` binary

**Service file contents**

```sh
/bin/sh
```

![img](/assets/images/CTF/Proving_Grounds/Sunset-midnight/root.png)

**Root obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).