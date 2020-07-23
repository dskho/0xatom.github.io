---
title: SuNiNaTaS - Forensic 15
description: My writeup on forensic 15 challenge.
categories:
 - suninatas
tags: suninatas metadata exiftool
---

![](https://i1.daumcdn.net/thumb/C264x200/?fname=https://t1.daumcdn.net/cfile/tistory/99DE7733599504E81D)

Hi all, let's pwn the second forensic challenge.

You can start pwning there > [SuNiNaTaS](http://suninatas.com/){:target="_blank"}

## Solution

Challenge gives us an mp3 file to download:

![](https://i.imgur.com/vQj7ZPc.png)

Let's download the mp3:

```
$ wget -q http://suninatas.com/challenge/web15/diary.mp3
$ file diary.mp3 
diary.mp3: Audio file with ID3 version 2.3.0, contains:MPEG ADTS, layer III, v1, 128 kbps, 44.1 kHz, JntStereo
```

I started to doing stuff with sonic-visualiser but the solution is much simplier, the authkey is hidden in metadata.

```
$ exiftool diary.mp3 
ExifTool Version Number         : 11.98
File Name                       : diary.mp3
.. junk data ..
Conductor                       : GoodJobMetaTagSearch
```

Authkey: `GoodJobMetaTagSearch`

![](https://i.imgur.com/eZHDs5X.png)
