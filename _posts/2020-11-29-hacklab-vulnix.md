---
title: Vulnhub - HackLAB Vulnix
description: My writeup on HackLAB Vulnix box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://i.imgur.com/B2q8rOI.jpg)

## Box Stats

| Box Info      | Details       | 
| ------------- |:-------------:| 
| Box Name :    | **HackLAB Vulnix**  | 
| Series :      | **HackLAB**         |
| Difficulty :  | **Medium**             |   
| Release Date :| **10 Sep 2012**      |    
| OS :          | **GNU/Linux**        |   
| Maker :       | [Reboot User](https://twitter.com/oshearing){:target="_blank"}     | 
| Download :    | [HackLAB Vulnix](https://www.vulnhub.com/entry/hacklab-vulnix,48/){:target="_blank"}      | 
| Recommended : | Yes :heavy_check_mark:      | 

## Summary

Ah, many years have passed since i pwned this box. I did this box with my friend `@Redfox`, it was a great teamwork. Let's pwn it! :sunglasses:

## PoC 

![]()

## Target IP

Always first step is to find target IP address, i prefer to use the `arp-scan` utility. Then we'll save the IP address in a variable. 

```
$ arp-scan --localnet | grep "VMware"
192.168.1.21	00:0c:29:a8:8a:f7	VMware, Inc.
$ ip=192.168.1.21
```

## Enumeration/Reconnaissance

Now as always let's continue with a nmap scan.

```
$ nmap -p- -sC -sV -oN nmap/vulnix.vulnhub $ip
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-29 12:59 EET
Nmap scan report for vulnix.zte.com.cn (192.168.1.21)
Host is up (0.0032s latency).
Not shown: 65518 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 10:cd:9e:a0:e4:e0:30:24:3e:bd:67:5f:75:4a:33:bf (DSA)
|   2048 bc:f9:24:07:2f:cb:76:80:0d:27:a6:48:52:0a:24:3a (RSA)
|_  256 4d:bb:4a:c1:18:e8:da:d1:82:6f:58:52:9c:ee:34:5f (ECDSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: vulnix, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
|_ssl-date: 2020-11-29T11:03:13+00:00; +4s from scanner time.
79/tcp    open  finger     Linux fingerd
|_finger: No one logged on.\x0D
110/tcp   open  pop3?
|_ssl-date: 2020-11-29T11:03:13+00:00; +4s from scanner time.
111/tcp   open  rpcbind    2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      37743/tcp6  mountd
|   100005  1,2,3      38588/udp   mountd
|   100005  1,2,3      39752/tcp   mountd
|   100005  1,2,3      56194/udp6  mountd
|   100021  1,3,4      41725/udp6  nlockmgr
|   100021  1,3,4      50894/tcp6  nlockmgr
|   100021  1,3,4      53356/tcp   nlockmgr
|   100021  1,3,4      59885/udp   nlockmgr
|   100024  1          39049/udp   status
|   100024  1          48215/udp6  status
|   100024  1          50402/tcp6  status
|   100024  1          50714/tcp   status
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
143/tcp   open  imap       Dovecot imapd
|_ssl-date: 2020-11-29T11:03:13+00:00; +4s from scanner time.
512/tcp   open  exec?
513/tcp   open  login      OpenBSD or Solaris rlogind
514/tcp   open  tcpwrapped
993/tcp   open  ssl/imaps?
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2020-11-29T11:03:12+00:00; +3s from scanner time.
995/tcp   open  ssl/pop3s?
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2020-11-29T11:03:12+00:00; +3s from scanner time.
2049/tcp  open  nfs_acl    2-3 (RPC #100227)
39752/tcp open  mountd     1-3 (RPC #100005)
48722/tcp open  mountd     1-3 (RPC #100005)
50714/tcp open  status     1 (RPC #100024)
53356/tcp open  nlockmgr   1-4 (RPC #100021)
57501/tcp open  mountd     1-3 (RPC #100005)
```
