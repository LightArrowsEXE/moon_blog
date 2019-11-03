---
layout: post
title:  "A field day with interpolation"
date:   2019-11-03
excerpt: "Explaining what armchair encoders can't."
tag:
- Technology
- Encoding
comments: true
---

If you've been following the Nyaa buzz recently (at least, as of the time of this writing), you might have seen people that don't know what they're talking about complaining about people spreading "fake 60fps anime". Now, while it's understandable that one might want to nip conspiracies like these in the bud if possible, it quickly becomes very apparent who knows what they're talking about and who doesn't when it turns out that yes; the anime actually *does* have native 60fps scenes.

For the uninitiated, the topic of today covers interpolation, fields, and why armchair encoders are the worst blight on the fansubbing scene. The show for today: ***Hi Score Girl***. For those that want a more bite-sized read, I suggest checking out [motbob's release on Nyaa](https://nyaa.si/view/1190356).

{% capture images %} {{ site.url }}/assets/res/2019-11-03-interpolation-and-fields/me_rn.png {% endcapture %} {% include gallery images=images %}

## It's dangerous to go alone, take some terminology!

For starters, let's get some of the lesser-known technobabble out of the way:

### <u>Fields</u>
In essence, a field is every horizontal row of a frame. Every 1920x1080 frame contains 1080 *fields*. Usually you'll only talk about them in the case of frames being interleaved, blending two frames together.

[Here is an example of two frames that were originally progressive, but were interlaced afterwards](https://files.catbox.moe/y3u92t.png) (by me. I'm so sorry, Illya.)

### <u>Interleaving</u>
In the context of this post, it refers to the fields from two frames being merged together into one frame.

### <u>Scan Order</u>
This dictates in what order the fields should be read.

## The meat of the story

The topic of today's post is, as you may imagine, one of those cases where we have a true 60fps video on our hands. The catch? Rather than distribute it in glorious 60p chad video, they distributed a 30i virgin video with interleaved fields. Here's an example of one of those frames:

{% capture images %} {{ site.url }}/assets/res/2019-11-03-interpolation-and-fields/OP_interleaved_1.png {% endcapture %} {% include gallery images=images %}


Your average encoder will likely immediately jump to the conclusion of "hold on a second, this looks interlaced. Time to throw a deinterlacer over it!" And while I do understand that notion and often think so myself, it is important to sometimes take a step back and try out some things beforehand.

A simple script like this returns the following:
```py
# Note: Shifting by 1px vertically because fields will always be 1px off from eachother due to the way they're handled
sep = core.std.SeparateFields(src, tff=True)
stack = lvf.stack_compare(sep[3521], sep[3522].resize.Point(src_top=1), make_diff=True, stack_vertical=True)
```

{% capture images %} {{ site.url }}/assets/res/2019-11-03-interpolation-and-fields/OP_interleaved_2.png {% endcapture %} {% include gallery images=images %}

Notice how the two fields, once separated, show completely different information? This holds true for every frame with game content. The character animation itself is animated on the far more familiar twos (or, well, fives in this case because of the interpolation).

**EDIT:** After some more research, it actually appears that the foreground assets move at instances of 2-3-2-3-2-3 frames, indicating that the foreground animation is done at 24p, and every 4th frame is duplicated. Backgrounds appear to be done in 30p. Interesting stuff.

**Note:** big! Open with care!<br>
[Link to image](https://files.catbox.moe/64d6og.png)


```py
def count(n, clip):
    return core.text.Text(clip, f'Field Number: {n}')

sep = core.std.SeparateFields(src, tff=True)
sep = core.std.FrameEval(sep, partial(count, clip=sep))
sep = core.std.AddBorders(sep, 2, 2, 2, 2) # Adding borders to make different frames clearer

stacka = lvf.stack_compare(*sep[1024:1039])
stackb = lvf.stack_compare(*sep[1039:1054])

stack = lvf.stack_compare(stacka, stackb, stack_vertical=True)
```

From this information alone we can already surmise that yes, this is very likely native 60fps.

However, that alone isn't enough. Take note of how we now have double the temporal resolution at the cost of half the vertical resolution. This is where interpolation comes into play.

**Note 1:** big! Open with care!<br>
**Note 2:** Downscaled to 720p equivalents because >over 100MB images<br>
[Link to image](https://files.catbox.moe/uip8jr.png)

```py
# This should ideally be further optimized, but I'm too lazy to do so for this example.
interp = haf.QTGMC(src, TFF=True)
```

The frames here get interpolated pretty well, allowing for smooth 60fps playback, as was originally intended.

After all of this hassle, you might be wondering: but why can't they just release it 60fps and get it over with? And the answer is as simple as it is boring: 60fps is not a standard for video on blu-rays or TV.

Yes, web players typically *can* handle it (you need not look further than YouTube to see that), however think about it like this: Why would the studio bother making yet another master for streaming companies when they'll at best be releasing the 30i master on BDs anyway? Now, I should clarify that this is a speculative reason for why they won't do that from my part, but I wouldn't be surprised if this were the case.

## Q&A Time!

Now, for a quick Q&A session! Questions are sourced from the wonderful Nyaa.si, where some commenters are rather confused and wish to find answers to their questions, and others are dumb as bricks and think they know it all! Basically, this is the place where I make fun of people and also answer some common questions I've heard and especially seen in this recent kerfuffle:

**Q:** If they go 60fps for this show, why don't they it for other shows too?<br>
**A:** Because 60fps isn't natively supported on any of the ways they distribute it other than web players. Why they did it for this particular show? Beats me.

**Q:** You said that the *game* scenes were all 60fps native, but does that really mean this should be encoded in 60fps?<br>
**A:** Ideally, no—you'd encode this with a variable framerate. However, that's not really a big issue here. Multiple duplicate frames compress extremely well, and that's what happens here too. Since a lot of the interpolated fields are just duplicate frames of the previous and next farme in normal scenes, they will compress pretty well.

**Q:** If we take subtitles from a 30 fps source, do we have to retime them for the 60fps video?<br>
**A:** Aegisub works with timestamps, but those alone aren't enough. Snapping to keyframes should fix up most of the timing, however.

<hr>

*Some Sources and other informative tidbits:*<br>
#1 source: I do the video encode thing a lot. Also common sense for encoders that know how to use stuff beyond HandBrake.

[motbob's Nyaa release, covering this topic as well](https://nyaa.si/view/1190356)<br>
[Captain Disillusion's video on interlacing](https://www.youtube.com/watch?v=5eu_KjKsnpM)<br>

I'll probably add more to this if I find interesting articles, videos, etc. I'm too tired to continue.