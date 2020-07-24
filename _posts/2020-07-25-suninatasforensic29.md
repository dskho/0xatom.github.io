---
title: SuNiNaTaS - Forensic 29
description: My writeup on forensic 29 challenge.
categories:
 - suninatas
tags: suninatas
---

![](https://i1.daumcdn.net/thumb/C264x200/?fname=https://t1.daumcdn.net/cfile/tistory/99DE7733599504E81D)

Hi all, finally a good and interesting challenge i suggest you to try this out! This challenge took me lot of hours because the OS is korean based and i had troubles with the language.

You can start pwning there > [SuNiNaTaS](http://suninatas.com/){:target="_blank"}

## .egg file enumeration - import the VM

This is the challenge:

![](https://i.imgur.com/0JT2wmF.png)

We have to download a whole windows 7 vitrual machine and import it into VMware and start answering the questions. The flag is the MD5 of all the answers: 

`MD5(Key of Q1+Answer of Q2+Answer of Q3+Key of Q4)`

The file is kinda big `Windows7(SuNiNaTaS) (3,4G)`, took me some time to download it. After download i opened the file into the [HxD hex editor](https://mh-nexus.de/en/hxd/){:target="_blank"} we can notice the egg signature.

![](https://i.imgur.com/ti8lUdg.png)

After some google-fu how can i open this file, i found the answer. EGG file is a compressed archive created by ALZip. You can download it from [here](https://download.cnet.com/ALZip/3000-2250_4-10326198.html){:target="_blank"}, let's open the file now.

![](https://i.imgur.com/3DSB6DR.png)

![](https://i.imgur.com/DhpRKPo.png)

Contains a whole windows7 vitrual machine, we have to double click the `.vmx` file this is the primary configuration file for the VM & let's boot it.

![](https://i.imgur.com/5NyYqNm.png)

## Q1 Solution

`Q1 : When you surf "www.naver.com", Web browser shows something wrong. Fix it and you can find a Key`

Hm, when we visit the site we can't see something wrong:

![](https://i.imgur.com/pWGgkzD.png)

Then i had the idea to check the hosts file, and i noticed a second hidden hosts file:

![](https://i.imgur.com/DxnYPM3.png)

This has the first answer in:

![](https://i.imgur.com/VMqqbeq.png)

Q1 Answer: `what_the_he11_1s_keey`

## Q2 Solution

`Q2 : Installed Keylogger's location & filename(All character is lower case)`

