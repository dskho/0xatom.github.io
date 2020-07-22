---
title: SuNiNaTaS - Web 1
description: My writeup on web 1 challenge.
categories:
 - suninatas
tags: suninatas
---

![](https://i1.daumcdn.net/thumb/C264x200/?fname=https://t1.daumcdn.net/cfile/tistory/99DE7733599504E81D)

Hi all, i discovered this korean jeopardy-style CTF. This site has good challenges, i suggest you to try it out!

You can start pwning there > [SuNiNaTaS](http://suninatas.com/){:target="_blank"}

## Source Code Analysis

Challenge provide us the source code that we have to "exploit":

![](https://i.imgur.com/QJ5Wf05.png)

```
<%
    str = Request("str")

    If not str = "" Then
        result = Replace(str,"a","aad")
        result = Replace(result,"i","in")
        result1 = Mid(result,2,2)
        result2 = Mid(result,4,6)
        result = result1 & result2
        Response.write result
        If result = "admin" Then
            pw = "????????"
        End if
    End if
%>
```

Took me some time to figure out the language of it because im not familiar with. It's ASP.NET (server-side web-application framework). I have no idea about ASP, i never coded in my life but if you know a programming language you can easily understand what is going with some google.

We have to input something that after the structures to be equal to admin:

```
If result = "admin" Then
            pw = "????????"
```

From these functions we can understand that for sure our input should contain `a` & `i`:

```
result = Replace(str,"a","aad")
result = Replace(result,"i","in")
```

From `ai` -> `aadin` we can simple add `m` so will be `ami` -> `aadmin`

Then we have these functions:

```
result1 = Mid(result,2,2)
result2 = Mid(result,4,6)
```

Mid function returns a specified number of characters from a string. The syntax is:

`Mid(string, start[, length])`

So we have `aadmin` so there first Mid() will be -> `ad` - The second one will be ->  `min` so we have `admin`, Let's input `ami` and grab the flag.

![](https://i.imgur.com/eil8Jtl.png)

`Authkey : k09rsogjorejv934u592oi`

After we enter the Authkey:

![](https://i.imgur.com/wtMjlJS.png)

Was really fun, all the `*.kr` stuff are amazing! ;)


