---
title: TryHackMe - Easy steganography Writeup
description: My writeup on Easy steganography challenge.
categories:
 - tryhackme
tags: tryhackme
---

Hi all, let's pwn this.

You can find the challenge there > [Easy Steganography](https://tryhackme.com/room/easysteganography){:target="_blank"}

# Flag 1

Simply we run `hexdump` on the image and after lot of search we can see flag.

```bash
$ hexdump -C flag1.jpeg | grep S
|..pg<.St3g4n0...|
```

Flag 1 > St3g4n0

# Flag 2

We'll use now `binwalk` you can install it by `sudo apt-get install binwalk -y`.

```bash
$ binwalk --dd='.*' flag2.jpeg
$ cd _flag2.jpeg.extracted/
$ ls 
0  1326F  2C51  C
$ file 1326F 
1326F: JPEG image data ... <other_data>
$ display 1326F
```

Flag 2 > algorithm

# Flag 3

This is one is pretty easy, we just run `file` on the image & we can see the flag.

```bash
$ file flag3.jpeg 
flag3.jpeg: JPEG image data, Exif standard: [TIFF image data, little-endian, direntries=0], comment: "The passphrase to this challenge is Math", progressive, precision 8, 526x263, components 3
```

Flag 3 > Math

# Flag 4

This is one is pretty easy too. We just run `strings` on it.

```bash
$ strings flag4.jpeg
TryHardered
```
Flag 4 > TryHardered

Pretty easy challenges, can teach the basics to people that have no idea on steganography.

See you!
