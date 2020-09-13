---
title: Vulnhub - The Planets Mercury
description: My writeup on The Planets Mercury box.
categories:
 - vulnhub
tags: vulnhub sqlmap sqli sudo
---

![](https://i.pinimg.com/originals/f0/95/97/f095972de1a5052f7568d673edd66a46.jpg)

You can find the machine there > [The Planets Mercury](https://www.vulnhub.com/entry/the-planets-mercury,544/){:target="_blank"}

## Summary

This machine is quite easy, i have to admit that i stuck on privilege escalation for a while. Starting off with a sqlmap scan on the page we can grab some creds and brute force ssh with them, this will give us a low-privilege shell on the box. Second privesc is really easy we just have to decode some base64 data & the final privesc to root is tricky we have to think smart to exploit it.
Let's start! :sunglasses:

## Enumeration/Reconnaissance

Let's start always with nmap.

```
