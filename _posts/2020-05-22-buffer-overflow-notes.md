---
title: Buffer Overflow Notes
description: My Buffer Overflow Notes.
categories:
 - binary exploitation
tags: binary exploitation, bof
---

# Buffer 

`Buffer` -> Buffer is a memory location which is used by a running program. This memory location store temporary data.

Example :

```c
#include <stdio.h>

int main() {
    char username[20];

    printf("Hello. What's your name?\n");
    scanf("%s", &username);
    printf("Hi there, %s", username);
    return 0;
}
```

`char .. [20]` ->  This is where we specify the buffer, it's 20 chars/bytes.

# Buffer Overflow

`Buffer Overflow` -> Buffer Overflow is when a running program attempts to write data outside the buffer

```
U S E R N A M E | 1 2
0 1 2 3 4 5 6 7 | 8 9
[Buffer 8bytes]  [overflow]
```

Example :

```c
#include <stdio.h>

int main () {
   char username[20];

   printf("Enter your name: ");
   scanf("%s", username);

   printf("Hello %s\n", username);
   printf("Program exited normally");
   return(0);
}
```

The buffer for username is 20bytes,  it’s good if username length is less than 20 bytes. But if we enter more than 20 bytes the program will crash.

```bash
$ ./program
Enter your name: atom
Hello atom
Program exited normally
```

Now let's enter 50bytes!

```bash
$ python -c 'print "A" * 50' | ./program
Enter your name: Hello AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Segmentation fault
```

Now we don't see the `program exited normally` but we get a `Segmentation fault` error.

This happened because we entered 30 extra bytes.

`Segmentation fault (segfault)` -> Error caused by accessing memory that does not belong to you.

# gdb

`gdb (GNU Project debugger)` -> Allows you to see what is going on inside the program while it executes.

```bash
$ gdb -q ./program
Reading symbols from /tmp/program...(no debugging symbols found)...done.
(gdb) r
Starting program: /tmp/program
Enter your name: atom
Hello atom
Program exited normally
(gdb)
```

The input "atom" is less than 20 bytes so the program exited normally. Now let's input more than 20bytes.

```bash
(gdb) shell python -c 'print "A" * 50'
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
(gdb) r
Starting program: /tmp/program
Enter your name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Hello AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Program received signal SIGSEGV, Segmentation fault.
0x41414141 in ?? ()
(gdb)
```

Let's input now a hex value & let's look at the registers.

```bash
(gdb) shell python -c 'print "\x11" * 50' > input
(gdb) r < input
Starting program: /tmp/program < input
Enter your name: Hello

Program received signal SIGSEGV, Segmentation fault.
0x11111111 in ?? ()
(gdb) info registers
eax            0x0      0
ecx            0x0      0
edx            0xb7fd9340       -1208118464
ebx            0xb7fd7ff4       -1208123404
esp            0xbffff800       0xbffff800
ebp            0x11111111       0x11111111
esi            0x0      0
edi            0x0      0
eip            0x11111111       0x11111111
eflags         0x210296 [ PF AF SF IF RF ID ]
cs             0x73     115
ss             0x7b     123
ds             0x7b     123
es             0x7b     123
fs             0x0      0
gs             0x33     51
(gdb)
```

`Register` -> Register is a storage area inside the CPU.

We can see most memory addresses are overwritten with 11.

# Buffer Overflow Dangerous ?

A buffer overflow is dangerous when the vulnerable binary is `SUID`, for example we can get a root shell!

# Vulnerable C functions to Buffer Overflow

1) gets

2) scanf

3) sprintf

4) strcpy

# Ways To Print (n) "A" (one-liners)

```python
python -c 'print "A" * 100'
```

```perl
perl -e 'print "A" x 100;'
```

# I/O

Command output as argument

```$ ./binary `cmd_here` ```

`$ ./binary $(cmd_here)`

Command output as input

`$ cmd_here | ./binary`

Use file as input

`$ cmd_here > file`

`$ ./binary < file`

# Check if the binary is vulnerable

```bash
user@protostar:/opt/protostar/bin$ python -c 'print "A" * 100' | ./stack3
Segmentation fault
```

