---
title: "Proving grounds Practice: Algernon"
layout: post
date: 2023-08-26 06:00
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
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd => Anonymous login
80/tcp    open  http          Microsoft IIS httpd 10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
9998/tcp  open  http          Microsoft IIS httpd 10.0
17001/tcp open  remoting      MS .NET Remoting services
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
```

## Directory Fuzzing

```text
http://192.168.172.65/aspnet_client/
http://192.168.172.65:9998/interface/root#/login
```

## PORT: 9998

![img](/assets/images/CTF/Proving_Grounds/Algernon/9998.png)

### Searchsploit

![img](/assets/images/CTF/Proving_Grounds/Algernon/searchsploit.png)

Change the IP addressess and PORT in the exploit code and run netcat listener on the PORT specified.

![img](/assets/images/CTF/Proving_Grounds/Algernon/exploit.png)

Run the python exploit.

![img](/assets/images/CTF/Proving_Grounds/Algernon/shell.png)

**Shell Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).