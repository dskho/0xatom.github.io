---
title: SuNiNaTaS - Forensic 19
description: My writeup on forensic 19 challenge.
categories:
 - suninatas
tags: suninatas binary caesar
---

![](https://i1.daumcdn.net/thumb/C264x200/?fname=https://t1.daumcdn.net/cfile/tistory/99DE7733599504E81D)

Hi all, let's continue with forensic.

You can start pwning there > [SuNiNaTaS](http://suninatas.com/){:target="_blank"}

## Solution

We have another cipher:

![](https://i.imgur.com/sK11B8S.png)

Obviously is binary, let's decode it with [cyberchef](https://gchq.github.io/CyberChef/){:target="_blank"}

`NVCTFDV KF JLEZERKRJ REU KFURP ZJ R XFFU URP REU RLKYBVP ZJ GCRZUTKWZJMVIPYRIU`

Now using a [cipher identifier](https://www.boxentriq.com/code-breaking/cipher-identifier){:target="_blank"} tell us that this is caesar cipher, let's decode it using this [tool](https://www.dcode.fr/caesar-cipher){:target="_blank"}

`WELCOME TO SUNINATAS AND TODAY IS A GOOD DAY AND AUTHKEY IS PLAIDCTFISVERYHARD`

Authkey: `PLAIDCTFISVERYHARD`

![](https://i.imgur.com/fPQ9ywa.png)
