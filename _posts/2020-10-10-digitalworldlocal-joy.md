---
title: Vulnhub - digitalworld.local JOY
description: My writeup on digitalworld.local JOY box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://i.imgur.com/h3KUnSs.png)

## Box Stats

| Box Info      | Details       | 
| ------------- |:-------------:| 
| Box Name :    | **digitalworld.local JOY**  | 
| Series :      | **digitalworld.local**           |
| Difficulty :  | **Medium/Hard**      |   
| Release Date :| **31 Mar 2019**      |    
| OS :          | **GNU/Linux**        |   
| Maker :       | **Donavan**      | 
| Download :    | [digitalworld.local JOY](https://www.vulnhub.com/entry/digitalworldlocal-joy,298/){:target="_blank"}      | 

## Summary

digitalworld.local JOY was a pretty good & realistic box that i really enjoyed pwning. Let’s pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

TCP Scan:

```
$ ip=192.168.1.19
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 23:19 EEST
Nmap scan report for joy.zte.com.cn (192.168.1.19)
Host is up (0.00057s latency).
Not shown: 65523 closed ports
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
110/tcp open  pop3
139/tcp open  netbios-ssn
143/tcp open  imap
445/tcp open  microsoft-ds
465/tcp open  smtps
587/tcp open  submission
993/tcp open  imaps
995/tcp open  pop3s
MAC Address: 08:00:27:6D:C5:DC (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.35 seconds
$ nmap -p 21,22,25,80,110,139,143,445,465,587,993,995 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 23:20 EEST
Stats: 0:05:17 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 98.96% done; ETC: 23:25 (0:00:01 remaining)
Nmap scan report for joy.zte.com.cn (192.168.1.19)
Host is up (0.00044s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxr-x   2 ftp      ftp          4096 Jan  6  2019 download
|_drwxrwxr-x   2 ftp      ftp          4096 Jan 10  2019 upload
22/tcp  open  ssh         Dropbear sshd 0.34 (protocol 2.0)
25/tcp  open  smtp        Postfix smtpd
|_smtp-commands: JOY.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, 
| ssl-cert: Subject: commonName=JOY
| Subject Alternative Name: DNS:JOY
| Not valid before: 2018-12-23T14:29:24
|_Not valid after:  2028-12-20T14:29:24
|_ssl-date: TLS randomness does not represent time
80/tcp  open  http        Apache httpd 2.4.25 ((Debian))
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2016-07-19 20:03  ossec/
|_
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Index of /
110/tcp open  pop3?
|_ssl-date: TLS randomness does not represent time
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_ssl-date: TLS randomness does not represent time
445/tcp open  netbios-ssn Samba smbd 4.5.12-Debian (workgroup: WORKGROUP)
465/tcp open  smtp        Postfix smtpd
|_smtp-commands: JOY.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, 
| ssl-cert: Subject: commonName=JOY
| Subject Alternative Name: DNS:JOY
| Not valid before: 2018-12-23T14:29:24
|_Not valid after:  2028-12-20T14:29:24
|_ssl-date: TLS randomness does not represent time
587/tcp open  smtp        Postfix smtpd
|_smtp-commands: JOY.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, 
| ssl-cert: Subject: commonName=JOY
| Subject Alternative Name: DNS:JOY
| Not valid before: 2018-12-23T14:29:24
|_Not valid after:  2028-12-20T14:29:24
|_ssl-date: TLS randomness does not represent time
993/tcp open  ssl/imaps?
|_ssl-date: TLS randomness does not represent time
995/tcp open  ssl/pop3s?
|_ssl-date: TLS randomness does not represent time
```

UDP Scan:

```
$ nmap -p- --min-rate 10000 -sU $ip                                               
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 23:26 EEST
Warning: 192.168.1.19 giving up on port because retransmission cap hit (10).
Nmap scan report for joy.zte.com.cn (192.168.1.19)
Host is up (0.00079s latency).
Not shown: 65454 open|filtered ports, 78 closed ports
PORT    STATE SERVICE
123/udp open  ntp
137/udp open  netbios-ns
161/udp open  snmp
```
