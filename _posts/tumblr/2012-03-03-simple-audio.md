---
layout: post
title: Simple Audio
date: '2012-03-03T00:18:34+01:00'
tags:
- Audio
- Web Audio
- Media Stream
tumblr_url: http://blog.aventine.se/post/18627646284/simple-audio
---
We have been discussing a lot of audio at Official.fm Labs, and since we're working with audio in different ways and have different views on what should be a first step; I am throwing out a proposal for them, and for you.

In addition to this one, there are at least two more proposals (which are a lot less sketchy and have partial implementations) for real-time audio on the web, [https://dvcs.w3.org/hg/audio/raw-file/tip/streams/StreamProcessing.html](https://dvcs.w3.org/hg/audio/raw-file/tip/streams/StreamProcessing.html) from Mozilla's Robert O'Callahan and [https://dvcs.w3.org/hg/audio/raw-file/tip/webaudio/specification.html](https://dvcs.w3.org/hg/audio/raw-file/tip/webaudio/specification.html) from Google's Chris Rogers.

Both proposals are designs based on graphs, containing various nodes. But their proposals are many pages long and I don't have the time or energy for that, so I will try to show you that audio in the browser can be described on a napkin (using both sides).

My idea is that audio is not really that complicated, you do not need a significant amount of routing or specialist code in the browser because you can do all of that in Javascript to once you have a few of the basic building blocks, and then when there are performance issues or other limitations, you add the more advanced features.

A first implementation should provide a seed, not a forest. To do that, we need a way to read and/or write audio samples to a stream and receive events relating to that stream.

This is essentially what the Mozilla Audio Data API does, but with a more expressive API.

 > Note that I wrote this in a few hours, so nothing is fixed, especially not names. Also, as an OS X zealot, I have been inspired quite a bit by Core Audio and any similarities are probably not coincidence.


Usage Scenarios
--------------------------------------------------------------------------------

 - Playing short sounds with low latency and accurate timing: Useful for games and similar applications where sounds react to user interaction.
 - Playing longer audio segments: Useful for example music players, or other streaming uses.
 - Capture audio from a microphone: Useful for all sorts of audio conferencing needs, for example something like Teamspeak or Skype in a browser.
 - Bypassing the lack of codec support in the HTML5 media elements: Useful for anyone with files in any format that is not supported in all browsers, like MP3, AAC, Vorbis, ALAC, FLAC, etc.

These are the usage scenarios that I am considering at first, and I think it covers 90% of the applications need audio (right now). Because if we look at what traditional applications that include audio are most popular, we notice that teleconference, games and music almost certainly will come up on top for almost every user.

And while I think the last 10% are absolutely awesome as well, (an HTML5 digital audio workstation for example) I think that they can wait a bit until we have the basics before we go on to the really mind-blowing stuff.


Features
--------------------------------------------------------------------------------

 1. Writing audio to devices.
 2. Reading audio from devices (adding support for *audio* or *video* elements should be trivial).
 3. Accurate timing, relatively low latency.
 4. Events from the audio subsystem.
 5. Easy to implement.
 6. Designed for future extensibility instead of providing a kitchen sink now.

Accurate timing and low latency is very important for certain kinds of games, if you need to wait 100ms from Mario hitting the coin until the sound starts playing, players will be confused and the experience will be bad. For multimedia, the audio needs to be in sync with whatever other things are happening.

Events are required, all applications should be able to act correctly on hot-plugging events and so on. For example, if a user uses an application where you can call landlines, without a microphone, then plugging in a microphone should directly enable audio input without a reload.

The same thing should be supported if for example a USB headset is connected while on a music site, the site should be able to react to this, and play through the headset instead.

It should also be relatively easy to implement, and in the future extend for more advanced functionality.


API Overview
--------------------------------------------------------------------------------

 1. An AudioContext, referring to the whole state of the audio subsystem of the browser.
 2. An AudioStream interface, representing a single audio stream, which can support both input and output.
 3. An AudioStreamDescription interface, a description of the data flowing in the stream.
 4. An AudioBuffer interface, contains data for a set of channels.
 5. An AudioTimeStamp interface, contains data detailing a specific point in time, relative to the clock driving a specific stream.


The Audio Context
--------------------------------------------------------------------------------

The audio context is essentially singleton, you can create multiple contexts, but they just masquerade for the global state, the streams available should be the same in all contexts. 

{% highlight javascript %}
interface AudioContext {
  readonly attribute AudioStream[] streams;
  readonly attribute AudioStream defaultInputStream;
  readonly attribute AudioStream defaultOutputStream;
}
{% endhighlight %}

Create a context via

 `var audio = new AudioContext()`

### Attributes

 - `streams`: An array of AudioStreams that are available for output or input.
 - `defaultInputStream`: An AudioStream representing the default input device, or `null` if there are no input devices.
 - `defaultOutputStream`: An AudioStream representing the default output device, or `null` if there are no output defices.


### Events

 - `NewStreamAvailable`: Contains the new stream.
 - `DefaultInputStreamChanged`: Contains the new default input stream, and the old default input stream. It is triggered when for example, the user plugs in a microphone.
 - `DefaultOutputStreamChanged`: Contains the new default output stream, and the old default output stream. It is triggered when for example, the user plugs in a pair of headphones.
 
 
### Discussion

To begin with, the only exposed streams would be the `defaultInputStream` and the `defaultOutputStream`, but more advanced applications like for example, a web-based digital audio workstation, could require additional streams to support a large amount of channels, or to provide for example DJ with two different outputs.

The AudioContext should be accessible from a worker, allowing audio processing to be done in a separate context from the rest of the application to provide latency sensitive applications with a more stable environment, less affected by garbage collection pauses.

There are a lot more things that would be interesting to send events about from an audio perspective, but which possibly should not be in the audio context, the first thing that pops to mind is an event when a device returns from deep sleep, to allow applications to prevent the accidental output when resuming for example a laptop.

In addition, there should be a method that allows you to create streams from media elements, allowing the programmer to post-process the audio in for example a video.


Audio Stream Description
--------------------------------------------------------------------------------

An audio stream description is a description of the current, or in the future, the desired state of an audio stream and are designed to hold a lot of information that is useless in the normal case of uncompressed linear PCM.

{% highlight javascript %}
interface AudioStream {
  readonly attribute DOMString identifier;
  readonly attribute double sampleRate;
  readonly attribute DOMString[] channels;
  readonly attribute short bitsPerChannel;
  
  /* Only for formats with a fixed frame-size */
  readonly attribute long bytesPerFrame;
  
  /* PCM specific attributes */
  readonly attributes DOMString sampleType;
  readonly attributes DOMString endian;
  readonly attributes DOMString aligned;
  readonly attributes boollean interleaved
}
{% endhighlight %}

### Attributes

 - `identifier`: Always 'Linear PCM' for now
 - `sampleRate`: The sampling rate in samples per second
 - `channels`: The canonical name of each channel
 - `bitsPerChannel`: The number of useful bits in each sample
 - `bytesPerFrame`: The number of bytes in each frame, including padding


### PCM specific attributes

 - `sampleType`: 'float', 'signed-integer', 'unsigned-integer'
 - `endian`: 'big', 'little'
 - `aligned`: 'packed', 'high', 'low'
 - `interleaved`: boolean

### Notes

Different codecs need different attributes, and if an attribute does not make sense for a specific stream, then it should not include it in the description.

To begin with, there is only need for Linear PCM, since that is the format of almost all modern hardware. But in the future, more complex format descriptions would be required, especially to describe more complex formats that could be extracted from for example a media element.

Another thing that might need a change is the channel descriptions, some sort of location or something could be useful, so maybe they should be changed from strings to objects.

In some future, with bitstreaming of audio, or codecs exposed to Javascript, more complex features might be required from the stream description.


Audio Time Stamp
--------------------------------------------------------------------------------

A time stamp object is simply represents a point in time, relative to the clock for a specific direction in a stream.

{% highlight javascript %}
interface AudioTimeStamp {
  readonly attribute AudioStream stream;
  readonly attribute DOMString direction;
  
  readonly attribute Date hostTime;
  readonly attribute double sampleTime;
}
{% endhighlight %}

### Attributes

 - `stream`: the audio stream that this time stamp is relative to
 - `direction`: 'output' or 'input', the direction of the stream that the timestamp is relative to
 - `hostTime`: a date, the time when the first sample will be played, or when the first sample was captured
 - `sampleTime`: Number of samples passed since the stream started (as a double, since a Javascript integer would only hold precision for 3 hours at 192kHz, a double on the other hand is fine for 1500 years).

### Notes

Some additional time measurment systems could be included, like the relative time in seconds, etc. But it feels unnecessary or first implementation.


Audio Buffer
--------------------------------------------------------------------------------

The simplest object, you do not create these yourself. They are passed to you when you need them.

{% highlight javascript %}
interface AudioBuffer {
  readonly attribute ArrayBuffer data;
  readonly attribute AudioTimeStamp timeStamp;
  readonly attribute long channels;
}
{% endhighlight %}

### Attributes

 - `data`: An ArrayBuffer containing data, or into which you need to write data.
 - `timeStamp`: The time at which the buffer 'starts', or `null`.
 - `channels`: The number of channels interleaved in this buffer.

### Notes

This is the construction I am least sure about, currently the channels could be inferred from the stream description, and the entire object could be replaced by a simple ArrayBuffer. I am not sure if there is any requirements for extensibility either, Google has some extra properties for these that while useful, could also be inferred from the stream description.


Audio Streams
--------------------------------------------------------------------------------

An audio stream represents a stream of audio data to/from a device or media element. The simplest way to get a stream is to get the default ones,

{% highlight javascript %}
interface AudioStreams {
  readonly attribute AudioStreamDescription input;
  readonly attribute AudioStreamDescription output;
}
{% endhighlight %}

### Attributes

There are two interesting attributes on the stream,

 - `input`: The input format.
 - `output`: The output format.

for most streams, only one is not null.


### Events

 - `inputDescriptionUpdated`: contains the stream, the new AudioStreamDescription.
 - `outputDescriptionUpdated`: contains the stream, the new AudioStreamDescription.
 - `processAudio`: contains the stream, an array of AudioBuffers to read from, and an array of AudioBuffers to write to.


### Notes

There is a lot of room for improvements here, reconfiguring the stream is an obvious first step. Another thing is actually exposing the device that the stream belongs to, which could include a device name and so on, but there are possible privacy aspects concerning that.

It is possible that output and input streams should be split into two different interfaces, but both APIs are essentially equivalent for most purposes.


Garbage Collector
--------------------------------------------------------------------------------

It can be important that the `processAudio` event is triggered just before garbage collection (depending on the length of the collection, and the buffer states) to allow the application to fill all buffers before the collection pause to minimize the risk for underflow.

If the application allocates significant amounts of memory during this callback, the garbage collector could trigger anyhow, but a careful programmer should be able to create pauseless playback in this manner.


Battery
--------------------------------------------------------------------------------

Running on mobile devices is important, and the API can easily handle different devices and usage scenarios, when a low-powered device needs audio, it simply provides larger buffers to the applications, increasing latency, but to compensate, it does not need to power up the processor as often.

In addition, a low-powered device could provide streams with lower sampling rates, which would in some cases could reduce the amount of processing that would be required.


Security
--------------------------------------------------------------------------------

I am not sure how you should request input device access from the user, and it is possibly out of scope of the API, but in a browser, the user needs to be asked for permission before any input device is activated or a massive privacy breach is bound to happen.

In addition, if additional information about audio streams were provided (like the audio hardware name) then it could be an information leak that when combined with other information, could uniquely identify a user.

Inside of something like Node.js, no additional permissions compared to a regular native applicaton would be required, so all of the API could be accessed by default.


Advantages
--------------------------------------------------------------------------------

Compared to the Web Audio proposal

 - It is a lot simpler to understand, and work with for basic Javascript audio playback.
 - Supports input.

Compared to the Media Stream proposal

 - It is a lot simpler to understand, and work with for basic Javascript audio playback.
 - Allows Javascript to generate samples outside of a worker.


Disadvantages
--------------------------------------------------------------------------------

 - It doesn't support a lot of built-in effects for example.
 - It does not provide a way to interact with media elements (but this could easily be fixed with some extra work).
 - It is a lower-level API that might require libraries to provide a higher-level API, for example for constructing processing graphs or an API for playing short effects for games.


Notes
--------------------------------------------------------------------------------

If you have any good ideas for names or otherwise, throw them in my direction on Twitter (@jensnockert), jens@aventine.se or here.

All events should be implemented with something like the DOM events and addEventListener, to make the system work as well together with normal web applications as possible.

Also, the API is not restricted to browsers outside of the interaction with media elements, a Node.js implementation should be possible and would be useful for certain desktop and server applications.
