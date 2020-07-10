---
title: Vulnhub - CyberSploit
description: My writeup on CyberSploit box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://pbs.twimg.com/media/Dsh2wzTWkAEgsd0.jpg)

Hi all, i really hate this type of boxes, but anyway let's pwn it! :D

You can find the machine there > [CyberSploit](https://www.vulnhub.com/entry/cybersploit-1,506/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.7
$ cybersploit nmap -sC -sV -p- -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-10 12:47 EEST
Nmap scan report for 192.168.1.7
Host is up (0.0013s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 01:1b:c8:fe:18:71:28:60:84:6a:9f:30:35:11:66:3d (DSA)
|   2048 d9:53:14:a3:7f:99:51:40:3f:49:ef:ef:7f:8b:35:de (RSA)
|_  256 ef:43:5b:d0:c0:eb:ee:3e:76:61:5c:6d:ce:15:fe:7e (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Hello Pentester!
MAC Address: 00:0C:29:3A:64:E3 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

