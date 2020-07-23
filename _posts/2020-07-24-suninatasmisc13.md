---
title: SuNiNaTaS - Misc 13
description: My writeup on misc 13 challenge.
categories:
 - suninatas
tags: suninatas
---

![](https://i1.daumcdn.net/thumb/C264x200/?fname=https://t1.daumcdn.net/cfile/tistory/99DE7733599504E81D)

Hi all, let's pwn the second misc challenge.

You can start pwning there > [SuNiNaTaS](http://suninatas.com/){:target="_blank"}

## Solution

Challenge says "key finding", no idea what that means i tried stuff like `key`, `key.asp` but nothing:

![](https://i.imgur.com/E7vOmOY.png)

Comment says:

```html
<!--	Hint : 프로그래머의 잘못된 소스백업 습관 -->
<!--	Hint : The programmer's bad habit of backup source codes -->
```

`backup` source codes.. hm i stuck here for a bit then i tried different extensions and `.zip` worked! :D

```
$ wget -q http://suninatas.com/challenge/web13/web13.zip
$ unzip web13.zip 
Archive:  web13.zip
[web13.zip] whitehack1.jpg password: 
```

Needs a password, let's crack it:

```
$ fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt web13.zip

PASSWORD FOUND!!!!: pw == 7642
$ unzip -P 7642 web13.zip                                      
Archive:  web13.zip
  inflating: whitehack1.jpg          
  inflating: whitehack2.jpg          
  inflating: whitehack3.jpg          
  inflating: whitehack4.jpg          
  inflating: ��+Ӧ���+�4++��-�+�.txt  
```

Now we have to do some stego on all images to grab the keys and put them together. I made an one-liner:

```
$ strings whitehack1.jpg | grep key | cut -d ' ' -f 4 ; strings whitehack2.jpg | grep key | cut -d ' ' -f 4 ; strings whitehack3.jpg | grep key | cut -d ' ' -f 3 ; strings whitehack4.jpg | grep key | cut -d ' ' -f 4
3nda192n
84ed1cae
8abg9295
cf9eda4d
```

We have the authkey: `3nda192n84ed1cae8abg9295cf9eda4d`

![](https://i.imgur.com/1r1h9WU.png)

yay!

