---
layout: post
title:  "The many styles of video encoding"
date:   2019-10-31
excerpt: "And of course most people had to pick the wrong one"
tag:
- Fansubbing
- Technology
- Encoding
comments: true
---

If you have ever dabbled in the fine arts of either translation, editing, or typesetting, you've likely come to realize that there are as many ways to handle them as there are stars in the sky. The same, perhaps surprisingly, holds true for video encoding. A common misconception I hear that kinda puts me on edge is that people always want "the best quality video", which to anyone that knows what they're doing sounds like a very dumb thing for one simple reason: "The best" is, at least to some extent, subjective.

For the uninitiated, let's start with a short explanation of what video encoding entails in the fansubbing scene. The encoder takes care of the video and audio, and in some circles also generate the keyframes and maybe even do chapters. I've also seen places where the encoder is expected to mux everything by the end so that the quality control can look at the end result. They are in charge of making sure there's as few issues with the video as possible, and apply specialized filtering and encoding techniques in an attempt to obtain a reasonable file size for release. If you've ever seen a video that looks very bad, odds are high that the encoder behind it is the one to blame.

{% capture images %} {{ site.url }}/assets/res/2019-10-31-different-encodes/Sympho1BD_01.png {% endcapture %} {% include gallery images=images %}

Nothing brings the message home as well as *aWarpSharp2* does. Universally hated by older encoders, but newer encoders tend to like it for its ability to sharpen the entire clip decently well. The catch? It achieves this sharpness through warping. Depending on the encoder, this can either look *really* good or *really* bad. Here's an [example](https://slowpics.org/comparison/0862f36b-dd4c-4928-a341-0edce4553101):

#### Source clip
{% capture images %} {{ site.url }}/assets/res/2019-10-31-different-encodes/CRYSTAR_OP1_src.png {% endcapture %} {% include gallery images=images %}
#### Warped clip
{% capture images %} {{ site.url }}/assets/res/2019-10-31-different-encodes/CRYSTAR_OP1_warp.png {% endcapture %} {% include gallery images=images %}

Note that the lines get completely warped (more examples above). In some scenes, this actually looks kinda alright. A lot of detail is lost to the ether however, and some lines that weren't supposed to be mended together are now. Yet, some people think this looks much better than the source video.

This is just one of the more obvious filters. Another point of contention you sometimes get is whether or not you should add grain to a video. There are reasons to do this that can aid in preserving gradients and hiding artifacts, but some encoders simply like the way it looks.

#### Filtered (no grain)
{% capture images %} {{ site.url }}/assets/res/2019-10-31-different-encodes/CaseFilesBD_00_1080_no_grain.png {% endcapture %} {% include gallery images=images %}
#### Filtered (grained)
{% capture images %} {{ site.url }}/assets/res/2019-10-31-different-encodes/CaseFilesBD_00_1080_grain.png {% endcapture %} {% include gallery images=images %}

Filtering aside, you've also got the age-old fight of size versus quality. Some people don't care for video quality, they care for file size. Now whilst I would generally call them stupid, they serve as a good example here. Below is a comparison between *[Hakata Ramen]* and *[Kaleido-Flax]*'s release of *Kimetsu no Yaiba*. I made this comparison when the Hakata encoder kept insisting I look at his encode and compare them with mine. [The results are as follows.](https://slowpics.org/comparison/81290fa9-1e55-4361-ad52-5213d0e2e25b)

#### [Kaleido-Flax] (Size: 1.01 GB)
{% capture images %} {{ site.url }}/assets/res/2019-10-31-different-encodes/Yaiba_Kaleido.png {% endcapture %} {% include gallery images=images %}
#### [Hakata Ramen] (Size: 302.4 MB)
{% capture images %} {{ site.url }}/assets/res/2019-10-31-different-encodes/Yaiba_Hakata.png {% endcapture %} {% include gallery images=images %}

Now, is there anything wrong with this? Oh yeah, definitely. No question about it. But is it still acceptable? The answer to that is personal, and some people will side with "it looks good enough", especially when taking the file size into account. Similarly, you have people that prefer having much bigger file sizes to ensure every tiny bit of detail gets preserved as well as possible.

{% capture images %} {{ site.url }}/assets/res/2019-10-31-different-encodes/damn_that_bloooaaat.png {% endcapture %} {% include gallery images=images %}

... And as you can see, this may result in massive file sizes for some shows, whereas a smaller encode will likely be of the same perceivable quality to the majority of viewers.

So what's the takeaway from this? It's actually pretty simple: If you're a video encoder, try to find what works for you. It's fine if you can't find the perfect way to do it right from the get-go. If there are as many ways to encode as there are stars in the sky, there surely is a spark just for you.

<hr>

Please note that I do not intend to throw shade over either Hakata or SCY in this post. They just happened to be good examples for the point I was trying to make, and each have their own reasons for doing things the way they are.