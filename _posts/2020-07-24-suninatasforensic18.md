---
title: SuNiNaTaS - Forensic 18
description: My writeup on forensic 18 challenge.
categories:
 - suninatas
tags: suninatas base64 cyberchef decimal
---

![](https://i1.daumcdn.net/thumb/C264x200/?fname=https://t1.daumcdn.net/cfile/tistory/99DE7733599504E81D)

Hi all, let's pwn the third forensic challenge.

You can start pwning there > [SuNiNaTaS](http://suninatas.com/){:target="_blank"}

## Solution

It gives us a simple cipher:

![](https://i.imgur.com/dIQt9x1.png)

We can understand that this is base10(decimal 0-9), we can use [cyberchef](https://gchq.github.io/CyberChef/){:target="_blank"} to decode it. It gives us a base64 string:

`VG9kYXkgaXMgYSBnb29kIGRheS4gVGhlIEF1dGhLZXkgaXMgVmVyeVZlcnlUb25nVG9uZ0d1cmkh`

```
$ echo VG9kYXkgaXMgYSBnb29kIGRheS4gVGhlIEF1dGhLZXkgaXMgVmVyeVZlcnlUb25nVG9uZ0d1cmkh | base64 -d
Today is a good day. The AuthKey is VeryVeryTongTongGuri!
```

Authkey: `VeryVeryTongTongGuri!`

![](https://i.imgur.com/wWSyG3p.png)
