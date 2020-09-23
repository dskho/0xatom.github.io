---
title: Vulnhub - Bizarre Adventure Mrr3b0t
description: My writeup on Bizarre Adventure Mrr3b0t box.
categories:
 - vulnhub
tags: vulnhub hydra
---

![](https://images-na.ssl-images-amazon.com/images/I/71Z2amvT-tL._AC_SL1062_.jpg)

You can find the machine there > [Bizarre Adventure Mrr3b0t](https://www.vulnhub.com/entry/bizarre-adventure-mrr3b0t,561/){:target="_blank"}

## Summary

Let's pwn it! :sunglasses:

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
