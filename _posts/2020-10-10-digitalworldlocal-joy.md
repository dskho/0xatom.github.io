---
title: Vulnhub - digitalworld.local JOY
description: My writeup on digitalworld.local JOY box.
categories:
 - vulnhub
tags: vulnhub udp
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

Ton of services, let's start with FTP since we have anonymous allowed. Once we connect we can see 2 directories `download/upload` the `upload` directory has lot of stuff in. One file the `directory` has the "copy" of Patrick's directory. Let's download it & analyze it:

```
$ ftp $ip
Connected to 192.168.1.19.
220 The Good Tech Inc. FTP Server
Name (192.168.1.19:root): anonymous
331 Anonymous login ok, send your complete email address as your password
Password:
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxrwxr-x   2 ftp      ftp          4096 Jan  6  2019 download
drwxrwxr-x   2 ftp      ftp          4096 Jan 10  2019 upload
226 Transfer complete
ftp> cd upload
250 CWD command successful
ftp> ls -la
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxrwxr-x   2 ftp      ftp          4096 Jan 10  2019 .
drwxr-x---   4 ftp      ftp          4096 Jan  6  2019 ..
-rwxrwxr-x   1 ftp      ftp          2312 Oct  9 20:33 directory
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_armadillo
-rw-rw-rw-   1 ftp      ftp            25 Jan  6  2019 project_bravado
-rw-rw-rw-   1 ftp      ftp            88 Jan  6  2019 project_desperado
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_emilio
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_flamingo
-rw-rw-rw-   1 ftp      ftp             7 Jan  6  2019 project_indigo
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_komodo
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_luyano
-rw-rw-rw-   1 ftp      ftp             8 Jan  6  2019 project_malindo
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_okacho
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_polento
-rw-rw-rw-   1 ftp      ftp            20 Jan  6  2019 project_ronaldinho
-rw-rw-rw-   1 ftp      ftp            55 Jan  6  2019 project_sicko
-rw-rw-rw-   1 ftp      ftp            57 Jan  6  2019 project_toto
-rw-rw-rw-   1 ftp      ftp             5 Jan  6  2019 project_uno
-rw-rw-rw-   1 ftp      ftp             9 Jan  6  2019 project_vivino
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_woranto
-rw-rw-rw-   1 ftp      ftp            20 Jan  6  2019 project_yolo
-rw-rw-rw-   1 ftp      ftp           180 Jan  6  2019 project_zoo
-rwxrwxr-x   1 ftp      ftp            24 Jan  6  2019 reminder
226 Transfer complete
ftp> get directory
local: directory remote: directory
200 PORT command successful
150 Opening BINARY mode data connection for directory (2312 bytes)
226 Transfer complete
2312 bytes received in 0.00 secs (13.8673 MB/s)
ftp> exit
```

`directory` file contents:

```
Patrick's Directory

total 116
drwxr-xr-x 18 patrick patrick 4096 Oct 10 04:30 .
drwxr-xr-x  4 root    root    4096 Jan  6  2019 ..
-rw-r--r--  1 patrick patrick    0 Oct 10 04:25 7pYsPPfWWJGuEcxGzrCIaan0R5PZxasS.txt
-rw-------  1 patrick patrick  185 Jan 28  2019 .bash_history
-rw-r--r--  1 patrick patrick  220 Dec 23  2018 .bash_logout
-rw-r--r--  1 patrick patrick 3526 Dec 23  2018 .bashrc
drwx------  7 patrick patrick 4096 Jan 10  2019 .cache
drwx------ 10 patrick patrick 4096 Dec 26  2018 .config
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Desktop
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Documents
drwxr-xr-x  3 patrick patrick 4096 Jan  6  2019 Downloads
-rw-r--r--  1 patrick patrick   24 Oct 10 04:30 dvPzl5ksSoFc1f4s5wB9aJSE4w5Ckbnp0IxVAOPveiqONxpUuwle0GktTWY9ZY9S.txt
-rw-r--r--  1 patrick patrick   24 Oct 10 04:25 eLkugh9zZn90nDbkTWqZATIXNkuEnS3BDEDaK21jqkA7TPB1sleyH447hyXe2Go9.txt
drwx------  3 patrick patrick 4096 Dec 26  2018 .gnupg
-rwxrwxrwx  1 patrick patrick    0 Jan  9  2019 haha
-rw-------  1 patrick patrick 8532 Jan 28  2019 .ICEauthority
drwxr-xr-x  3 patrick patrick 4096 Dec 26  2018 .local
drwx------  5 patrick patrick 4096 Dec 28  2018 .mozilla
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Music
drwxr-xr-x  2 patrick patrick 4096 Jan  8  2019 .nano
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Pictures
-rw-r--r--  1 patrick patrick  675 Dec 23  2018 .profile
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Public
d---------  2 root    root    4096 Jan  9  2019 script
-rw-r--r--  1 patrick patrick    0 Oct 10 04:20 SOqnxq7l9smzwdSfatyfFdnzXk9zyC8g.txt
drwx------  2 patrick patrick 4096 Dec 26  2018 .ssh
-rw-r--r--  1 patrick patrick    0 Jan  6  2019 Sun
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Templates
-rw-r--r--  1 patrick patrick    0 Jan  6  2019 .txt
-rw-r--r--  1 patrick patrick  407 Jan 27  2019 version_control
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Videos
-rw-r--r--  1 patrick patrick   24 Oct 10 04:20 wKtEXVOvDSqfqlkUVSMz6lbV21bwBqjZv8VajeyO2m6yXkWfZ7wTLCq8ZAkLdCxb.txt
-rw-r--r--  1 patrick patrick    0 Oct 10 04:30 Xuc1ngPttrOz9HoFPek6OjXDHVhLSfC6.txt

You should know where the directory can be accessed.

Information of this Machine!

Linux JOY 4.9.0-8-amd64 #1 SMP Debian 4.9.130-2 (2018-10-27) x86_64 GNU/Linux
```



