---
title: SuNiNaTaS - Misc 12
description: My writeup on misc 12 challenge.
categories:
 - suninatas
tags: suninatas
---

![](https://i1.daumcdn.net/thumb/C264x200/?fname=https://t1.daumcdn.net/cfile/tistory/99DE7733599504E81D)

Hi all, let's start with misc challenges.

You can start pwning there > [SuNiNaTaS](http://suninatas.com/){:target="_blank"}

## Solution

Challenge says we need to login as admin:

![](https://i.imgur.com/uMKbixe.png)

I tried stuff like `admin.asp`, `admin` nothing. Then i had an idea to run `gobuster` on it:

```
$ gobuster dir -q -u http://suninatas.com/ -w /usr/share/dirb/wordlists/common.txt 
/account (Status: 301)
/admin (Status: 301)
/Admin (Status: 301)
/ADMIN (Status: 301)
```

`/admin` is the answer, silly me haha. Once we visit it we can see a QR image let's download it and read it.

```
$ wget -q http://suninatas.com/admin/qrcode.jpg
$ zbarimg qrcode.jpg 
QR-Code:MECARD:N:;TEL:;EMAIL:;NOTE:;URL:http://suninatas.com/admin/admlogin.asp;ADR:;
```

It gaves us a new page: `http://suninatas.com/admin/admlogin.asp` If we check the source code of it we can see `.swf` file:

```html
<embed src="admlogin.swf">
```

`.swf` is an adobe flash file, contains animations and stuff. We have to reverse engineering it, we can use `flasm` disassembler.

```
$ wget -q http://suninatas.com/admin/admlogin.swf
$ flasm -d admlogin.swf

  frame 0
    stop
  end // of frame 0

  defineButton 9

    on overDownToOverUp
      constants 'flashid', 'admin', 'flashpw', 'myadmin!@', 'flashmessage', 'Wrong ID or PW', 'Auth : "Today is a Good day~~~"', 'receipt'  
```

We have the authkey: `Today is a Good day~~~`

![](https://i.imgur.com/8PkXP64.png)

Really good one.




