---
title: SuNiNaTaS - Web 4
description: My writeup on web 4 challenge.
categories:
 - suninatas
tags: suninatas burp user-agent
---

![](https://i1.daumcdn.net/thumb/C264x200/?fname=https://t1.daumcdn.net/cfile/tistory/99DE7733599504E81D)

Hi all, let's pwn the 4 web challenge. This challenge was an easy one.

You can start pwning there > [SuNiNaTaS](http://suninatas.com/){:target="_blank"}

## Solution

When we visit the challenge page we can see this:

![](https://i.imgur.com/BZHW2Gt.png)

Source code has a comment, we have to make our points to 50 & SuNiNaTaS.

```
<!-- Hint : Make your point to 50 & 'SuNiNaTaS' -->
<!-- M@de by 2theT0P -->
```

Hm i start to press the plus button until 50 and at 25 points i got this alert message:

![](https://i.imgur.com/HhDZFEs.png)

![](https://i.imgur.com/BeFEvDK.png)

This is where `User-Agent` (identify the browser and operating system of the client) comes.. we have to change our user-agent to SuNiNaTaS to continue, let's fire up burp and capture the request and send to repeater.

![](https://i.imgur.com/KstXYCW.png)

Change the user-agent and press the send button 24 more times:

![](https://i.imgur.com/csHZvvJ.png)

Now at 49 we capture the request and change the user-agent and we pwned it!

![](https://i.imgur.com/y18kRCf.png)

![](https://i.imgur.com/ZorCHZd.png)

We get the authkey: `Change your Us3r Ag3ent`

![](https://i.imgur.com/t13FiND.png)

Interesting one!

