---
title: TryHackMe - CTF collection Vol 1
description: My writeup on CTF collection Vol 1 challenges.
categories:
 - tryhackme
tags: tryhackme ctf base64 stego cipher wireshark
---

![](https://i.imgur.com/mn1SW0T.png)

## Box Stats

| Box Info      | Details       |
| ------------- |:-------------:|
| Box Name :    | **CTF collection Vol 1**  |
| Difficulty :  | **Easy**             |
| Play :    | [CTF collection Vol 1](https://tryhackme.com/room/ctfcollectionvol1){:target="_blank"}      |
| Recommended : | Yes :heavy_check_mark:      |

## Summary

Hello, some really cool CTF tasks to pass your time. Let's start!

## What does the base said?

We simply have to do a base64 decoding.

```
$ echo "VEhNe2p1NTdfZDNjMGQzXzdoM19iNDUzfQ==" | base64 -d
THM{ju57_d3c0d3_7h3_b453}
```

## Meta meta

We've to download an image, as the title says we have the check the metadata. Metadata provides information about an image. We'll do that using the `exiftool`.

```
$ exiftool Findme.jpg | grep -i 'THM'
Owner Name                      : THM{3x1f_0r_3x17}
```

## Mon, are we going to be okay?

This challenge gives us an image again, we'll use `steghide` to extract a txt file from it.

```
$ steghide extract -sf Extinction.jpg
Enter passphrase:
wrote extracted data to "Final_message.txt".
$ cat Final_message.txt
It going to be over soon. Sleep my child.

THM{500n3r_0r_l473r_17_15_0ur_7urn}
```

## Erm......Magick

This one is pretty easy, we just have to highlight the text.

![](https://i.imgur.com/kUZF6GO.png)

## QRrrrr

We just have to read a QR image, we can do it from terminal. Im using this toolkit `apt-get install zbar-tools`

```
$ zbarimg QR.png
QR-Code:THM{qr_m4k3_l1f3_345y}
scanned 1 barcode symbols from 1 images in 0.06 seconds
```

## Reverse it or read it?

It gives us an ELF file:

```
$ file hello.hello
hello.hello: ELF 64-bit LSB pie executable
```

We just have to apply basic reverse engineering on it, using `strings`.

```
$ strings hello.hello | grep -i 'THM'
THM{345y_f1nd_345y_60}
```

## Another decoding stuff

This one is a simply base58 decode, im using my favorite tool [CyberChef](https://gchq.github.io/CyberChef/){:target="_blank"}

![](https://i.imgur.com/UUiWHJH.png)

## Left or right

It's just ROT13 but we have to play a bit with the amount.

![](https://i.imgur.com/3HBnneo.png)

## Make a comment

We'll use the page inspector from firefox to search for the flag.

![](https://i.imgur.com/G5vhzPu.png)

## Can you fix it?

We have a broken PNG file:

```
$ file spoil.png
spoil.png: data
```

We have to add the right magic number. A magic number is located at the beginning of a file & verify the content of a file. You can find the list [here](https://en.wikipedia.org/wiki/List_of_file_signatures){:target="_blank"}

We can see from the hexdump that using the wrong magic number:

```
$ xxd -p spoil.png | head -1
2333445f0d0a1a0a0000000d4948445200000320000003200806000000db
```

I like to use the `hexedit` hex editor, you can install it using this command `apt install hexedit`. Let's add the right one `89 50 4E 47 0D 0A 1A 0A`

```
$ xxd -p spoil.png | head -1
89504e470d0a1a0a0000000d4948445200000320000003200806000000db
```

We can now see the image is fixed & we can read the flag.

```
$ file spoil.png
spoil.png: PNG image data
```

![](https://i.imgur.com/7IQqFmD.png)

## Read it

For this one we have to search the web for the flag, hint says to us to check `reddit` so i used advanced operators to make it easy.

`site:reddit.com/r/tryhackme ctf collection`

I found the flag [here](https://www.reddit.com/r/tryhackme/comments/eizxaq/new_room_coming_soon/){:target="_blank"}

## Spin my head

That's brainfuck, we have to use a brainfuck deobfuscator to solve that. I always use this [one](https://www.splitbrain.org/_static/ook/){:target="_blank"}

![](https://i.imgur.com/bLevrG3.png)

## An exclusive!

We've to use the XOR (exclusive or) binary operator. I found an online [XOR calculator](http://xor.pw/){:target="_blank"}

![](https://i.imgur.com/oJKtrWe.png)

## Binary walk

As the title said we have to use `binwalk`.

```
$ binwalk -e hell.jpg

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.02
30            0x1E            TIFF image data, big-endian, offset of first image directory: 8
265845        0x40E75         Zip archive data, at least v2.0 to extract, uncompressed size: 69, name: hello_there.txt
266099        0x40F73         End of Zip archive, footer length: 22

$ cd _hell.jpg.extracted
$ ls   
40E75.zip  hello_there.txt
$ cat hello_there.txt
Thank you for extracting me, you are the best!

THM{y0u_w4lk_m3_0u7}
```

## Darkness

We get a dark image, we'll use [stegsolve](https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve/stegsolve){:target="_blank"} to apply color filters on image.

```
$ wget -q https://github.com/eugenekolo/sec-tools/raw/master/stego/stegsolve/stegsolve/stegsolve.jar
$ java -jar stegsolve.jar
```

![](https://i.imgur.com/iVpCXMw.png)

## A sounding QR

It gives us another QR image we read it and drives us to a soundcloud link:

```
$ zbarimg QRCTF.png
QR-Code:https://soundcloud.com/user-86667759/thm-ctf-vol1
```

It says the flag, i used an [audio to text converter](https://www.kukarella.com/audio-to-text-converter){:target="_blank"} on it.

![](https://i.imgur.com/GMKS4mH.png)

Since it says all CAPS, we can use linux magic to do it faster.

```
$ echo "s o u n d i n g q r" | tr '[:lower:]' '[:upper:]' | tr -d ' '
SOUNDINGQR
```

## Dig up the past

On this one we have to visit an old site, we can do that using the [wayback machine](https://web.archive.org/){:target="_blank"}

![](https://i.imgur.com/xo8pp8t.png)

![](https://i.imgur.com/CXcBzUU.png)

## Uncrackable!

We've to deal with a vigenere cipher but we don't know the key. I always use [this](https://www.guballa.de/vigenere-solver){:target="_blank"} tool to such cases.

![](https://i.imgur.com/IVP3bay.png)

## Small bases

Here we have to deal with radix as the hint says `dec -> hex -> ascii`

`581695969015253365094191591547859387620042736036246486373595515576333693`

goes to:

`54484D7B31375F6A7535375F346E5F307264316E3472795F62343533357D`

goes to:

`THM{17_ju57_4n_0rd1n4ry_b4535}`

I used these 2 tools: [dec -> hex](https://www.rapidtables.com/convert/number/hex-to-decimal.html){:target="_blank"} - [hex -> ascii](https://www.rapidtables.com/convert/number/hex-to-ascii.html){:target="_blank"}

## Read the packet 

On this challenges, it gives us a pcapng file. It's a packet capture format that contains a dump of data packets captured over a network.

```
$ file flag.pcapng
flag.pcapng: pcapng capture file - version 1.0
```

We'll open that file using `wireshark`. Wireshark is a packet sniffer tool, captures network traffic on the local network and stores that data for offline analysis.

First let's open the file.

![](https://i.imgur.com/k0gp622.png)

We will use now wireshark filters to search by protocol `HTTP`

![](https://i.imgur.com/27YVPfZ.png)

Now we will follow the HTTP stream to get the flag.

![](https://i.imgur.com/kpsxhyK.png)

![](https://i.imgur.com/kyNrI7p.png)

## Thank You

Thank you for taking the time to read my writeup. If you don't understand something from the writeup or want to ask me something feel free to contact me through discord(0xatom#8707) or send me a message through twitter [0xatom](https://twitter.com/0xatom){:target="_blank"}

Until next time keep pwning hard! :fire:
