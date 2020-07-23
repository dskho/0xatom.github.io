---
title: SuNiNaTaS - Forensic 14
description: My writeup on forensic 14 challenge.
categories:
 - suninatas
tags: suninatas john shadow
---

![](https://i1.daumcdn.net/thumb/C264x200/?fname=https://t1.daumcdn.net/cfile/tistory/99DE7733599504E81D)

Hi all, let's start with forensic challenges.

You can start pwning there > [SuNiNaTaS](http://suninatas.com/){:target="_blank"}

## Solution

Asks if we know password of suninatas and gives us a file to download, let's download it and untar it:

![](https://i.imgur.com/pYEQD5S.png)

```
$ wget -q http://suninatas.com/challenge/web14/evidence.tar
$ tar -xvf evidence.tar 
passwd
shadow
$ cat shadow | tail -1
suninatas:$6$QlRlqGhj$BZoS9PuMMRHZZXz1Gde99W01u3kD9nP/zYtl8O2dsshdnwsJT/1lZXsLar8asQZpqTAioiey4rKVpsLm/bqrX/:15427:0:99999:7:::
```

We can see user suninatas and his encrypted password `$6$ -> SHA-512`, let's crack it with john.

```
$ cat shadow | tail -1 > passwordcrack
$ john --wordlist=/usr/share/wordlists/rockyou.txt passwordcrack 
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
iloveu1          (suninatas)
```

Authkey: `iloveu1`

![](https://i.imgur.com/1LvR7Ra.png)
