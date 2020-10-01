---
title: HackMyVM - Connection
description: My writeup on Connection box.
categories:
 - HackMyVM
tags: HackMyVM
---

![](https://i.imgur.com/VYEVjve.png)

You can start playing there > [HackMyVM](https://hackmyvm.eu/){:target="_blank"}

## Summary

I found this really cool lab, similar to vulnhub but has point system/ranking etc. I suggest you to try this out, 101% worth it. Let’s pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.17
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-01 12:27 EEST
Nmap scan report for connection.zte.com.cn (192.168.1.17)
Host is up (0.00020s latency).
Not shown: 65531 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 08:00:27:49:78:7B (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.20 seconds
$ nmap -p 22,80,139,445 -sC -sV -oN nmap/initial $ip 
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-01 12:28 EEST
Nmap scan report for connection.zte.com.cn (192.168.1.17)
Host is up (0.00047s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 b7:e6:01:b5:f9:06:a1:ea:40:04:29:44:f4:df:22:a1 (RSA)
|   256 fb:16:94:df:93:89:c7:56:85:84:22:9e:a0:be:7c:95 (ECDSA)
|_  256 45:2e:fb:87:04:eb:d1:8b:92:6f:6a:ea:5a:a2:a1:1c (ED25519)
80/tcp  open  http        Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Apache2 Debian Default Page: It works
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
```
