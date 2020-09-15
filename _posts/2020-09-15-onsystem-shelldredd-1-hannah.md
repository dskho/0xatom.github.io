---
title: Vulnhub - OnSystem ShellDredd #1 Hannah
description: My writeup on OnSystem ShellDredd #1 Hannah box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://i.imgur.com/LeUKXhe.png)

You can find the machine there > [OnSystem ShellDredd #1 Hannah](https://www.vulnhub.com/entry/onsystem-shelldredd-1-hannah,545/){:target="_blank"}

## Summary

This box was a really easy one, starting off by login into FTP anonymous and grab hannah's SSH private key this will give us a low low-privilege shell. Rooting the box is a bit tricky because the cpulimit SUID doesn't allow us to spawn a root shell directly. Let's pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.7 
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-15 12:59 EEST
Nmap scan report for shelldredd.zte.com.cn (192.168.1.7)
Host is up (0.00016s latency).
Not shown: 65533 closed ports
PORT      STATE SERVICE
21/tcp    open  ftp
61000/tcp open  unknown
MAC Address: 08:00:27:F6:95:7A (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.40 seconds
$ nmap -p 21,61000 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-15 12:59 EEST
Nmap scan report for shelldredd.zte.com.cn (192.168.1.7)
Host is up (0.00038s latency).

PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.1.18
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
61000/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 59:2d:21:0c:2f:af:9d:5a:7b:3e:a4:27:aa:37:89:08 (RSA)
|   256 59:26:da:44:3b:97:d2:30:b1:9b:9b:02:74:8b:87:58 (ECDSA)
|_  256 8e:ad:10:4f:e3:3e:65:28:40:cb:5b:bf:1d:24:7f:17 (ED25519)
```
