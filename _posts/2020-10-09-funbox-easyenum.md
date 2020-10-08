---
title: Vulnhub - Funbox EasyEnum
description: My writeup on Funbox EasyEnum box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://i.imgur.com/6ghxvXy.png)

## Box Stats

| Box Info      | Details       | 
| ------------- |:-------------:| 
| Name :        | **Funbox EasyEnum**  | 
| Difficulty :  | **Easy/Medium**      |   
| Release Date :| **19 Sep 2020**      |    
| OS :          | **GNU/Linux**        |   
| Maker :       | [@0815R2d2](https://twitter.com/@0815R2d2){:target="_blank"}      | 
| Download :    | [Funbox EasyEnum](https://www.vulnhub.com/entry/funbox-easyenum,565/){:target="_blank"}      | 

## Summary

Let’s pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.17 
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 02:01 EEST
Nmap scan report for funbox7.zte.com.cn (192.168.1.17)
Host is up (0.00049s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:DE:A0:BD (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.40 seconds
$ nmap -p 22,80 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 02:01 EEST
Nmap scan report for funbox7.zte.com.cn (192.168.1.17)
Host is up (0.00035s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9c:52:32:5b:8b:f6:38:c7:7f:a1:b7:04:85:49:54:f3 (RSA)
|   256 d6:13:56:06:15:36:24:ad:65:5e:7a:a1:8c:e5:64:f4 (ECDSA)
|_  256 1b:a9:f3:5a:d0:51:83:18:3a:23:dd:c4:a9:be:59:f0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```

