---
title: SuNiNaTaS - Web 3
description: My writeup on web 3 challenge.
categories:
 - suninatas
tags: suninatas web url
---

![](https://i1.daumcdn.net/thumb/C264x200/?fname=https://t1.daumcdn.net/cfile/tistory/99DE7733599504E81D)

Hi all, let's pwn the third web challenge. This challenge requires from us to understand the URL anatomy, it's a really basic one.

You can start pwning there > [SuNiNaTaS](http://suninatas.com/){:target="_blank"}

## Solution

Once we visit the challenge page, says to write an article on notice board:

![](https://i.imgur.com/Enr6SQd.png)

If we go to notice board there is no write button:

![](https://i.imgur.com/zdAdH7F.png)

Now if we go to free board we can see a write button:

![](https://i.imgur.com/73Gt2uI.png)

If we presss on it we can see the URL goes like: `suninatas.com/board/free/write` we can simple change it to `suninatas.com/board/notice/write` & write a random article:

![](https://i.imgur.com/ORZ9lVM.png)

When we press submit we get the authkey!

`Authkey : 1q2w3e4r5t6y7u8i9o0p`

![](https://i.imgur.com/eSPzy2G.png)

Funny.




