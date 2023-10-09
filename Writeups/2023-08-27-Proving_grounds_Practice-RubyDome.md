---
title: "Proving grounds Practice: RubyDome"
layout: post
date: 2023-08-27 02:00
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
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9bc8f013f855df95cd9fbb615a01e74 (ECDSA)
|_  256 53d97f3d228afd5798fe6b1a4cac7967 (ED25519)
3000/tcp open  http    WEBrick httpd 1.7.0 (Ruby 3.0.2 (2021-07-07))
|_http-title: RubyDome HTML to PDF
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: WEBrick/1.7.0 (Ruby/3.0.2/2021-07-07)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web

![img](/assets/images/CTF/Proving_Grounds/RubyDome/web.png)

WEBrick 1.7.0 is vulnerable to Command injection and as per the above screenshot the application gets the URL and converts the page content into pdf. Passing malicious inputs or invalid URL resulted in server error with exception shown in the web page.

![img](/assets/images/CTF/Proving_Grounds/RubyDome/url.png)

The PDFKit is used to convert the contents to pdf. The PDFkit used in the application is vulnerable to [Command Injection](https://www.exploit-db.com/exploits/51293).

![img](/assets/images/CTF/Proving_Grounds/RubyDome/if.png)

**Initial Foothold Obtained**

## Privilege Escalation

Check the system user executable permissions.

![img](/assets/images/CTF/Proving_Grounds/RubyDome/permissions.png)

As per the above image the user `andrew` can run the file `app.rb` using ruby as sudo user without password.

Add the below content to the `app.rb` file and execute the file using `/usr/bin/ruby` as super user.

```sh
echo 'exec "/bin/bash"' > app.rb
```

![img](/assets/images/CTF/Proving_Grounds/RubyDome/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).