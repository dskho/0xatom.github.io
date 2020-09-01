---
title: Vulnhub - Photographer
description: My writeup on Photographer box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://i.imgur.com/9KCQ8Re.png)

Hi all, after a long break im back again! Ready for new CTF adventures, let's begin!

You can find the machine there > [Photographer](https://www.vulnhub.com/entry/photographer-1,519/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.5
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-01 16:12 EEST
Nmap scan report for photographer.zte.com.cn (192.168.1.5)
Host is up (0.0013s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
8000/tcp open  http-alt
MAC Address: 00:0C:29:89:92:69 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 2.42 seconds
$ nmap -p 80,139,445,8000 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-01 16:12 EEST
Nmap scan report for photographer.zte.com.cn (192.168.1.5)
Host is up (0.00030s latency).

PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Photographer by v1n1v131r4
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
8000/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Koken API error
|_http-trane-info: Problem with XML parsing of /evox/about
MAC Address: 00:0C:29:89:92:69 (VMware)
Service Info: Host: PHOTOGRAPHER
```
