---
title: ringzer0ctf - SysAdmin Linux
description: My writeup on ringzer0ctf SysAdmin Linux challenges.
categories:
 - ringzer0ctf
tags: ringzer0ctf linux
---

![](https://miro.medium.com/max/1472/1*5M1nixCj5FzRwBMVrU-EkQ.png)

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
FLAG-d325e738fa7d87d4f5607c302b37db20oracle
```

## SysAdmin Part 5
