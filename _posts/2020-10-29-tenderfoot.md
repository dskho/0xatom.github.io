---
title: Vulnhub - TenderFoot
description: My writeup on TenderFoot box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://i.imgur.com/wjyiTq8.png)

## Box Stats

| Box Info      | Details       | 
| ------------- |:-------------:| 
| Box Name :    | **TenderFoot**  | 
| Series :      | **TenderFoot**         |
| Difficulty :  | **Easy**             |   
| Release Date :| **5 Oct 2020**      |    
| OS :          | **GNU/Linux**        |   
| Maker :       | [Anant Chauhan](https://twitter.com/_Anant_chauhan){:target="_blank"}     | 
| Download :    | [TenderFoot](https://www.vulnhub.com/entry/tenderfoot-1,581/){:target="_blank"}      | 

## Summary

Let's pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.7                                                                                                                                                                130 ↵
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-29 11:37 EET
Nmap scan report for tenderfoot.zte.com.cn (192.168.1.7)
Host is up (0.00049s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:EA:E6:8D (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.49 seconds
$ nmap -p 22,80 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-29 11:37 EET
Nmap scan report for tenderfoot.zte.com.cn (192.168.1.7)
Host is up (0.00045s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a2:b7:2d:95:e1:06:7f:a3:f1:8e:bc:5b:4c:29:19:61 (RSA)
|   256 42:0c:c9:6d:1d:e9:84:19:6a:8a:d5:51:2c:69:c6:98 (ECDSA)
|_  256 14:4d:74:42:78:67:9b:f3:dd:00:40:24:4d:12:c9:de (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```
