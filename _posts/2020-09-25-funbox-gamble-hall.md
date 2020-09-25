---
title: Vulnhub - Funbox Gamble Hall
description: My writeup on Funbox Gamble Hall box.
categories:
 - vulnhub
tags: vulnhub wordpress
---

![](https://filmdaily.co/wp-content/uploads/2020/03/Gamble-lede.jpg)

You can find the machine there > [Funbox Gamble Hall](https://www.vulnhub.com/entry/funbox-gamble-hall,551/){:target="_blank"}

## Summary

This was a really tricky box, that enjoyed pwning! Let's pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.12
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-25 15:52 EEST
Nmap scan report for funbox6.zte.com.cn (192.168.1.12)
Host is up (0.00020s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:18:41:BB (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.28 seconds
$ nmap -p 22,80 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-25 15:56 EEST
Nmap scan report for funbox6.zte.com.cn (192.168.1.12)
Host is up (0.00051s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 0e:4f:3c:37:75:8a:a4:4d:bb:17:50:1b:ec:93:02:15 (RSA)
|   256 d7:dc:fc:b1:76:d6:76:13:da:ea:c4:30:04:bc:da:d2 (ECDSA)
|_  256 51:19:47:a6:29:c8:22:10:c2:73:34:ad:de:7f:57:d3 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Did not follow redirect to http://funbox6.box/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
```
