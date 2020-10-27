---
title: Vulnhub - Hemisphere Lynx
description: My writeup on Hemisphere Lynx box.
categories:
 - vulnhub
tags: vulnhub cewl hydra ssh base64
---

![](https://i.imgur.com/Dayq2fI.png)

## Box Stats

| Box Info      | Details       | 
| ------------- |:-------------:| 
| Box Name :    | **Hemisphere Lynx**  | 
| Series :      | **Hemisphere**         |
| Difficulty :  | **Easy**             |   
| Release Date :| **1 Oct 2020**      |    
| OS :          | **GNU/Linux**        |   
| Maker :       | [d4t4s3c](https://github.com/d4t4s3c){:target="_blank"}     | 
| Download :    | [Hemisphere Lynx](https://www.vulnhub.com/entry/hemisphere-lynx,577/){:target="_blank"}      | 

## Summary

This was a really easy box, pretty basic stuff perfect to pass your time. Let's pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.101                                                                                                                                                               130 ↵
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-27 20:32 EET
Nmap scan report for lynx.zte.com.cn (192.168.1.101)
Host is up (0.00041s latency).
Not shown: 65530 closed ports
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 08:00:27:8F:D2:91 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.39 seconds
$ nmap -p 21,22,80,139,445 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-27 20:32 EET
Nmap scan report for lynx.zte.com.cn (192.168.1.101)
Host is up (0.00047s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 26:21:06:43:f3:27:b0:2f:df:eb:37:c0:26:d7:58:2a (RSA)
|   256 cd:a2:e4:63:31:78:79:a1:56:1d:1d:bd:85:ee:6b:fb (ECDSA)
|_  256 dd:bc:7e:1d:a3:ad:ff:aa:1a:3f:d3:68:a4:42:ea:1b (ED25519)
80/tcp  open  http        Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title:  Lynx 
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
```
 
Let's start the enumeration with port 80, i don't understand the language of the text but the box desc says "Brute Forze" so we can simply generate a custom wordlist with `cewl` and run a brute force on SSH.

## Shell as johannes

```
$ cewl http://$ip/ | tee wordlist.txt
```

```
$ ssh johannes@$ip                                                                                                                                                               255 ↵
johannes@192.168.1.101's password: 

johannes@Lynx:~$ whoami;id
johannes
uid=1000(johannes) gid=1000(johannes) grupos=1000(johannes),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth)
```

## Shell as root


