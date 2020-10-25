---
title: Vulnhub - ColddBox Easy
description: My writeup on ColddBox Easy box.
categories:
 - vulnhub
tags: vulnhub wordpress wpscan
---

![](https://i.imgur.com/Zf32sKS.png)

## Box Stats

| Box Info      | Details       | 
| ------------- |:-------------:| 
| Box Name :    | **ColddBox**  | 
| Series :      | **ColddBox**         |
| Difficulty :  | **Easy**             |   
| Release Date :| **23 Oct 2020**      |    
| OS :          | **GNU/Linux**        |   
| Maker :       | [C0ldd](https://twitter.com/@C0ldd__){:target="_blank"}     | 
| Download :    | [ColddBox](https://www.vulnhub.com/entry/colddbox-easy,586/){:target="_blank"}      | 

## Summary

Hello all! Finally i found the time to do some CTF! :smile: This one was a very very very easy box based on wordpress stuff, a good box for warmup. We start by running a wpscan scan and we discover 3 users & we run a brute force on them using the rockyou wordlist. This way we get a shell as www-data. Privesc to root is simple we can exploit multiple stuff. Let’s pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.15                                                                                                                                                                130 ↵
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-25 23:54 EET
Nmap scan report for colddbox-easy.zte.com.cn (192.168.1.15)
Host is up (0.00028s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE
80/tcp   open  http
4512/tcp open  unknown
MAC Address: 08:00:27:5A:F4:E8 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.42 seconds
$ nmap -p 80,4512 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-25 23:54 EET
Nmap scan report for colddbox-easy.zte.com.cn (192.168.1.15)
Host is up (0.00061s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.1.31
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: ColddBox | One more machine
4512/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4e:bf:98:c0:9b:c5:36:80:8c:96:e8:96:95:65:97:3b (RSA)
|   256 88:17:f1:a8:44:f7:f8:06:2f:d3:4f:73:32:98:c7:c5 (ECDSA)
|_  256 f2:fc:6c:75:08:20:b1:b2:51:2d:94:d6:94:d7:51:4f (ED25519)
```
