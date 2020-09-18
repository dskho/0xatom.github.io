---
title: Vulnhub - Monitoring
description: My writeup on Monitoring box.
categories:
 - vulnhub
tags: vulnhub nagios metasploit
---

![](https://3.bp.blogspot.com/-WvEbEYfcXkM/WfzbFrCIClI/AAAAAAAALBA/PcLISE3UWJ4RjRYmINybOdk0rY2RZt04ACLcBGAs/s1600/Nagios-XI.jpg)

You can find the machine there > [Monitoring](https://www.vulnhub.com/entry/monitoring-1,555/){:target="_blank"}

## Summary

This was a pretty easy box, i had some troubles finding the right exploit for it. It's all about exploiting a vulnerable version of Nagios XI this will provide us a root shell. Let's pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.9
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-18 12:01 EEST
Nmap scan report for ubuntu.zte.com.cn (192.168.1.9)
Host is up (0.00010s latency).
Not shown: 65529 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
389/tcp  open  ldap
443/tcp  open  https
5667/tcp open  unknown
MAC Address: 00:0C:29:DF:7A:FC (VMware)

Nmap done: 1 IP address (1 host up) scanned in 2.19 seconds
$ nmap -p 22,25,80,389,443,5667 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-18 12:02 EEST
Nmap scan report for ubuntu.zte.com.cn (192.168.1.9)
Host is up (0.00052s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b8:8c:40:f6:5f:2a:8b:f7:92:a8:81:4b:bb:59:6d:02 (RSA)
|   256 e7:bb:11:c1:2e:cd:39:91:68:4e:aa:01:f6:de:e6:19 (ECDSA)
|_  256 0f:8e:28:a7:b7:1d:60:bf:a6:2b:dd:a3:6d:d1:4e:a4 (ED25519)
25/tcp   open  smtp       Postfix smtpd
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
| ssl-cert: Subject: commonName=ubuntu
| Not valid before: 2020-09-08T17:59:00
|_Not valid after:  2030-09-06T17:59:00
|_ssl-date: TLS randomness does not represent time
80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Nagios XI
389/tcp  open  ldap       OpenLDAP 2.2.X - 2.3.X
443/tcp  open  ssl/http   Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Nagios XI
| ssl-cert: Subject: commonName=192.168.1.6/organizationName=Nagios Enterprises/stateOrProvinceName=Minnesota/countryName=US
| Not valid before: 2020-09-08T18:28:08
|_Not valid after:  2030-09-06T18:28:08
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
5667/tcp open  tcpwrapped
```

Lot of stuff, http/https display the same website Nagios XI platform. Nagios is an open-source computer-software application that monitors systems, networks etc. I've 0 experience with nagios and nagios pentesting.
