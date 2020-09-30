---
title: Vulnhub - UnDiscovered
description: My writeup on UnDiscovered box.
categories:
 - vulnhub
tags: vulnhub nfs
---

![](https://i.imgur.com/mmvSbqs.png)

You can find the machine there > [UnDiscovered](https://www.vulnhub.com/entry/undiscovered-101,550/){:target="_blank"}

## Summary

This was one of the best vulnhub box, thanks to [@aniqfakhrul](https://twitter.com/aniqfakhrul){:target="_blank"} & [@h0j3n](https://twitter.com/h0j3n){:target="_blank"} for this awesome box! Let’s pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.5                                     
$ nmap -p- -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-30 21:01 EEST
Nmap scan report for undiscovered.thm (192.168.1.5)
Host is up (0.00044s latency).
Not shown: 65530 closed ports
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:76:81:49:50:bb:6f:4f:06:15:cc:08:88:01:b8:f0 (RSA)
|   256 2b:39:d9:d9:b9:72:27:a9:32:25:dd:de:e4:01:ed:8b (ECDSA)
|_  256 2a:38:ce:ea:61:82:eb:de:c4:e0:2b:55:7f:cc:13:bc (ED25519)
80/tcp    open  http     Apache httpd 2.4.18
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
111/tcp   open  rpcbind  2-4 (RPC #100000)
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
|   100021  1,3,4      34047/tcp6  nlockmgr
|   100021  1,3,4      44886/tcp   nlockmgr
|   100021  1,3,4      51937/udp   nlockmgr
|   100021  1,3,4      55740/udp6  nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
2049/tcp  open  nfs      2-4 (RPC #100003)
44886/tcp open  nlockmgr 1-4 (RPC #100021)
```

Lot of interesting stuff, let's enumerate port 80 first.
