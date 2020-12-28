---
title: TryHackMe - Musical Stego
description: My writeup on Musical Stego box.
categories:
 - tryhackme
tags: tryhackme stego steghide morse sonicvisualizer
---

![](https://i.imgur.com/yl33L7z.png)

## Box Stats

| Box Info      | Details       |
| ------------- |:-------------:|
| Box Name :    | **Musical Stego**  |
| Difficulty :  | **Easy**             |   
| Play :    | [Musical Stego](https://tryhackme.com/room/musicalstego){:target="_blank"}      |
| Recommended : | Yes :heavy_check_mark:      |

## Summary

Hello, this one was an easy challenge based on audio steganography. Steganography is hiding a file/message inside of another file. Let's start!

## Who remixed the song?

The answer is in front of our eyes. `Kilmanjaro`

```
$ ls -lah
total 46M
drwxr-xr-x 2 root root 4.0K Dec 28 15:12  .
drwxr-xr-x 7 root root 4.0K Dec 27 21:40  ..
-rw-r--r-- 1 root root  46M Dec 28 15:09 'Language Arts - DEF CON 27- The Official Soundtrack - 02 Luckiness (Kilmanjaro Remix).wav'

```

## What link is hiding in the music?

On this one, we'll use a tool called `Sonic Visualizer`. Is a tool for analysing the contents of audio files. You can install it by using this command `apt install sonic-visualiser`.

First open the file.

![](https://i.imgur.com/5IX8vp5.png)

Then add spectrogram.

![](https://i.imgur.com/9HQik8t.png)

There is a QR code, play a bit with the filters so you can scan it.

![](https://i.imgur.com/T7zROji.png)

The scan gives the answer `https://voca.ro/imPgJC013AW`.

## What does the found audio convert to? [CHECK HINT, LINK IS DEAD]

When we visit the link we see another audio file, it sounds like `beep boop` that's morse audio code. Let's use a [morse code audio decoder](https://morsecode.world/international/decoder/audio-decoder-adaptive.html){:target="_blank"}

![](https://i.imgur.com/9g43FEo.png)

We have the answer `pastebin.com/LZKTB4ET`

## What was the found password?

Because link is dead, hint says to use a github link `https://github.com/m00-git/XXXXXXXX` just we have to replace `XXXXXX..` with the `LZKTB4ET` from pastebin.

We have the password `S3CR3T_P455`

## What is the final flag?

Now we just have to use `steghide` on the first audio file with the password we found before `S3CR3T_P455`

```
$ steghide extract -sf Language\ Arts\ -\ DEF\ CON\ 27-\ The\ Official\ Soundtrack\ -\ 02\ Luckiness\ \(Kilmanjaro\ Remix\).wav -p S3CR3T_P455
wrote extracted data to "secret".
$ cat secret
THM{f0und_m3!}
```

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"}

Until next time keep pwning hard! :fire:
