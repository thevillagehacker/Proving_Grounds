---
title: "Proving grounds Play: Vegeta1"
layout: post
date: 2023-09-21 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- Pg-Play
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds play linux machine writeup"
---

## Nmap

```sh
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 1f3130673f08302e6daee3209ebd6bba (RSA)
|   256 7d8855a86f56c805a47382dcd8db4759 (ECDSA)
|_  256 ccdede4e84a891f51ad6d2a62e9e1ce0 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web PORT: 80

![img](/assets/images/CTF/Proving_Grounds/Vegeta1/web.png)

## Directory Fuzzing

Upon fuzzing for directories the robots.txt file revealed a directory named `/find_me`.

There is a hidden base64 text present in the index file.

![img](/assets/images/CTF/Proving_Grounds/Vegeta1/hidden.png)

Upon decoding the text twice in base64 format using cyberchef revealed that the decoded data is a image. 

![img](/assets/images/CTF/Proving_Grounds/Vegeta1/decode.png)

Download the image and check for informations in it's meta data. After checking the file it was found to be a QR code image, use the online decoders to extract information.

![img](/assets/images/CTF/Proving_Grounds/Vegeta1/qrdecode.png)

It has password but we don't have a username to login. More directory fuzzing...

After doing more fuzzing the directory `/bulma/` has been discovered and it contains a `.wav` file which is a morse code.

Use the online morse code decoder as in audio format and the extracted information is the username and password for the SSH login.

![img](/assets/images/CTF/Proving_Grounds/Vegeta1/morse.png)

Login to the attacking machine using the credentials `trunks:u$3r`

**Initial Foothold obtained**

## Privilege Escalation

Check the `bash_history` file, the history shows the user added a new user `Tom` to the system, which means the user has write permission on `/etc/passwd` file.

![img](/assets/images/CTF/Proving_Grounds/Vegeta1/history.png)

check the `/etc/password` file to ensure the user has write permissions.

```sh
trunks@Vegeta:~$ ls -al /etc/passwd
-rw-r--r-- 1 trunks root 1539 Sep 21 08:28 /etc/passwd
trunks@Vegeta:~$ 
```
After checking the /etc/passwd file there are no entries for the user `Tom`. So add the user to the `/etc/passwd` file as shown in the history file. And copy the hash and crack it using john in the local machine.

Switch to user `Tom` using password `Password`.

![img](/assets/images/CTF/Proving_Grounds/Vegeta1/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).