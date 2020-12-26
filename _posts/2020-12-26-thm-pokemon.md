---
title: TryHackMe - Gotta Catch'em All!
description: My writeup on Gotta Catch'em All! box.
categories:
 - tryhackme
tags: tryhackme curl cyberchef hex rot14 grep find base64
---

![](https://i.imgur.com/kDEBdrm.png)

## Box Stats

| Box Info      | Details       |
| ------------- |:-------------:|
| Box Name :    | **Gotta Catch'em All!**  |
| Difficulty :  | **Easy**             |   
| Play :    | [Gotta Catch'em All!](https://tryhackme.com/room/pokemon){:target="_blank"}      |
| Recommended : | Yes :heavy_check_mark:      |

## Summary

Hello all, this box requires basic linux skills. It's all about finding 4 flags, let's start!

## Enumeration/Reconnaissance

```
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-26 19:50 EET
Warning: 10.10.124.228 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.124.228 (10.10.124.228)
Host is up (0.48s latency).
Not shown: 38306 filtered ports, 27227 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 61.06 seconds

$ nmap -p 22,80 -sC -sV -oN nmap/pokemon.thm $ip
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-26 19:53 EET
Nmap scan report for 10.10.124.228 (10.10.124.228)
Host is up (0.20s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 58:14:75:69:1e:a9:59:5f:b2:3a:69:1c:6c:78:5c:27 (RSA)
|   256 23:f5:fb:e7:57:c2:a5:3e:c2:26:29:0e:74:db:37:c2 (ECDSA)
|_  256 f1:9b:b5:8a:b9:29:aa:b6:aa:a2:52:4a:6e:65:95:c5 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Can You Find Them All?
```

Once we visit the webpage we can see the apache2 default page, checking the source code we find some credentials:

```
$ curl -s http://$ip/

...data...

        </div>
        <TRY_DO_IT_ALONE>:<TRY_DO_IT_ALONE>
        	<!--(Check console for extra surprise!)-->
      </div>
```

Tried them with SSH and it worked! Now let's start the flag hunt! :D

```
$ ssh pokemon@$ip

pokemon@root:~$ whoami;id
pokemon
uid=1000(pokemon) gid=1000(pokemon) groups=1000(pokemon),4(adm),24(cdrom),30(dip),46(plugdev),113(lpadmin),128(sambashare)
```

## Find the Grass-Type Pokemon

Under the desktop folder, a zip exists. Zip contains the grass-type flag but it's hex.

```
pokemon@root:~/Desktop$ ls -la
total 12
drwxr-xr-x  2 pokemon pokemon 4096 Jun 24  2020 .
drwxr-xr-x 19 pokemon pokemon 4096 Dec 26 12:50 ..
-rw-rw-r--  1 pokemon pokemon  383 Jun 22  2020 P0kEmOn.zip
pokemon@root:~/Desktop$ unzip -q P0kEmOn.zip
pokemon@root:~/Desktop$ cd P0kEmOn/
pokemon@root:~/Desktop/P0kEmOn$ ls
grass-type.txt
pokemon@root:~/Desktop/P0kEmOn$ cat grass-type.txt
50 6f TRY_DO_IT_ALONE 72 7d
```

Let's decode it:

```
pokemon@root:~/Desktop/P0kEmOn$ cat grass-type.txt | xxd -r -p
TRY_DO_IT_ALONE
```

## Find the Water-Type Pokemon

Under `/var/www/html` the water-type flag exists, but it's encoded.

```
pokemon@root:/var/www/html$ ls -la
total 24
drwxr-xr-x 2 root    root  4096 Jun 22  2020 .
drwxr-xr-x 3 root    root  4096 Jun 22  2020 ..
-rw-r--r-- 1 root    root 11217 Jun 24  2020 index.html
-rw-r--r-- 1 pokemon root    24 Jun 22  2020 water-type.txt
pokemon@root:/var/www/html$ cat water-type.txt
TRY_DO_IT_ALONE
```

It's rot-14, i used [CyberChef](https://gchq.github.io/CyberChef/){:target="_blank"} to decode it:

![](https://i.imgur.com/4EIijJd.png)

## Find the Fire-Type Pokemon

For fire-type flag i used `find`, Since all flags format is `something-type.txt`.

```
pokemon@root:/var/www/html$ find / -type f -name "*-type.txt" 2>/dev/null
/var/www/html/water-type.txt
/etc/why_am_i_here?/fire-type.txt
```

```
pokemon@root:/var/www/html$ cat /etc/why_am_i_here\?/fire-type.txt | base64 -d
TRY_DO_IT_ALONE
```

## Who is Root's Favorite Pokemon?

This one took me sometime to figure it out.. under `/home` the flag exists but we can't read it:

```
pokemon@root:/home$ ls -la
total 20
drwxr-xr-x  4 root    root    4096 Jun 22  2020 .
drwxr-xr-x 24 root    root    4096 Aug 11 11:07 ..
drwx------  6 root    root    4096 Jun 24  2020 ash
drwxr-xr-x 19 pokemon pokemon 4096 Dec 26 12:50 pokemon
-rwx------  1 ash     root       8 Jun 22  2020 roots-pokemon.txt
pokemon@root:/home$ cat roots-pokemon.txt
cat: roots-pokemon.txt: Permission denied
```

Only user `ash` can read it, i used `grep` to search for his password:

```
pokemon@root:~$ grep -r "ash" . 2>/dev/null
...data...
./Videos/Gotta/Catch/Them/ALL!/Could_this_be_what_Im_looking_for?.cplusplus:	std::cout << "ash : TRY_DO_IT_ALONE"
```

```
pokemon@root:~$ su - ash
Password:
$ cat /home/roots-pokemon.txt
TRY_DO_IT_ALONE
```

Sorry pokemonz.

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"}

Until next time keep pwning hard! :fire:
