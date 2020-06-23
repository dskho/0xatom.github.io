---
title: Vulnhub - EnuBox Mattermost
description: My writeup on EnuBox Mattermost box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://www.pngitem.com/pimgs/m/31-312481_mattermost-icon-png-transparent-png.png)

Hi all, that was a really awesome box! Let's pwn it!

You can find the machine there > [EnuBox: Mattermost](https://www.vulnhub.com/entry/enubox-mattermost,414/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.13
$ nmap -sC -sV -p- -oN nmaptcp $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-23 20:27 EEST
Nmap scan report for ubuntu.zte.com.cn (192.168.1.13)
Host is up (0.00063s latency).
Not shown: 65530 closed ports
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.1.16
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh           OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e9:8b:e3:46:0e:c1:7f:a2:1a:c3:df:9d:46:54:ad:1c (RSA)
|   256 ff:5b:25:68:09:f5:45:2b:14:68:66:e0:ce:00:27:b3 (ECDSA)
|_  256 bb:de:d2:db:03:b7:5c:cf:d7:3b:b7:21:65:21:5d:e3 (ED25519)
80/tcp   open  http          Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Sorry, This Page Can&#39;t Be Accessed
3389/tcp open  ms-wbt-server xrdp
8065/tcp open  unknown
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest, SSLSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Accept-Ranges: bytes
|     Cache-Control: no-cache, max-age=31556926, public
|     Content-Length: 3657
|     Content-Security-Policy: frame-ancestors 'self'; script-src 'self' cdn.segment.com/analytics.js/
|     Content-Type: text/html; charset=utf-8
|     Last-Modified: Tue, 23 Jun 2020 15:26:50 GMT
|     X-Frame-Options: SAMEORIGIN
|     X-Request-Id: 4wgq63s967gxbjca4oqryrjyey
|     X-Version-Id: 5.18.0.5.18.0.12da442d30d70d8726b0a9761a350d5a.false
|     Date: Tue, 23 Jun 2020 17:27:23 GMT
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <meta name='viewport' content='width=device-width, initial-scale=1, maximum-scale=1, user-scalable=0'>
|     <meta name='robots' content='noindex, nofollow'>
|     <meta name='referrer' content='no-referrer'>
|     <title>Mattermost</title>
|     <meta name='mobile-web-app-capable' content='yes'>
|     <meta name='application-name' content='Mattermost'>
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Date: Tue, 23 Jun 2020 17:27:23 GMT
|_    Content-Length: 0
MAC Address: 00:0C:29:81:A9:13 (VMware)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

You may notice the `nmaptcp`, that's because we have to run a UDP scan as well. Note : UDP scan is slower than TCP.

```
$ nmap -sU $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-23 22:23 EEST
Nmap scan report for ubuntu.zte.com.cn (192.168.1.6)
Host is up (0.00054s latency).
Not shown: 996 closed ports
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
69/udp   open|filtered tftp
631/udp  open|filtered ipp
5353/udp open|filtered zeroconf
```

`tftp` seems interesting, few words about `tftp`.

TFTP (Trivial File Transfer Protocol) it is simpler than FTP, does file transfer between client and server but does not provide user authentication.

If we try to connect, we can see that we can't list the files.

```
$ tftp $ip
tftp> ?
Commands may be abbreviated.  Commands are:

connect 	connect to remote tftp
mode    	set file transfer mode
put     	send file
get     	receive file
quit    	exit tftp
verbose 	toggle verbose mode
trace   	toggle packet tracing
status  	show current status
binary  	set mode to octet
ascii   	set mode to netascii
rexmt   	set per-packet retransmission timeout
timeout 	set total retransmission timeout
?       	print help information
tftp> ls
?Invalid command
```

Now if we visit the webpage we can see this :

![](https://i.imgur.com/aWPbryI.png)

We can try to download the `README.md` file from `tftp`.

```
tftp> get README.md
Received 65 bytes in 0.0 seconds
tftp> quit
$ cat README.md 
Hello Admin,

Please use the following key: ComplexPassword0!
```

We got some creds! `admin:ComplexPassword0!`
