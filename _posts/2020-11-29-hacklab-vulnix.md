---
title: Vulnhub - HackLAB Vulnix
description: My writeup on HackLAB Vulnix box.
categories:
 - vulnhub
tags: vulnhub smtp metasploit smtp-user-enum hydra
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

Ton of services, a good practise is to take them one by one. So a good start is the SMTP, SMTP (Simple Mail Transfer Protocol) is mostly used for sending out emails. SMTP allow us to perform username enumeration via the VRFY and EXPN commands.

```
VRFY = The server is asked to verify if a email address or username exists.
EXPN = This SMTP command asks for a confirmation about the identification.
```

I'll show you 2 ways to do that, with `metasploit` & `smtp-user-enum`.

Metasploit (Takes some time):

```
$ service postgresql start; msfconsole -q
msf6 > search name:smtp type:auxiliary

Matching Modules
================

   #  Name                                     Disclosure Date  Rank    Check  Description
   -  ----                                     ---------------  ----    -----  -----------
   ... data ...
   3  auxiliary/scanner/smtp/smtp_enum                          normal  No     SMTP User Enumeration Utility
   ... data ...
   
msf6 > use auxiliary/scanner/smtp/smtp_enum
msf6 auxiliary(scanner/smtp/smtp_enum) > set RHOST 192.168.1.21
RHOST => 192.168.1.21
msf6 auxiliary(scanner/smtp/smtp_enum) > exploit

[*] 192.168.1.21:25       - 192.168.1.21:25 Banner: 220 vulnix ESMTP Postfix (Ubuntu)
[+] 192.168.1.21:25       - 192.168.1.21:25 Users found: , backup, bin, daemon, games, gnats, irc, landscape, libuuid, list, lp, mail, man, messagebus, news, nobody, postfix, postmaster, proxy, sshd, sync, sys, syslog, user, uucp, whoopsie, www-data
[*] 192.168.1.21:25       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

smtp-user-enum (Kali has it pre-installed or you can find [here](https://github.com/pentestmonkey/smtp-user-enum){:target="_blank"}:

```
$ smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t $ip                                                                                 

######## Scan started at Sun Nov 29 14:09:22 2020 #########
192.168.1.21: backup exists
192.168.1.21: bin exists
192.168.1.21: daemon exists
192.168.1.21: games exists
192.168.1.21: gnats exists
192.168.1.21: irc exists
192.168.1.21: landscape exists
192.168.1.21: libuuid exists
192.168.1.21: lp exists
192.168.1.21: list exists
192.168.1.21: man exists
192.168.1.21: mail exists
192.168.1.21: messagebus exists
192.168.1.21: nobody exists
192.168.1.21: news exists
192.168.1.21: postmaster exists
192.168.1.21: postfix exists
192.168.1.21: proxy exists
192.168.1.21: root exists
192.168.1.21: ROOT exists
192.168.1.21: sshd exists
192.168.1.21: sync exists
192.168.1.21: sys exists
192.168.1.21: syslog exists
192.168.1.21: user exists
192.168.1.21: uucp exists
192.168.1.21: whoopsie exists
192.168.1.21: www-data exists
```

We can detect 2 "weird" usernames the `user` & `whoopsie` so we can perform a SSH brute force attack now using `hydra`. A brute force on `user` will give us access. Note: You **_MUST_** use number of connects in parallel(-t 4).




