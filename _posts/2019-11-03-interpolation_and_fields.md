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
In essence, this is half the vertical information of a frame. Every 1920x1080 frame contains 1080 *fields*. By extracting the fields, you can use the information obtained from those to interpolate the original frame.

### <u>Interleaving</u>
In the context of this post, it refers to the fields from two frames being merged together into one frame.

### <u>Scan Order</u>
This dictates in what order the fields should be read.

## The meat of the story

The topic of today's post is, as you may imagine, one of those cases where we have a true 60fps video on our hands. The catch? Rather than distribute it in glorious 60fps chad video, they distributed a 30fps virgin video with interleaved fields. Here's an example of one of those frames:

{% capture images %} {{ site.url }}/assets/res/2019-11-03-interpolation-and-fields/OP_Interleaved_1.png {% endcapture %} {% include gallery images=images %}
#### Note: big! Open with care!

Your average encoder will likely immediately jump to the conclusion of "hold on a second, this looks interlaced. Time to throw a deinterlacer over it!" And while I do understand that notion and often think so myself, it is important to sometimes take a step back and try out some things beforehand.

A simple script like this returns the following:
```py
sep = core.std.SeparateFields(src, tff=True)
stack = lvf.stack_compare(sep[3522], sep[3523], make_diff=True)
```

{% capture images %} {{ site.url }}/assets/res/2019-11-03-interpolation-and-fields/OP_Interleaved_2.png {% endcapture %} {% include gallery images=images %}

Notice how the two fields, once separated, show completely different information? This holds true for every frame with game content. The character animation itself is animated on the far more familiar twos (or, well, sixes in this case because of the interpolation).

{% capture images %} {{ site.url }}/assets/res/2019-11-03-interpolation-and-fields/OP_Interleaved_3.png {% endcapture %} {% include gallery images=images %}
#### Note: very big! Open with care!

```py
def count(n, clip):
    return core.text.Text(clip, f'Field Number: {n}')

sep = core.std.SeparateFields(src, tff=True)
sep = core.std.FrameEval(sep, partial(count, clip=sep))
sep = core.std.AddBorders(sep, 2, 2, 2, 2) # Adding borders to make different frames clearer

# This was painful and could probably be done in a way easier way
stacka = lvf.stack_compare(sep[1024], sep[1025], sep[1026], sep[1027], sep[1028], sep[1029], sep[1030], sep[1031], sep[1032], sep[1033], sep[1034], sep[1035], sep[1036], sep[1037], sep[1038])
stackb = lvf.stack_compare(sep[1039], sep[1040], sep[1041], sep[1042], sep[1043], sep[1044], sep[1045], sep[1046], sep[1047], sep[1048], sep[1049], sep[1050], sep[1051], sep[1052], sep[1053])

stack = lvf.stack_compare(stacka, stackb, stack_vertical=True)
```

From this information alone we can already surmise that yes, this is very likely native 60fps.

However, that alone isn't enough. Take note of how we now have double the temporal resolution at the cost of half the vertical resolution. This is where interpolation comes into play.

{% capture images %} {{ site.url }}/assets/res/2019-11-03-interpolation-and-fields/OP_Interleaved_4.png {% endcapture %} {% include gallery images=images %}

```py
# This should ideally be further optimized, but I'm too lazy to do so for this example.
interp = haf.QTGMC(src, TFF=True)
```

The frames here get interpolated pretty well, allowing for smooth 60fps playback, as was originally intended.

After all of this hassle, you might be wondering: but why can't they just release it 60fps and get it over with? And the answer is as simple as it is boring: 60fps is not a standard for video on blu-rays or TV.

Yes, web players typically *can* handle it (you need not look further than YouTube to see that), however think about it like this: Why would the studio bother making yet another master for streaming companies when they'll at best be releasing the 30fps master on BDs anyway? Now, I should clarify that this is a speculative reason for why they won't do that from my part, but I wouldn't be surprised if this were the case.

## Q&A Time!

Now, for a quick Q&A session! Questions are sourced from the wonderful Nyaa.si, where some commenters are rather confused and wish to find answers to their questions, and others are dumb as bricks and think they know it all! Basically, this is the place where I make fun of people and also answer some common questions I've heard and especially seen in this recent kerfuffle:

**Q:** If they go 60fps for this show, why don't they it for other shows too?<br>
**A:** Because 60fps isn't natively supported on any of the ways they distribute it other than web players. Why they did it for this particular show? Beats me.

**Q:** Since the fields are 540p once split, does that mean the native resolution is 1920x540p too?<br>
**A:** While understandable that someone might reach this conclusion, it should be noted that the interleaved fields in a frame are combined from two frames. This means that the original frame had a resoluton of 1920x1080, but only half the vertical rows of pixels were eventually mixed together (so a field from one frame interleaves with a field from another frame per frame, if that makes sense).

**Q:** I am still skeptical because the video I saw was 900p, so it was likely re-encoded, meaning frames were likely added.<br>
**A:** That is not a question. Get out of my studio.

**Q:** You said that the *game* scenes were all 60fps native, but does that really mean this should be encoded in 60fps?<br>
**A:** Ideally, no—you'd encode this with a variable framerate. However, that's not really a big issue here. Multiple duplicate frames compress extremely well, and that's what happens here too. Since a lot of the interpolated fields are just duplicate frames of the previous and next farme in normal scenes, they will compress pretty well.

**Q:** I’ve never ever seen any difference between the two (30/60) and one compresses better with lesser filesize<br>
**A:** While not a question either, this is something I read in a rant someone posted, and it's worth breaking up and explaining. 1) The differences between 30 and 60fps can be noticeable depending on how it was interpolated. In *High Score Girl's* case, the game scenes are very obviously 60fps, as I have demonstrated earlier in this post. 2) While yes, the less frames you have, the smaller the file size, the way it's framed here is very misleading. As I mentioned in the previous **A**, duplicate frames compress extremely well. This is because of the way compression is done for video, but that's a topic for another day. If you're interested in reading up on it, I suggest you read up on how [intra-frame coding](https://en.wikipedia.org/wiki/Intra-frame_coding) works. It's an interesting read. 3) As pointed out by a fellow commenter, just because you're blind doesn't mean everyone else is.

**Q:** If we take subtitles from a 30fps source, do we have to retime them for the 60fps video?<br>
**A:** To be quite honest, I don't know. My gut feeling tells me "yes", but it depends on how Aegi decides when to play back a line. If it works based on timestamps, you'll be fine. If not, you're gonna be spending your evening retiming.

**Q:** You're still wrong and I'd like to engage in the fisticuffs with you!<br>
**A:** ***SQUARE UP, THOT.*** Memes aside, I can be found in the [Weeb Autism Discord Server](https://discord.gg/ZB7ZXbN), and my PMs are always open at LightArrowsEXE#0476.

<hr>

*Some Sources and other informative tidbits:*<br>
#1 source: I do the video encode thing a lot. Also common sense for encoders that know how to use stuff beyond HandBrake.

[motbob's Nyaa release, covering this topic as well](https://nyaa.si/view/1190356)<br>
[Captain Disillusion's video on interlacing](https://www.youtube.com/watch?v=5eu_KjKsnpM)<br>

I'll probably add more to this if I find interesting articles, videos, etc. I'm too tired to continue.