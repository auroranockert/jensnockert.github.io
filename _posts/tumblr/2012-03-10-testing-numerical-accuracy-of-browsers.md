---
layout: post
title: Testing numerical accuracy of browsers
date: '2012-03-10T20:27:00+01:00'
tags:
- Floating Point
- Javascript
tumblr_url: http://blog.aventine.se/post/19071382040/testing-numerical-accuracy-of-browsers
---
According to the standard, only the arithmetic operations in Javascript need to be correctly rounded, the functions in `Math` does not have any accuracy requirements.

But out in the real world, browsers are a bit better than that, we have a feeling that the functions in `Math` are reasonably accurate, but if you need to be convinced (like me) then you should look at [https://github.com/JensNockert/accuracy.js](https://github.com/JensNockert/accuracy.js) which fuzz tests most of the operations in `Math` that have a tendency to be inaccurate.

If you want to be even more convinced, generate more test cases using generate.rb.

Ps. `sin`, `cos` and `tan` are missing, their periodicity makes them hard to fuzz using this technique.

Update
--------------------------------------------------------------------------------

 1. I fuzzed on Windows as well, and Chrome on Windows does not provide `sqrt` with correct rounding, a bug has been filed. Firefox and Opera provide as much precision on Windows as on OS X.