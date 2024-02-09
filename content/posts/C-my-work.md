---
title: "What I wish I knew when I learnt C"
date: 2024-01-07
draft: false
tags:
  - C
  - C++
  - Programming
  - Optimization
  - Teaching
  - Tips
categories:
  - Programming
---

One day, I decided that I wanted to know how to hack things. I was 15 years old and I did what everyone did: I googled "how to hack a webcam". I found a video from Nawop (you wont find much about him but he was doing tutorials about cybersecurity). It was a metasploit tutorial, showing how to build a reverse shell tcp for a windows target.

I struggled trying to reproduce it, I spent a whole month trying to understand the problems I faced. In the end, I managed to achieve my goal, it wasn't like in the movies but I was happy with it.

I talked about it on Nawop's Discord. I met some people there, they are now [offensivewave](https://offensivewave.com/), two of them told me "you should learn C!" so I did.

So, through this article, I will try to share things that I wish I knew sooner, things that are not always taught in C/C++ courses, things that are not that much known and so on.

I also address a special thanks to everyone who taught me C/C++ back then, you will recognize yourself ❤️.


## What is C?
I won't tell you what C is, you certainly already know it. But if it's not the case, [here](https://en.wikipedia.org/wiki/C_(programming_language)) is a start.

But here is some things that you may not know about C:
- C is named C because it's the successor of [B](https://en.wikipedia.org/wiki/B_(programming_language)), which was the successor of [A (APL)](https://en.wikipedia.org/wiki/APL_(programming_language)).
- C is actually updated ! There is C99 from 1999, C11 from 2011 and C17 from 2018 (don't ask me why), and C23,C2x are planned for 2024.
- In C, an identifier can be up to 31 characters long. It can contain letters, digits and underscores. The first character must be a letter or an underscore.
- C has been invented to build operating systems, specifically Unix.
- C is a compiled language, it's a low-level language but it's also a high-level language. It's a low-level language because it's close to the machine, it's a high-level language because it's portable and it's easy to write.

## Environment
### Compiling
I will tell you something you may not like: you should read (at least part of it) the man page of gcc. Because there is a lot and it will be helpful sometimes.

But here is some parameters you should know:
- `-Wall` enables all warnings.
- `-Wextra` enables some extra warnings.
- `-Werror` makes all warnings into errors.
- `-Wpedantic` issue all the warnings demanded by strict ISO C and ISO C++.
- `-Wshadow` warns whenever a local variable or type declaration shadows another variable, parameter, type, or class member (in C++), or whenever a built-in function is shadowed.
- `-D` defines a macro. For example `-DDEBUG` will define the macro `DEBUG`. and you can use it like this: `#ifdef DEBUG ... #endif`. I will talk about it later.
- `-O` enables optimizations. `-O0` is no optimization, `-O1` is some optimization, `-O2` is more optimization, `-O3` is even more optimization. You can also use `-Os` to optimize for size. You can also use `-Ofast` to optimize for speed, but it's not always a good idea. For debug purpose you should use `-O0`.
- `-g` adds debugging information to the executable file, so you can use a debugger like gdb.

### Debugging
Debugging with printf is ok to start, it's easy and it works. But when you start to work on bigger projects, it's not enough anymore. You need to use a debugger.
> Note: Using printf to print non-format strings is not a good idea, you should use `puts` instead. `printf` is only if you need to format your string.

So let me show you [GDB](https://en.wikipedia.org/wiki/GNU_Debugger).
I will not explain how to use it, there is a lot of tutorials about it. But I will tell you you can use it in your IDE, it's not only a command line tool. For example, in VSCode, you can use the [C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) and the [CodeLLDB extension](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb) to debug your code.

### Toolchain
Many languages have a tool to compile, debug, test, etc. I didn't use them of a long time and I never really liked them. But after while doing a cross-compilation, I realized that it's not that hard to create a toolchain and it's so much more convenient. So I created [my own Makefile](https://github.com/i5-650/useful-conf/blob/main/Makefile) to do it!
It's easy enough to understand but will be sufficient for most of your projects.

## C tricks
### Macros
Macros are a powerful tool in C. Even if you only now them for `#include` and `#define`, they can do much more!

You might have seen this:
```c
#ifndef MY_HEADER_H
#define MY_HEADER_H
// code
#endif // MY_HEADER_H
```

This means that if `MY_HEADER_H` is not defined, it will define it and execute the code. If it's already defined, it will not execute the code. It's useful to avoid multiple inclusions of the same header. But you could use that for other things:
```c
#define DEBUG 1

int main(void){
    #if DEBUG
        printf("Debug mode\n");
    #endif // DEBUG

    return 0;
}
```
This allow you to have a debug mode, you can also use `-DDEBUG` when compiling to enable it. (remember the `-D` option?)

There is also "function-like macros":
```c
#define MAX(a,b) ((a) > (b) ? (a) : (b))
```
This will replace `MAX(a,b)` by `((a) > (b) ? (a) : (b))`. It's useful to avoid repeating yourself.

But you can do much more like:
```c
#define DIV_ROUNDUP(A, B) \
({ \
	typeof(A) _a_ = A; \
	typeof(B) _b_ = B; \
	(_a_ + (_b_ - 1)) / _b_; \
})
```
This is a macro that will round up the division of A by B. It's useful when you want to divide something in multiple parts. For example, if you want to divide an array of 10 elements in 3 parts, you can do `DIV_ROUNDUP(10, 3)` and it will return 4. But for you it's like a function, you can use it like this: `int a = DIV_ROUNDUP(10, 3);`.

There is also the `__attribute__` macro, it's useful to tell the compiler some things about your code. For example, you can tell the compiler that a function is deprecated:
```c
void foo(void) __attribute__((deprecated));
```
You can also tell the compiler that a function is a constructor or a destructor:
```c
void foo(void) __attribute__((constructor));
void bar(void) __attribute__((destructor));
```
And for debug purpose you have `__func__` and `__LINE__`:
```c
printf("Error in %s at line %d\n", __func__, __LINE__);
```

You can also define your code to be optimized with preprocessing instructions:
```c
#pragma GCC optimize("O3")
```
