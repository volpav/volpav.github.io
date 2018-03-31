---
layout: post
title: Sprites is Now Open-Source
redirect_from:
  - /blog/sprites-is-now-open-source
---

As some of you may’ve heard, back in May I’ve made a tough decision to [shut down Sprites](http://blog.spritesapp.com/2016/05/08/sprites-is-shutting-down.html) after almost two and a half years of working on it. I’m now excited to announce that the complete source code of the app as well as all the related services have been open-sourced and are now available on GitHub under [“Sprites” organization](https://github.com/spritesapp).

## Points of Interest

There’s “[Architecture and Design Notes](https://github.com/spritesapp/sprites/raw/master/Data/Architecture%20and%20Design%20Notes.pdf)” document which will give you a good overview of how the software is structured and what are some of the tools and libraries that were used (and why). There’s quite a few pieces that can potentially be reused (e.g. the whole rendering engine, support for live data feeds, voiceover support, etc.).

One of the greatest features is support for video export. There’s a [native component](https://github.com/spritesapp/sprites-snapshotter) that enables this - I encourage you to check it out, it’s somewhat generic and there’re some good notes on how it works.

## What's Next

I hope you can find some use in the source code. If you have any questions - don’t hesitate to reach out to me and I’ll clarify anything.
