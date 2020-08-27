---
title:  "HackTheBox - Shocker"
excerpt: "Shocker is a webserver that is vulnerable to [Shellshock](https://owasp.org/www-pdf-archive/Shellshock_-_Tudor_Enache.pdf). After exploiting `shellshock` and getting a shell we grab the user flag. To escalate privileges to root we abust `SUDO` & `Perl`."
date: 2020-08-27

categories:
  - Write-Ups 
  - HackTheBox
tags:
  - Linux
  - BASH
  - Shellshock
  - Bashdoor
  - Perl
  - SUID
---
## Introduction
![](/assets/images/htb-shocker/thumb.png)

Difficulty Rating: Easy
Creator: [@mrb3n813](https://twitter.com/mrb3n813)

Shocker is a webserver that is vulnerable to [Shellshock](https://owasp.org/www-pdf-archive/Shellshock_-_Tudor_Enache.pdf). After exploiting `shellshock` and getting a shell we grab the user flag. To escalate privileges to root we abust `SUDO` & `Perl`.

## Summary 

- Enumerate a `/cgi-bin/` directory to find a shellshock vulnerability.
- Exploit `Shellshock`.
- Abuse `SUDO` & `perl` to gain root privileges.

## Tools Used

- Nmap
- Gobuster

## Enumeration

We begin with a simple nmap scan.

`-sC` Performs a script scan using the default set of scripts.
`-sV` Enables version detection

```
root@kali:~# nmap -sC -sV 10.10.10.56
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-27 00:49 EDT
Nmap scan report for 10.10.10.56
Host is up (0.060s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.61 seconds
```
Our Nmap scan reveals that there are 2 ports open. Port 80 running http and Port 2222 running ssh. Let's enumerate port 80 and see what we can find. 

Upon first glance there isn't much on the site.
![](INSERT WEB IMAGE)

Look's like were going to have to enumerate some more.

```
INSERT GOBUSTER OUTPUT

```
We don't find much with our scans except for index.html and a forbidden directory of `/cgi-bin/`. Let's enumerate `/cgi-bin/` some more to see if we can find anything else.

```
INSERT GOBUSTESR OUTPUT
```















## Privilege Escalation (Root)


## Remediations

