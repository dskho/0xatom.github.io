---
title: SuNiNaTaS - Web 2
description: My writeup on web 2 challenge.
categories:
 - suninatas
tags: suninatas javascript burp
---

![](https://i1.daumcdn.net/thumb/C264x200/?fname=https://t1.daumcdn.net/cfile/tistory/99DE7733599504E81D)

Hi all, let's pwn the second web challenge. This challenge requires basic javascript knowledge.

You can start pwning there > [SuNiNaTaS](http://suninatas.com/){:target="_blank"}

## Source Code Analysis

We have a simple login function:

![](https://i.imgur.com/vX6uH5j.png)

Let's check the source code.

```js
<script>
	function chk_form(){
		var id = document.web02.id.value ;
		var pw = document.web02.pw.value ;
		if ( id == pw )
		{
			alert("You can't join! Try again");
			document.web02.id.focus();
			document.web02.id.value = "";
			document.web02.pw.value = "";
		}
		else
		{
			document.web02.submit();
		}
	}
</script>
```

If our id & pw input are equal will call alert() with an error message & will also call the focus() method. If they're not equal will call the submit() method.

We have to bypass the javascript here, we will input something random like `test:lol` and we will intercept the request with burp and as the hide says: `id = pw` we will send 2 equal data.

## Exploitation

![](https://i.imgur.com/DVkXm4z.png)

![](https://i.imgur.com/OfFb0wB.png)

Now we can add 2 equal data:

![](https://i.imgur.com/0klVpiM.png)

We press forward and we get the authkey!

`Authkey : Bypass javascript`

![](https://i.imgur.com/l7FnYl3.png)

Tricky & fun! ;)


