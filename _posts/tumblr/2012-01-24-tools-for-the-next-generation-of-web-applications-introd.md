---
layout: post
title: 'Tools for the next generation of Web Applications: Introduction'
date: '2012-01-24T18:29:26+01:00'
tags:
- Tools for the next generation of Web Applications
- programming
- javascript
tumblr_url: http://blog.aventine.se/post/16413173742/tools-for-the-next-generation-of-web-applications-introd
disqus_id: http://blog.aventine.se/post/16413173742
---
I do not know how the web will evolve in the future, I don't think that anybody knows how the web of 2020 will look like, or what applications will be popular then.

But regardless of the direction the web evolves, we will undoubtedly see more and more complex client-side applications being developed. And a lot of the applications that were traditionally native applications will probably migrate to the browser within this time frame.

The migrating applications might include everything from games to large simulations to image and video editing, and everything inbetween. Your imagination is hopefully the only limit to what you will be able to achieve.

Because betting on the web is one of the safest bets to take, it is simply the platform that is most accessible to people today, and the platform people care about the most.

The goal of this series of articles is to give you some insight into some techniques, frameworks and tools that might be useful to build this new generation of applications, or to allow you to improve your current applications.

The tools that I am most interested in are tools that enable a new class of applications that we earlier could not build in the browser without plugins, and we'll primarily focus on the set of these are almost purely performance increasing.

For example,

 - Faster Javascript engines
 - Typed Arrays
 - SIMD Intrinsics
 - Workers
 - River Trail
 - WebCL
 - WebGL (for computing, not graphics)

But to understand these new tools of the web, we need to understand the native libraries and features that power them.

The faster Javascript engines of the future, typed arrays and any SIMD intrinsics are designed to accelerate each thread of your applications. And do so by allowing us to utilize each processing core in a better way, and program 'closer to the metal'.

