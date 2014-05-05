---
layout: post
title: "Argument Prescan of C Preprocessor"
published: true
tags: [C]
categories: [Programming]
---

Years ago, I found a macro substitution issue when I was using stringify macro. See the exacmple code listed as below:

	#include <stdio.h>
	
	#define HAHA lala
	#define _(x) #x
	#define TEST _(HAHA)
	
	int main(int argc, char *argv[])
	{ 
		printf("%s\n", TEST);
		return 0;
	}

What I expected the output is `"lala"`. But I got `"HAHA"`. Why the `HAHA` was not substituted in the `TEST` macro ?

So that I wrote an another example:

	#define <stdio.h>
	
	#define HAHA lala
	#define _(x) #x
	#define STR(x) _(x)
	#define TEST STR(HAHA)
	
	int main(int argc, char *argv[])
	{
		printf("%s\n", TEST);
		return 0;
	}

Finally, the output is `"lala"`. I did not understand util I found this [document](http://gcc.gnu.org/onlinedocs/cpp/Argument-Prescan.html#Argument-Prescan)

> Argument Prescan
> 
> Macro arguments are completely macro-expanded before they are substituted
> into a macro body, unless they are stringified or pasted with other tokens.

That is, if we defined a macro with argument(s), the argument will be expanded before substitution if this argument is an macro itself. __One of the exceptions is the argument is used in `#` or `##`__.
