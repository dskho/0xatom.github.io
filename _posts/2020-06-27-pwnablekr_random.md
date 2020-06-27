---
title: pwnable.kr - random
description: My writeup on random challenge.
categories:
 - pwnablekr
tags: pwnablekr
---

![](https://i.imgur.com/cg6lYs1.png)

Hi all, that was a really interesting challenge.

You can start playing there > [pwnable.kr](http://pwnable.kr/){:target="_blank"}

## Binary/Source Code - Enumeration/Reconnaissance

```
Daddy, teach me how to use random value in programming!

ssh random@pwnable.kr -p2222 (pw:guest)
```

First step we connect to `pwnable.kr` server, after we can see these files :

```
random@pwnable:~$ ls
flag  random  random.c
```

`flag` contains our flag, `random` is the binary that we have to exploit, `random.c` has the vulnerable code that we have to analyze.

```c
#include <stdio.h>

int main(){
	unsigned int random;
	random = rand();	// random value!

	unsigned int key=0;
	scanf("%d", &key);

	if( (key ^ random) == 0xdeadbeef ){
		printf("Good!\n");
		system("/bin/cat flag");
		return 0;
	}

	printf("Wrong, maybe you should try 2^32 cases.\n");
	return 0;
}
```

Note: i dont code C, but i can understand C.

The problem lies here :

```c
random = rand();
```

This code generates bad random numbers, when you call rand() without a seed it uses the value 1 as a default seed. Anyone else on the same machine with the same compiler who calls rand() with a seed of 1 will get the same random number. WOW dangerous.

So let's code a simple C code to grab this random number. You can go under `/tmp` and create a directory.

```c
#include <stdio.h>
int main() {
   unsigned int random;
   random = rand();  
   
   printf("%d\n", random);

   return 0;
}

```

Let's compile it & run it.

```
random@pwnable:/tmp/atooom$ gcc -Wall exploit.c -o exploit
exploit.c: In function ‘main’:
exploit.c:4:13: warning: implicit declaration of function ‘rand’ [-Wimplicit-function-declaration]
    random = rand();  
             ^
random@pwnable:/tmp/atooom$ ./exploit
1804289383
random@pwnable:/tmp/atooom$ ./exploit
1804289383
random@pwnable:/tmp/atooom$ ./exploit
1804289383
```

Now, to grab the flag we have to do this : 

```c
(key ^ random) == 0xdeadbeef 
```

## XOR operator explanation

^ = means XOR operation

For example :

```
0 ^ 0 = 0
0 ^ 1 = 1
1 ^ 0 = 1
1 ^ 1 = 0
```

We can do this with python pretty easy : 

```
>>> 0 ^ 0
0
>>> 1 ^ 0
1
```

## Exploitation






