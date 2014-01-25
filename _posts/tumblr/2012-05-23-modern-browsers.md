---
layout: post
title: Modern Browsers
date: '2012-05-23T18:57:07+02:00'
tags:
- Rants
- Modern Browsers
tumblr_url: http://blog.aventine.se/post/23614241360/modern-browsers
---
Today Google released a doodle, it was well executed, fun and I think that Robert Moog would have enjoyed it. But this isn't about the doodle, it is about a small piece of text that Google shows just beneath it.

> Upgrade to a modern browser and see what this doodle can really do.

This piece of text (and a link to the Google Chrome download page) shows up in all non-Chrome browsers, implying that they are not 'modern'.

I have been guilty of using the term as well, once during feature sniffing, mostly to mean 'not Internet Explorer older than X' and I am sorry about that, especially since Microsoft seems to be taking development of Internet Explorer 10 very seriously.

But we should investigate what 'modern' means to Google, someone hinted that you need to support the non-standard Web Audio API to be detected as 'modern' so I booted up WebKit Nightly (built with the Web Audio API enabled) and went to Google and got same message.

Guessing that it was a lot simpler, I switched back to Firefox and changed the UA string to something that looked like Chrome and suddenly the message is gone, switch back to the default UA string and it shows up again.

Just to make sure that I didn't do anything wrong, and because I was curious I investigated other browsers. Internet Explorer does get the message and the doodle, so does Opera. The iPhone and most silly UA strings do not get the doodle, and therefore does not get the message either. I don't have any Android devices, but I assume that you would either get both the message and the doodle, or neither depending on if your device supports Flash.

In the end, the conclusion is that a 'modern browser' according to Google is a browser which sends 'Chrome' as its UA string and supports Flash or the Web Audio API.

Can we instead on production sites standardize on something like "this site requires (experimental) features not yet present in your browser" (Thanks [@getify](https://twitter.com/#!/getify) for the idea) and a link to instructions on how they can update their browser, or if it is a browser specific feature, information about the feature and why it isn't yet supported in their browser of choice.

Note
--------------------------------------------------------------------------------

If you are trying to reproduce results, make sure that you're using google.com in English, the text about a 'modern browser' doesn't show up otherwise.
