---
title: Vulnhub - The Planets Mercury
description: My writeup on The Planets Mercury box.
categories:
 - vulnhub
tags: vulnhub sqlmap sqli sudo
---

![](https://i.pinimg.com/originals/f0/95/97/f095972de1a5052f7568d673edd66a46.jpg)

You can find the machine there > [The Planets Mercury](https://www.vulnhub.com/entry/the-planets-mercury,544/){:target="_blank"}

## Summary

This machine is quite easy, i have to admit that i stuck on privilege escalation for a while. Starting off with a sqlmap scan on the page we can grab some creds and brute force ssh with them, this will give us a low-privilege shell on the box. Second privesc is really easy we just have to decode some base64 data & the final privesc to root is tricky we have to think smart to exploit it.
Let's start! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.19
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-13 20:22 EEST
Nmap scan report for mercury.zte.com.cn (192.168.1.19)
Host is up (0.00016s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy
MAC Address: 08:00:27:C8:E8:0A (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.23 seconds
$ nmap -p 22,8080 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-13 20:22 EEST
Nmap scan report for mercury.zte.com.cn (192.168.1.19)
Host is up (0.00059s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
8080/tcp open  http-proxy WSGIServer/0.2 CPython/3.8.2
```

Let's start the enumeration on port `8080`, i couldn't find something interesting, only this message:

`Hello. This site is currently in development please check back later.`

On `/robots.txt` too:

```
User-agent: * 
Disallow: /
```

Next step is to run a `gobuster` scan on it, but i didn't find anything:

```
$ gobuster dir -q -u http://$ip:8080/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php,txt,html -o gobuster.txt
/robots.txt (Status: 200)
```

Anyway, i was testing different things on URL and i noticed that on the `Page not found (404)` error provide us a path:

```
Using the URLconf defined in mercury_proj.urls, Django tried these URL patterns, in this order:

[name='index']
robots.txt [name='robots']
mercuryfacts/
```

I browsed `/mercuryfacts` path & i noticed this message `Fact id: 1. (('Mercury does not have any moons or rings.',),)` under `Mercury Facts` link. Voilà! We can test for SQL Injection. Let's fire up sqlmap! :fire:

## Shell as 





