---
title: "Proving grounds Play: Potato"
layout: post
date: 2023-09-20 02:00
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
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ef240eabd2b316b44b2e27c05f48798b (RSA)
|   256 f2d8353f4959858507e6a20e657a8c4b (ECDSA)
|_  256 0b2389c3c026d5645e93b7baf5147f3e (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Potato company
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
2112/tcp open  ftp     ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 ftp      ftp           901 Aug  2  2020 index.php.bak
|_-rw-r--r--   1 ftp      ftp            54 Aug  2  2020 welcome.msg
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web PORT: 80

![img](/assets/images/CTF/Proving_Grounds/Potato/web.png)

## Directory Fuzzing

![img](/assets/images/CTF/Proving_Grounds/Potato/dir.png)

Directory fuzzing revealed the admin directory presence and upon surfing the same prompted with login form.

## PORT 2112: FTP

![img](/assets/images/CTF/Proving_Grounds/Potato/ftp.png)


PORT 2112 has the FTP service running and which allows anaonymous login. Login to FTP server and download the index.php.bak file.

```php
<html>
<head></head>
<body>

<?php

$pass= "potato"; //note Change this password regularly

if($_GET['login']==="1"){
  if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
    echo "Welcome! </br> Go to the <a href=\"dashboard.php\">dashboard</a>";
    setcookie('pass', $pass, time() + 365*24*3600);
  }else{
    echo "<p>Bad login/password! </br> Return to the <a href=\"index.php\">login page</a> <p>";
  }
  exit();
}
?>


  <form action="index.php?login=1" method="POST">
                <h1>Login</h1>
                <label><b>User:</b></label>
                <input type="text" name="username" required>
                </br>
                <label><b>Password:</b></label>
                <input type="password" name="password" required>
                </br>
                <input type="submit" id='submit' value='Login' >
  </form>
</body>
</html>
```

After doing some research, the code is vulnerable to php type juggling vulnerability. Read more [here](https://owasp.org/www-pdf-archive/PHPMagicTricks-TypeJuggling.pdf?ref=infosecarticles.com).

```php
  if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
```

By sending the password variable as array `password[]=""` will result in authentication bypass.

![img](/assets/images/CTF/Proving_Grounds/Potato/bypass.png)

Direct to the dashboard and check the logs option.

![img](/assets/images/CTF/Proving_Grounds/Potato/logs.png)

Select the log and click Get the log button. Intercept the request in the Burp suite and send it to repeater for further inspection. Change the file name of the log file to `../../../../../../etc/passwd`.

The application is vulnerable to LFI vulnerability. Now copy the webadmin user hash locally and crack the password using john.

![img](/assets/images/CTF/Proving_Grounds/Potato/john.png)

Now SSH to user `webadmin` using the password `dragon`.

## Privilege Escalation

Check the user permission allowed for the user webadmin.

```sh
badmin@serv:~$ sudo -l
Matching Defaults entries for webadmin on serv:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User webadmin may run the following commands on serv:
    (ALL : ALL) /bin/nice /notes/*
webadmin@serv:~$ 
```

The user has permission to run the binary `/bin/nice` on directory `/notes/*`. The `/notes/*` essentially means all the files and subdirectories that are contained within the "notes" directory. This is often used in commands to perform operations on multiple files or directories within a specific directory.

![img](/assets/images/CTF/Proving_Grounds/Potato/ls.png)

The user webadmin does not have any permission to add or edit existing files in the notes directory. So create a bash script with content as `"/bin/bash"` and save it in the webadmin home directory.

Apply `chmod +x` to the script to make it executable file.

Run the below command to obtain root shell.

![img](/assets/images/CTF/Proving_Grounds/Potato/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).