---
title: Vulnhub - GainPower
description: My writeup on GainPower box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://static.vecteezy.com/system/resources/previews/000/602/897/non_2x/creative-power-logo-concept-design-templates-vector.jpg)

Hi all, i have to admit that this box took me a while to pwn it. I got stuck HARD for the privesc & in the end was really easy but i got confused anyway let's pwn it. :)

You can find the machine there > [GainPower](https://www.vulnhub.com/entry/gainpower-1,493/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.2                       
$ nmap -sC -sV -p- -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-02 00:42 EEST
Nmap scan report for 192.168.1.2 (192.168.1.2)
Host is up (0.00096s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 88:41:61:11:e1:1f:18:7d:d6:0c:38:29:25:79:16:2c (RSA)
|   256 18:c5:fd:ce:cd:2b:92:f8:d9:17:17:21:24:9d:67:df (ECDSA)
|_  256 84:c5:14:e4:e9:33:21:41:6a:92:72:b9:a7:33:1a:ea (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS))
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS)
|_http-title: Watch shop | eCommers
8000/tcp open  http    Ajenti http control panel
|_http-title: Ajenti
MAC Address: 00:0C:29:5E:CF:3F (VMware)
```
