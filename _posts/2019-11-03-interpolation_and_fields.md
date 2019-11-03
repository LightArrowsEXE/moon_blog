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

If you've been following the Nyaa buzz recently (at least, as of the time of this writing), you might have seen hot talk about a new show this season (which is ironically a sequel to a previous hot topic with season 1 of this show). That show is ***High Score Girl***. Now, what might people be fighting over this time? It's simple: Can an anime possibly have 60 fps content?

For the uninitiated, the topic of today covers interpolation, fields, and many things you'll also find when dealing with interlacing. Hence, knowing at least a little bit about that is expected here. If you need snuffing up on that, I suggest looking for other posts that explain it in more depth. For those that want a more bite-sized read on this particular topic, I suggest checking out [motbob's release on Nyaa](https://nyaa.si/view/1190356).

{% capture images %} {{ site.url }}/assets/res/2019-11-03-interpolation-and-fields/me_rn.png {% endcapture %} {% include gallery images=images %}

<hr>

For starters, let's get some of the lesser-known technobabble out of the way:

### <u>Fields</u>
In essence, a field is every horizontal row of a frame. Every 1920x1080 frame contains 1080 *fields* that are each 1920 pixels long. Usually you'll only talk about them in the case of frames being weaved together, like for example when dealing with interlacing.

{% capture images %} {{ site.url }}/assets/res/2019-11-03-interpolation-and-fields/infograph_new_to_interleaved.png {% endcapture %} {% include gallery images=images %}

Here is an example of two frames that were originally progressive of which the fields were weaved together. The third frame contains essentially half the information of both frames, combined into one frame.

### <u>Interleaving</u>
In the context of this post, it refers to the fields from two frames being woven together into a single frame.

### <u>Scan Order</u>
This dictates in what order the fields should be read. We won't really touch upon this here, but you can usually find this information with MediaInfo and filter accordingly.

<hr>

The topic of today's post is on a case where we have a true 60fps video on our hands. The catch? Rather than distribute it in glorious 60p chad video, they distributed a 30i virgin video with interleaved fields. If you are familiar with the way interlaced frames look, you'll be forgiven for thinking it looks like it was interlaced. You can verify if it is by simply separating the fields and checking for any new information, but usually there's no need to—how often are you going to run into something like that that *isn't* interpolated anyway? However, for *High School Girl*, that is definitely not the case.

Give [this video](https://files.catbox.moe/esh6p8.mp4) a watch. The left frame indicates the original clip (just wish duplicated frames for showcasing), the middle clip contains an interpolated clip, and the right contains a diff. If you pause throughout the video or framestep, you'll notice that the original clip shows what appear to be interlacing artifacts on *every. Single. Frame.* There is no pattern to it, unlike with interlacing. Every frame has some form of field weaving.

Now, by separating the fields and checking if every frame is unique, it quickly becomes apparent that the native framerate is 60 frames per second. This means you will either have to drop half the fields in every frame to get rid of the interlacing artifacts *or* separate every field and interpolate them back to 1080p.

### Dropping the fields

The usual option to go for if you can't deinterlace something properly is to simply drop half the fields and interpolate them back up. This can be done fairly easily with *nnedi3*.
```py
drop_the_fields = core.nnedi3.nnedi3(src, field=1)
```

This removes half the fields and interpolates it for you, which is pretty effective. However, the issue is that you might end up with stuttering during fluent animations or panning scenes. This will become especially apparent in scenes with a lot of motion.

### Interpolating the fields

{% capture images %} {{ site.url }}/assets/res/2019-11-03-interpolation-and-fields/infograph_interleaved_to_new.png {% endcapture %} {% include gallery images=images %}

In general, the best solution is splitting the fields and interpolating each of them. Ideally with motion estimation, which nnedi3 unfortunately does not offer. QTGMC does, though! The motion estimation helps a lot, since it keeps the frames before and after in mind when interpolating, offering a smoother playback.

```py
interpolate_the_fields = haf.QTGMC(src, TFF=True)
```

[Here is an example of dropping the fields versus interpolating them.](https://files.catbox.moe/uq6giy.mp4) Note how the left clip stutter more than the middle clip. In the middle clip you can see that the frames get interpolated pretty well, allowing for smooth 60fps playback, as was originally intended.

<hr>

So we've talked about how you should be interpolating these fields when dealing with High School Girl. But if it worked so well here, why don't we *always* do that, even with interlaced video? The answer is relatively simple. If you interpolate, you throw away half the information of a frame. With the usual interlaced video, there is no need for that. Every field you need to get the original frame back is there, just interlaced. 

*High School Girl* is a very special case. Since none of the standard broadcast methods support native 60 frames per second, you can't truly get all the information required to get the original frames. That simply isn't the case for interlacing methods like 3:2 pulldown, where you can get the correct fields easily.

{% capture images %} {{ site.url }}/assets/res/2019-11-03-interpolation-and-fields/interlacing_explained.png {% endcapture %} {% include gallery images=images %}

<hr>

After all of this hassle, you might be wondering: but why can't they just release it 60fps and get it over with? And the answer is as simple as it is boring: 60fps is not a standard for video on blu-rays or TV.

Yes, web players typically *can* handle it (you need not look further than YouTube to see that), however think about it like this: Why would the studio bother making yet another master for streaming companies when they'll at best be releasing the 30i master on BDs anyway? Now, I should clarify that this is a speculative reason for why they won't do that from my part, but I wouldn't be surprised if this were the case.

<hr>

Now, for a quick Q&A session! Questions are sourced from the wonderful Nyaa.si, where some commenters are rather confused and wish to find answers to their questions, and others are dumb as bricks and think they know it all! Basically, this is the place where I make fun of people and also answer some common questions I've heard and especially seen in this recent kerfuffle:

**Q:** If they go 60fps for this show, why don't they it for other shows too?<br>
**A:** Because 60fps isn't natively supported on any of the ways they distribute it other than web players. Why they did it for this particular show? Beats me.

**Q:** You said that the *game* scenes were all 60fps native, but does that really mean this should be encoded in 60fps?<br>
**A:** Ideally, no—you'd encode this with a variable framerate. However, that's not really a big issue here. Multiple duplicate frames compress extremely well, and that's what happens here too. Since a lot of the interpolated fields are just duplicate frames of the previous and next frame in normal scenes, they will compress pretty well.

**Q:** If we take subtitles from a 30 fps source, do we have to retime them for the 60fps video?<br>
**A:** Aegisub works with timestamps, but those alone aren't enough. Snapping to keyframes should fix up most of the timing, however.

<hr>

*Some Sources and other informative tidbits:*<br>
#1 source: I do the video encode thing a lot. Also common sense for encoders that know how to use stuff beyond HandBrake.

[motbob's Nyaa release, covering this topic as well](https://nyaa.si/view/1190356)<br>
[Captain Disillusion's video on interlacing](https://www.youtube.com/watch?v=5eu_KjKsnpM)<br>

I'll probably add more to this if I find interesting articles, videos, etc. I'm too tired to continue.