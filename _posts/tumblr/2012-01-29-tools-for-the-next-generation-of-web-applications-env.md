---
layout: post
title: Environment and Feature Detection
date: '2012-01-29T01:33:00+01:00'
tags:
- Tools for the next generation of Web Applications
- javascript
- programming
- WebCL
- River Trail
tumblr_url: http://blog.aventine.se/post/16665382175/tools-for-the-next-generation-of-web-applications-env
disqus_id: http://blog.aventine.se/post/16665382175
---
 > A warning, most of these extensions may have extreme security issues currently, they are prototypes after all. Use a separate browser instance and profile for this series.

If you already have a working install of WebCL and River Trail, you can skip this part. Do not assume that you have them just because you have the latest version of your browser, because they are not available yet in _any_ browser without a special build or extension.


Preparation
--------------------------------------------------------------------------------

The first step is to install OpenCL drivers for all your devices.

If you are running OS X (Snow Leopard or Lion) then all OpenCL drivers are already installed and you're good to go.

If you are running Linux or Windows, then you might need to install some OpenCL drivers. If your CPU is supported by both the AMD and Intel SDKs then I recommend you to install both.

 - [Intel OpenCL SDK](http://software.intel.com/en-us/articles/vcsource-tools-opencl-sdk/) (requires a CPU with support for SSE 4.1, see [here](http://software.intel.com/en-us/articles/opencl-release-notes/#2))
 - [AMD APP SDK](http://developer.amd.com/sdks/AMDAPPSDK/downloads/Pages/default.aspx) (requires a CPU with support for SSE 2, see [here](http://developer.amd.com/sdks/AMDAPPSDK/pages/DriverCompatibility.aspx))

If you have an nVidia or AMD graphics card, you probably already have OpenCL drivers installed for them (they are included in the graphics drivers), but you should make sure they are the latest version.

Now we're onto a few semi-optional tools that you probably want, but can avoid them if you want to,

 - [Git](http://git-scm.com/)
 - [Node.js](http://nodejs.org/)
 - [Coffeescript](http://coffeescript.org/)

Git is my version control system of choice, and you will probably want to check out the repositories of examples and on Github.

Coffeescript is a thin wrapper around Javascript that I happen to like, it has a bit more Pythonesque syntax and is really nice. A lot of the support libraries, and a few of the examples will be written in Coffeescript, and you might want to be able to recompile them.


Installing WebCL
--------------------------------------------------------------------------------

Now we can start installing the WebCL prototypes.

If you are running Linux or Windows, then you need to first install a 32-bit version of Firefox 10 and make sure that you have installed [Firebug](http://getfirebug.com/downloads) into your new profile, then you can install the [Nokia WebCL prototype](http://webcl.nokiaresearch.com/).

If you are running OS X, then you need to install the [Samsung WebCL prototype](http://code.google.com/p/webcl/) based on WebKit. It is a bit complicated since you need to compile it from scratch.

Just follow the included readme, after a while into the build, you might meet some compilation errors but they are easily fixable.


Installing River Trail
--------------------------------------------------------------------------------

To allow River Trail code to be accelerated via OpenCL you need to install the [River Trail extension](https://github.com/RiverTrail/RiverTrail/wiki).

On Windows or Linux it really does need the Intel OpenCL driver, the AMD OpenCL driver or the driver for your graphics card is not enough. But if your computer does not support the Intel OpenCL then you can still execute River Trail code using a normal Javascript engine.

On OS X, the built in OpenCL drivers are fine.


Detecting WebCL and River Trail
--------------------------------------------------------------------------------

So, after installing all that, we need a simple way to check that it is working. The easiest way is to checkout [https://github.com/JensNockert/tools-for-the-next-generation]() with git (or download an archive of the repository from Github).

Under "01 - Feature Detection" there are two html files, *webcl.html* and *rivertrail.html* that contain feature detection code. Try both to make sure that your setup works. If everything installed correctly, it will look something like this for River Trail,

![](http://media.tumblr.com/tumblr_lyj9vrtzoP1ql0q2q.png)

And something like this for WebCL,

![](http://media.tumblr.com/tumblr_lyj9vhrpUd1ql0q2q.png)

The code is not really that spectacular, but feel free to check out the source and see my horrible DOM manipulation code. (Hook me up with a pull request if you enjoy that kind of stuff)


About OpenCL
--------------------------------------------------------------------------------

Make sure you have *webcl.html* open in a browser, and make a small note of the structure of the information.

The first level in the output, "Apple" in my screenshot is the OpenCL platform name and underneath all OpenCL devices corresponding to that platform (but a single piece of hardware can be devices under multiple platforms.)

In OpenCL there are two domains where code can execute, the host (in WebCL this is the browser) or on a device which is connected to a host. The code we run on the host we call the 'Application' and on the code on devices we call 'Kernels'.

And as we will learn in future lessons, calling kernels is different from how we call normal functions from the host. Another important thing to note about kernels is that they are not written in Javascript but a high-performance variant of C.


Summary
--------------------------------------------------------------------------------

To summarize on what you should install,

 - Browser capable of WebCL
 - Browser capable of accelerated River Trail

and make sure they work. The rest is mainly sugar that could help you reach that goal.


Notes
-------------------------------------------------------------------------------

I will be using Firefox most of the time, but the example code that does not depend on a specific feature should be portable to most major platforms (Firefox, Chrome, Safari and Opera.)

Any specific feature dependencies will be noted in the corresponding article (and please point it out in the comments if it is not.)


Edits
-------------------------------------------------------------------------------

 1. Updated for Firefox 10