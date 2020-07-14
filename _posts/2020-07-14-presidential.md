---
title: Vulnhub - Presidential
description: My writeup on Presidential box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://www.history.com/.image/ar_16:9%2Cc_fill%2Ccs_srgb%2Cfl_progressive%2Cg_faces:center%2Cq_auto:good%2Cw_768/MTY3MTc2NDg2OTU5MjYxMDM2/presidential-elections-gettyimages-78679210.jpg)

Hi all, this was a really interesting and "hard" box. Took me around 10 hours to pwn it. Let's pwn it!

You can find the machine there > [Presidential](https://www.vulnhub.com/entry/presidential-1,500/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.14
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-14 15:02 EEST
Nmap scan report for votenow.local (192.168.1.14)
Host is up (0.00010s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE
80/tcp   open  http
2082/tcp open  infowave
MAC Address: 00:0C:29:A1:D7:E1 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 2.24 seconds
$ nmap -p 80,2082 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-14 15:04 EEST
Nmap scan report for votenow.local (192.168.1.14)
Host is up (0.00057s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.5.38)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.5.38
|_http-title: Ontario Election Services &raquo; Vote Now!
2082/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 06:40:f4:e5:8c:ad:1a:e6:86:de:a5:75:d0:a2:ac:80 (RSA)
|   256 e9:e6:3a:83:8e:94:f2:98:dd:3e:70:fb:b9:a3:e3:99 (ECDSA)
|_  256 66:a8:a1:9f:db:d5:ec:4c:0a:9c:4d:53:15:6c:43:6c (ED25519)
MAC Address: 00:0C:29:A1:D7:E1 (VMware)
```

When we visit the website we can see this in top left corner : `contact@votenow.local` That's probably the domain name. Since the anatomy of an email address is this:

```
contact@votenow.local
  ^          ^
 user    domain name
```

Let's add it to `/etc/hosts`:

`192.168.1.14    votenow.local`

Let's run `gobuster` on it now.

```
$ gobuster dir -q -u http://$ip/ -w $dir_medium -x php,txt,html -o gobuster.txt
/index.html (Status: 200)
/about.html (Status: 200)
/assets (Status: 301)
/config.php (Status: 200)
```

Nothing interesting. `/config.php` is empty. Here i stuck for lot of hours then i tried to brute force extensions but `gobuster` doesnt support extenstion file. Only `wfuzz` can help here:



