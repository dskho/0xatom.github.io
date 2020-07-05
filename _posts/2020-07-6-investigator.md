---
title: Vulnhub - Investigator
description: My writeup on Investigator box.
categories:
 - vulnhub
tags: vulnhub android
---

![](https://hackersfun.com/wp-content/uploads/2019/03/evil-android.jpg)

Hi all, firstly i didnt want to try this box because i have no idea how to pentest/hack an android box. In real life i don't even use/have a smart phone but then i say to myself it's a great way to learn something new, so let's pwn it!

You can find the machine there > [Investigator](https://www.vulnhub.com/entry/investigator-1,504/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.10
$ nmap -sC -sV -p- -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-06 01:52 EEST
Nmap scan report for android-25abe18209db8058.zte.com.cn (192.168.1.10)
Host is up (0.00025s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
5555/tcp  open  adb     Android Debug Bridge device (name: android_x86; model: VMware Virtual Platform; device: x86)
8080/tcp  open  http    PHP cli server 5.5 or later
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Welcome To  UnderGround Sector
22000/tcp open  ssh     Dropbear sshd 2014.66 (protocol 2.0)
| ssh-hostkey: 
|   2048 19:e2:9e:6c:c6:8d:af:4e:86:7c:3b:60:91:33:e1:85 (RSA)
|_  521 46:13:43:49:24:88:06:85:6c:75:93:73:b5:1d:8f:28 (ECDSA)
MAC Address: 00:0C:29:37:42:7C (VMware)
Service Info: OSs: Android, Linux; CPE: cpe:/o:linux:linux_kernel
```
