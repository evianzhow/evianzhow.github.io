---
layout: post
title: Resolve Final Cut Pro share 10008 error
date: '2017-03-12 10:21:32'
permalink: "/resolving-final-cut-pro-share-10008-error/"
tags:
- final-cut-pro
---

Somehow, Final Cut Pro will fail to export (a.k.a. share) and giving you an error code: 10008. 

From [research](http://www.fcp.co/fcp-forum/4-final-cut-pro-x-fcpx/25722-exporting-error-rendering-error-10008), this error is caused by some glitchy or corrupted frames in your media file. But sometimes FCP **will not telling you which frame**, instead, it just yielding:
```
The share operation “XXX” has failed.
Video rendering error: 10008 (Final Cut Pro error 10008: renderVideoFrame failed)
```

To fix that problem, I encourage you share as "Master File" first because in my case, FCP yields more detailed error message with frame sequence number which blocks the whole sharing process. 

It should be something like this: 
```
The operation could not be completed because an error occurred when creating frame FF ((Error 10008f, HH:MM:SS;FF <<(Error -12909c, input.mp4 @ HH:MM:SS;FF)>>)).
```

It seems like this behavior vary from version to version. So try another format if FCP is not yielding as expected. 

Then, you go to **Preferences** and change **Time Display** to **Frames**. (If FCP yield error message containing different format, change to the correspondent one.)

Then you should zoom your timeline to the basic unit - frame.

Export as "Master File", switch to that specific frame, use **Blade** tool to cut that frame out and **disable** it. Try export again, if the problem persists, repeat above steps until export successfully. 

Personally I recommend copying last frame before the corrupted one, and pasting it after (same as repeat that frame) in order not to cause the flashing black screen. 

<blockquote class="twitter-tweet" data-lang="zh-cn"><p lang="en" dir="ltr"><a href="https://twitter.com/evianzhow">@evianzhow</a> <a href="https://twitter.com/_HairForceOne">@_HairForceOne</a> I really wish FCP group will enhance this process by providing an option to skip all corrupted frame.</p>&mdash; Evian Zhow (@evianzhow) <a href="https://twitter.com/evianzhow/status/840806279236923392">2017年3月12日</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>