`segfault` confirms that binary is vulnerable to buffer overflow.

# Find out the size of the buffer (number of characters that overwrite the EIP)

```bash
$ ./pattern_create.rb -l 100
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

$ gdb -q ./stack3
Reading symbols from /opt/protostar/bin/stack3...done.
(gdb) r
Starting program: /opt/protostar/bin/stack3 
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
calling function pointer, jumping to 0x63413163

Program received signal SIGSEGV, Segmentation fault.
0x63413163 in ?? ()

$ ./pattern_offset.rb -q 0x63413163
[*] Exact match at offset 64
```
# Get memory address of a function

```bash
$ objdump -d ./binary | grep function_name
```

# Program Memory (Segments)

```
--------------- high address

     Stack      <---- Store functions & local variables

---------------

 Unused Memory  

---------------
 
     Heap       <---- Dynamic Memory

---------------

     .bss       <---- Uninitialized Data

---------------

    .data       <---- Initialized Data

---------------

    .text       <---- Program Code

--------------- low address
```

# Stack

```
+------------+    High Memory
|            |         .
| 0x12345678 |         .
|            |         .
+------------+         .
|            |         .
| 0x87654321 |         .
|            |         .
+------------+         .
                       .
                       .
                       .
                       .
                       .
                       V
                  Low Memory


ESP Register (Extended Stack Pointer) -> Points to top of stack

Stack 2 operations :

1) PUSH -> Push a value into the stack
2) POP  -> Remove a value from the stack
```

# Endianness

Computer CPU stores data in big or little endian format, depending on the CPU architecture.

Examples :

```
+-----------------------+---------------+
|         CPU           |   Endianness  |
+-----------------------+---------------+
| Intel x86 (32 bit)    | Little Endian |
| Intel x86_64 (64 bit) | Little Endian |
| Sun Sparc             | Big Endian    |
+-----------------------+---------------+
```

`Big Endian` -> The most significant byte of the data is placed at the byte with the lowest address.

`Little Endian` -> The least significant byte of the data is placed at the byte with the lowest address.

0x12674592

Big Endian -> `12 67 45 92`

Little Endian -> `92 45 67 12`

Little Endian Python Ways!

```python
>>> from pwn import *
>>> p32(0xdeadbeef)
b'\xef\xbe\xad\xde'
```

```python
>>> def p(x):
      return pack('<L', x)
>>> p(0xdeadbeef)
b'\xef\xbe\xad\xde'
```

# CPU architectures

`CPU(Central Processing Units)` -> processing and executing instructions. 

`memory address` -> memory address is an exact location in RAM used to track where information is stored.

`32-bit or x86 or i686 or i386` -> A 32-bit CPU can store 2³² or 4,294,967,296 memory addresses.

`64-bit or x86_64` -> A 64-bit CPU can store 2⁶⁴ or 18,446,744,073,709,551,616 memory addresses.

Differences :

```
+---------------------------------+---------------------------------------------+
|          32-bit CPU             |                 64-bit CPU                  |
+---------------------------------+---------------------------------------------+
| 64-bit softwares won't work.    | 32-bit softwares will work                  |
|                                 |                                             |
| Need a 32-bit operating system. | It can run on 32 & 64-bit operating system. |
|                                 |                                             |
| Not multi-tasking.              | Performing multi-tasking                    |
+---------------------------------+---------------------------------------------+
```

# gdb cheat sheet

`gdb -q ./binary` -> enter binary in gdb

`set disassembly-flavor intel` -> set assembly syntax into Intel, it’s easier to read

`info functions` -> list binary functions

`disassemble Function/Address` -> gets assembly code

`break *Address` -> breakpoint makes your program to stop

`x/Length-Format E.g. : x/40x $esp` -> displays the memory contents

`r` -> run binary

`c` -> continue execution

`q` -> exit gdb

# Examples

[stack0](https://0xatom.github.io/protostar-stack0)

[stack1](https://0xatom.github.io/protostar-stack1)

[stack2](https://0xatom.github.io/protostar-stack2)

[stack3](https://0xatom.github.io/protostar-stack3)

[stack4](https://0xatom.github.io/protostar-stack4)
