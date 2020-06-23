---
title: Vulnhub - EnuBox Mattermost
description: My writeup on EnuBox Mattermost box.
categories:
 - vulnhub
tags: vulnhub udp mattermost tftp ftp ssh
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

We got some creds! `admin:ComplexPassword0!` we can use them under port `8065`, there Mattermost is running an open-source chat service & we're in!

After some enumeration under `plugin marketplace`, i noticed the `zoom` plugin has a weird URL :

![](https://i.imgur.com/gUCqB6k.png)

I used inspect element to copy the link and it gives us some creds for `ftp` : 

```
$ curl http://$ip/JK94vsNKAns6HBkG/AxRt6LwuA7A6N4gk/index.html
Hello Admin,

FTP credentials help you edit, transfer and delete files from your site. This is why it's important to keep these credentials handy.

FTP Credentials: ftpuser / ftppassword

Make sure to keep these to yourself.
```

## Shell as mattermost

After we login into, we can see a user `mattermost` with a `Welcome!!!` message.

```
$ ftp $ip
Connected to 192.168.1.6.
220 (vsFTPd 3.0.3)
Name (192.168.1.6:root): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Jan 05 14:11 .
drwxr-xr-x    4 ftp      ftp          4096 Jan 05 13:59 ..
-rw-r--r--    1 ftp      ftp           220 Jan 05 13:59 .bash_logout
-rw-r--r--    1 ftp      ftp          3771 Jan 05 13:59 .bashrc
-rw-r--r--    1 ftp      ftp           807 Jan 05 13:59 .profile
-rw-r--r--    1 ftp      ftp          8980 Jan 05 13:59 examples.desktop
drwxr-xr-x    3 ftp      ftp          4096 Jan 05 14:11 users
226 Directory send OK.
ftp> cd users
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jan 05 14:11 mattermost
226 Directory send OK.
ftp> cd mattermost
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            11 Jan 05 14:11 message
226 Directory send OK.
ftp> get message
local: message remote: message
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for message (11 bytes).
226 Transfer complete.
11 bytes received in 0.00 secs (9.8643 kB/s)
ftp> exit
221 Goodbye.
$ cat message 
Welcome!!!
```

I tried them `mattermost:Welcome!!!` with SSH and i got in!

```
$ ssh mattermost@$ip
mattermost@192.168.1.6's password: 
mattermost@ubuntu:~$ whoami
mattermost
```

## mattermost -> root

under `/Desktop` we can see this file : 

```
mattermost@ubuntu:~/Desktop$ ls -la
total 24
drwxr-xr-x  2 mattermost mattermost 4096 Jan  2 15:37 .
drwxr-xr-x 18 mattermost mattermost 4096 Jan  6 01:13 ..
-rw-r--r--  1 mattermost mattermost  233 Jan  2 15:37 README.md
-rwsr-xr-x  1 root       root       8584 Jan  2 15:26 secret
```

Let's do some basic reverse engineering on it.

reverse engineering = understand what a binary does, when there is no source code available.

Let's do some binary enumeration : 

When we run it ask from us to enter the "secret key", so we have to find out the secret key.

```
mattermost@ubuntu:~/Desktop$ ./secret 
Hello Admin, Please enter the secret key:
1337
Your is either invalid or expired
```

It's a 64bit ELF.

```
mattermost@ubuntu:~/Desktop$ file secret 
secret: setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=199fcaacbd26c0c5a26dd1afb01b4ebfcfaa4d18, not stripped
```

If we run strings on it we can understand that if we enter the right secret key will give us a shell.

```
mattermost@ubuntu:~/Desktop$ strings secret 
..junk data..
Hello Admin, Please enter the secret key:
/bin/bash
Your is either invalid or expired
;*3$"
..junk data..
```

Let's copy this file in our box, and let's try to decompile it using IDA Pro.

decompile = takes the machine code and reverts it back to the main language.

```
$ scp mattermost@$ip:/home/mattermost/Desktop/secret .
mattermost@192.168.1.6's password: 
secret                                        100% 8584     9.3MB/s   00:00
```

Let's open it now with IDA Pro, to decompile do View -> Open subviews -> Pseudocode and we can see this pseudocode gives us the key : 

![](https://i.imgur.com/dJgsQFL.png)

Let's enter it and get root shell!

```
mattermost@ubuntu:~/Desktop$ ./secret 
Hello Admin, Please enter the secret key:
62535
root@ubuntu:~/Desktop# whoami;id
root
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),46(plugdev),116(lpadmin),126(sambashare),130(ftp),1000(mattermost)
```

Was an awesome box, that teaches us how important enumeration is!



