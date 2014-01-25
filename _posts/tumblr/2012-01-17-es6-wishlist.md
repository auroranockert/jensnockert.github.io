---
layout: post
title: A wishlist for ES6
date: '2012-01-17T20:03:00+01:00'
tags:
- programming
- javascript
- harmony
tumblr_url: http://blog.aventine.se/post/16015782471/es6-wishlist
---
First, I am not a web designer, and I realize that my needs are not the same as everyone elses. But I am going to argue that there are only a few features that _need_ to be added to Javascript in the next version (ES6 or Harmony).

 1. Typed Arrays
 2. An improved numerical library
 3. Continuations

And you will probably have your own list, but if you don't agree with at least the first two, then you are probably wrong. They are really important, both in the browser and outside. The third is more of a personal preference, but I think it is probably the one change that would improve the language the most.

 
Typed Arrays
--------------------------------------------------------------------------------
 
[Khronos' typed arrays](www.khronos.org/registry/typedarray/specs/latest/) are essentially just an optimization, they work just like arrays, but you can only store a specific type, and their size is specified when the object is created.

This gives a bit of extra performance for many kinds of applications. And in the future we will probably write a lot more applications that manipulate binary data in Javascript. In addition, the interface is pretty good and provides sugar for different sorts of type-conversion. It does not introduce any new syntax and most browsers support them to a certain degree already, so backwards compatibility isn't a problem.

The only thing missing in the current specification is a way to determine the native endianness of the machine the it is executing on, and you therefore need a small method to do it for you.

Float64Arrays are not implemented in Safari, DataViews are not implemented in Safari, Firefox or Opera. Typed Arrays are not supported at all in IE before version 10.

Numerical Library
--------------------------------------------------------------------------------

The numerical library in Javascript (the Math module) is not only exceptionally sparse, it isn't defined what it should do either. `Math.sin(x)` could return 4 and still follow the specification.

This makes coding to the specification impossible, and useless, since only the name of the function is defined, and a few edge cases. Four is a valid approximation of the value of sine, `sin(x) = cos(x) - 1` is another valid approximation of sine that is legal but not very good. Or `sin(x) = 1 - e^x` which quickly becomes an extremely bad approximation.

The [proposal](http://wiki.ecmascript.org/doku.php?id=harmony:more_math_functions) for ES6 isn't any better, since it does not include specifications either. The correct solution to the problem is probably that all operations should be correctly rounded (as in the IEEE 754-2008 specification) but [OpenCL](http://www.khronos.org/registry/cl/specs/opencl-1.2.pdf) (see Chapter 7) provides relative error bounds, and that would be acceptable solution to the specification problem as well.

There are also some significant functions in IEEE 754-2008 that are not included in the current ES6 proposal, the most significant is the Fused Multiply-Add which is quite slow to emulate in software.

OpenCL also provides a lot of useful functions that may be interesting to implement, especially for games and other applications that require geometry and colour operations. And if the new correctly rounded functions are too slow for embedded devices, or for specific applications, then adding native, fast, half (or single) versions of some of the functions could make sense. (see the OpenCL specification again)

This change would not really affect any applications, if anything, it would make the output of a Javascript application using the Math module a lot more predictable. It also does not introduce any new syntax, and should be easy to add.

Continuations
--------------------------------------------------------------------------------

Adding continuations is more of a personal thing, I find them extremely useful in other languages, and I would find them extremely useful to handle the whole callback-inception that you usually get stuck in while coding Javascript.

It is a more controversial extension to the language since it actually involves syntax, but on the other hand, a reasonable Python-style implementation is already available in Spider Monkey (under the name [generators](https://developer.mozilla.org/en/JavaScript/Guide/Iterators_and_Generators)), so any syntax changes is arguably already there.

Summary
--------------------------------------------------------------------------------

I don't really think that we need a lot of the syntax that people are proposing and while some are reasonable (blocks for example), I do not really feel that we really need them.

The same thing for classes and so on, we should be relatively restrictive about adding syntax, since it restricts our ability to extend syntax in the future. On the other hand, we can be relatively liberal when adding library functions that can be deprecated in later versions of the standard.
