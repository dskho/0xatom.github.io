---
title: Vulnhub - Super Mario Host
description: My writeup on Super Mario Host box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://static.posters.cz/image/750/%CE%91%CF%86%CE%AF%CF%83%CE%B5%CF%82/super-mario-characters-i22822.jpg)

Hi all, Let's pwn it!

You can find the machine there > [Super Mario Host](https://www.vulnhub.com/entry/super-mario-host-101,186/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.6
$ nmap -sC -sV -p- -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-26 00:44 EEST
Nmap scan report for mario.supermariohost.local (192.168.1.6)
Host is up (0.0010s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 1c:97:c0:06:3b:cb:4f:6f:0f:65:8d:37:82:c4:23:59 (DSA)
|   2048 45:2d:fe:04:bb:98:ed:00:d7:7b:36:da:8f:cf:44:1c (RSA)
|   256 09:5c:25:9d:5c:54:ae:8d:90:e3:44:9b:5e:a1:4d:e0 (ECDSA)
|_  256 c9:d5:6a:32:53:ab:8a:43:74:4b:85:fb:a0:ba:40:52 (ED25519)
8180/tcp open  http    Apache httpd
|_http-server-header: Apache
|_http-title: Mario
MAC Address: 00:0C:29:DF:8C:2E (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

When we visit the site, we get the default nginx page :

![](https://i.imgur.com/OW6e7me.png)

Let's run `gobuster` on it.

```
$ gobuster dir -q -u http://$ip:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -o gobuster1.txt
/vhosts (Status: 200)
/server-status (Status: 403)
```

`/vhosts` has a virtual host file :

```
<VirtualHost *:80>

	ServerName mario.supermariohost.local
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/supermariohost
	DirectoryIndex mario.php

	ErrorLog ${APACHE_LOG_DIR}/supermariohost_error.log
	CustomLog ${APACHE_LOG_DIR}/supermariohost_access.log combined
 
</VirtualHost>
```

`ServerName mario.supermariohost.local` is the domain, let's add it at `/etc/hosts`

```
$ cat /etc/hosts
192.168.1.6     mario.supermariohost.local
```

Perfect, now if we browser the website with the domain name we can see a different page : 

![](https://i.imgur.com/GuqIVTU.png)

Let's run `gobuster` on it.


`