River Trail and WebCL utilizes [OpenCL](http://www.khronos.org/opencl/) to allow a piece of code to run on multiple processor cores, and in the case of WebCL, allow your code to run on graphics cards and even specialist OpenCL accelerators right from your browser.

If you haven't heard of OpenCL, it is a framework for heterogenous computing designed by Khronos (who are also maintaining the OpenGL standard), and allows you to execute kernels written in a high-performance variant of C on just about any processor around. OpenCL supports everything from large clusters down to small embedded systems.


Single-threaded
--------------------------------------------------------------------------------

Currently there are two engines that I enjoy to code for: the new Spidermonkey with type inference introduced in Firefox 9 which is really nice, and the V8 / Crankshaft engine used in Chrome. But the Javascript engines of the future has a lot more in store for us, and all of them are already picking up steam.

For example, Mozilla is currently working on [Ionmonkey](https://wiki.mozilla.org/IonMonkey). Ionmonkey is a new whole-method JIT for Spidermonkey that hopefully brings some significant speedup for many types of code (and especially the type of code that we are interested in). It isn't ready yet, but we can already see some benchmarks [here](http://arewefastyet.com/?a=b&view=breakdown) and follow how it develops.

Internet Explorer 9 introduced the new Chakra engine, which has some interesting features that will probably migrate to other engines. For example, it compiles code on a separate thread, allowing it to load code faster and start executing it quicker. And I am convinced that Internet Explorer 10 will introduce features that will allow Internet Explorer to defend its position as the most widely used browser.

One of these features that will be included in the next version of Internet Explorer (but is already supported in all other major browsers) is one of the most significant API developments in high-performance Javascript during the last few years: typed arrays. Typed arrays behave in most respects like regular Javascript arrays, but they have a fixed type and length. This on one hand gives Javascript programmers a nice way to interact with binary data and on the other hand gives the Javascript engines a lot more opportunities for optimization.

The Google Chrome team also introduced [NaCl (Native Client)](https://developers.google.com/native-client/) the last year, which is a reasonably interesting proposition from a performance standpoint, since it allows you to replace some of your Javascript with native code. It seems like you should be able to implement an OpenCL to NaCl compiler, which could be very interesting. Unfortunatly since it uses binaries instead of code, it is very hard to inspect the scripts, unlike in Javascript.

The two other browser vendors, Webkit (Apple and friends) and Opera recently shipped new browser engines, and support all the engine-level features that we currently expect, and are very likely to stay competitive in the future.

But there are a lot of other features that are in the planning stage. A pet feature of mine, for example, are SIMD intrinsics. SIMD (Single-Instruction Multiple-Data) is a method for improving computational throughput in modern processors by performing the same operation in parallel on multiple pieces of data.

These SIMD intrinsics are very simple functions that essentially map down to a few simple SIMD assembly instructions. The Javascript engine would be aware of how these functions work, and generate special optimizations for them. 

This is mainly an optimization, but it would also allow us to write more easily readable code when manipulating 'strange types' in Javascript, for example, long (64-bit) integers.

While there are currently not even any proposals of how these SIMD intrinsics should behave, there is still a high probability that we will see something along those lines in a future revision of the Javascript language.

This leads us to more complex parallelization features that introduce more than one thread of execution.


Multi-threaded
-------------------------------------------------------------------------------

[Workers](http://dev.w3.org/html5/workers/) are currently the only way of executing Javascript in parallel that is widely supported in current browsers, but they are not really designed for the task. They are designed to allow for background tasks, but are not really suitable for computation on their own.

But make sure that you do not forget about them, because they are a good fallback and can be a force multiplier when combined with more advanced features.

[River Trail](https://github.com/RiverTrail/RiverTrail) utilizes OpenCL to execute Javascript kernels on a multi-core CPU using a friendly API. I am quite convinced that it will be a popular choice in the future.

The most compelling feature of River Trail is that it is tightly integrated with the browser, and therefore allows for a lot more optimization than WebCL (or OpenCL) allows. Don't be surprised if a future River Trail implementation outpaces WebCL significantly on short kernels where OpenCL imposes a too high communication overhead.

Another interesting thing is that River Trail can be combined with a lot of other performance increasing features in Javascript, for example SIMD intrinsics, which (like the SIMD features in OpenCL) could significantly increase performance and readability for certain kernels.

[WebCL](http://www.khronos.org/webcl/) is essentially the big brother of River Trail, exposing the full OpenCL API to the web programmer, and allows you to use unmodified OpenCL kernels in your application. It is essentially the more flexible version of River Trail, and is designed to allow you to use any OpenCL accelerator in the system, including graphics processors and so on.

WebCL is also the API that we will be using the most throughout this series, mainly since the compilers are mature, and it is also the language and framework that I am most familiar with.

But River Trail has some interesting opportunities that we won't see in WebCL since could be tighter integrated into the browser at a future point. For example could an implementation of River Trail significantly reduce the communication overhead required to run kernels on the CPU, which is currently quite significant in OpenCL.

WebCL currently has the advantage that the infrastructure is a bit more mature on the kernel side, on the Javascript side both technologies are noticably not ready.

[WebGL](http://www.khronos.org/webgl/) on the other hand has the advantage that it is reasonably mature, and allows execution on just about every graphics card available.

But I generally wouldn't recommend using it for computation though unless you have very specific requirements, since even the simplest tasks can easily turn extremely complex unless you're very good at GLSL and WebGL. It is simply not designed for computation, only graphics.


Conclusion
--------------------------------------------------------------------------------

There are many tools and frameworks already available in a pre-release form for us to play with, and the best way to get used to them is to actually use them. Just be be aware of the changing and non-final nature and use this time to your advantage, most developers won't start using these tools until they are almost ready, and by then it is to late to influence their growth.

In addition to tools that 'merely' grant us faster performance, we have a lot of tools that simply allow us to do a lot of things that we could not do before, but those are interesting enough to get their own introductions when we meet them later in the series.

The next episode will be a shorter one, and contain instructions on how to set up our development environment on Windows or Linux. For example, installing different OpenCL drivers, WebCL plugins and River Trail.


Notes
--------------------------------------------------------------------------------

There are currently at least three different implementations of WebCL,

 - [Nokia Prototype](http://webcl.nokiaresearch.com/)
 - [Samsung prototype](http://code.google.com/p/webcl/)
 - [Mozilla prototype](http://hg.mozilla.org/projects/webcl).
