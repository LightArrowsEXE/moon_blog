---
layout: post
title:  "mpv made easy"
date:   2019-09-12
excerpt: "aka why you should use mpv over literally anything else"
tag:
- Fansubbing
- Technology
comments: true
---

An important aspect of playing back videos is the video player. There’s many options, but there’s typically three that people levitate towards. Those are **VLC**, **MPC**, and **mpv**. Now if you ask any fansubber worth their salt, they will tell you that there is only one option: mpv. However, there’s a caveat: in order to configure mpv, your average user will have to step outside of his comfort zone, and that can be tough.

There are a couple things that fansubbers will preach about in regards to mpv, but since that’s not the main point of the post, I will skim over them really quickly:
* mpv uses ffmpeg in its backend, allowing it to play pretty much any media file under the sun.
* mpv uses libass for subtitle rendering by default, which is what most of the bigger groups are moving towards over the years.
* mpv is highly configurable, allowing you to customize it however you wish.
* The GUI is highly minimalistic, removing a lot of bells and whistles that just confuse your average user.

Now, configuring mpv is one of the reasons most people will tell you to "just use VLC". Whilst yes, VLC works right out of the box, a common misconception is that mpv does not. mpv works just as well out of the box as its alternatives do, but what makes it seem like it isn’t is because you’ll be lacking a lot of bells and whistles. These can be added back fairly simple with an `mpv.conf`. This is a configuration file where you can configure mpv to your liking.

An example of an `mpv.conf` can be found [here](https://github.com/LightArrowsEXE/dotfiles/blob/master/mpv/.config/mpv/mpv.conf) (but don’t mindlessly copy it just yet).

This might look daunting for a beginner, but in actuality it’s very simple. mpv is a command-line program at heart. When run from the command-line you can set flags, which allows you to turn specific settings on and off. A configuration file acts as a middleman in that regard. mpv will read that by default and carry over all of its settings, regardless of whether a video is run from the command-line or not.

# Preparing your mpv.conf
You can prepare the mpv.conf in two locations.

The first is to just dump it in the same directory as your mpv.exe. This is the easiest and most obvious way, but can get messy pretty quickly.

The second is the more preferred way. Open your explorer and in the top bar write down `%appdata%`. This will open your `Appdata/roaming` directory. In there you need to look for a directory called `mpv`. If it doesn’t exist, you can create it yourself. This is the directory where we will dump our settings and anything else mpv may need.

# Settings

We will only cover some basic settings in this post. If you wish to delve deeper, I highly suggest you take a look at [the official documentation](https://mpv.io/manual/master/#configuration-files).

## General
For starters, it is *highly* recommended that you set `profile=gpu-hq`. This will set a lot of a settings by default optimized for quality using your GPU. This alone should be enough for your average viewer to run everything at the highest quality they can.

There are a couple of additional settings you may want to set based on your preference.

`keep-open=yes` will ensure your player stays open, even after a video is done playing.

`keepaspect` will keep the aspect ratio of the video intact when scaling the window.

`curser-autohide=100` will automatically hide the curser while playing a file.

*My mpv.conf:*
```
# General
profile=gpu-hq
vo=gpu
#gpu-api=vulkan
vd-lavc-dr=yes
spirv-compiler=auto
cursor-autohide=100
keep-open=yes
force-window=yes
msg-module
msg-color
keepaspect
```

## Subtitles

Sometimes you will run into subtitles that look bad, and so you want to change them. There’s a couple settings for that.

`sub-ass-override=yes` will always override subtitle styling. set to `no` to prevent it from doing that, which may be nice to set just in case.

`sub-font` allows you to set a font. For example `sub-font=’Gandhi Sans’`. You can find the name by checking Fonts on Windows.

`sub-font-size` sets the size of the subtitles. Sane values are between 42 and 64.

*My mpv.conf:*
```
# Subtitles
sub-ass-override=no
sub-font=’Gandhi Sans’
sub-blur=0.2
sub-color=0.95
sub-border-color=0.05/0.05/0.03
sub-border-size=2.2
sub-bold
sub-spacing=0.6
sub-margin-x=180
sub-margin-y=36
sub-font-size=43
```

## Language Priority

These settings set priority on specific languages. This is nice if you prefer watching dual-audio releases and want the dub to play by default, or other similar situations. Also useful to forcibly set to your preferences either way.

`slang` sets the priority for the *subtitle tracks*. For English, you’ll want to look at `en` and `eng`. There’s also `enm`, which is usually used for secondary English tracks. Think for dubbed releases with a track that translates the signs, or a release that has a no-honorific track and a honorific track.

`alang` sets the priority for the *audio tracks*. The logic is the same as `slang`. For Japanese, you’ll want `ja`, `jp`, and `jpn`.

There is also a `vlang`, however there is almost never a language set for a video track. This can be safely skipped.

*My mpv.conf:*
```
# Priority
slang=en,eng
alang=ja,jp,jpn,en,eng
```

## Screenshots

Even taking screenshots can be optimized as much as you want.

`screenshot-format` sets the format. It is highly recommended that you set this to `png`. By default it’s set to `jpeg`, which heavily compresses the image. png on the other hand is a lossless format.

`screenshot-directory` allows you to set a path to a directory where every screenshot will be placed. By default this is either in the directory that mpv was started in (with CMD) or the desktop (with GUI).

*My mpv.conf:*
```
# Screenshot
screenshot-format=png
screenshot-high-bit-depth=no
screenshot-png-compression=9
screenshot-directory="/media/Data/home/mpv/"
screenshot-directory="D:/Users/light/Pictures/mpv"
screenshot-template=vlcsnap-20%ty-%tm-%td-%tHh%tMm%tSs # Totally not trying to bait people here
```

## Scaling

Scaling is a surprisingly big reason that mpv is recommended by encoders. With more and more encoders opting to encode video in their native resolutions rather than 1080p, you will often find that the highest-quality release is below 1080p. That will require upscaling. There’s two ways to go about it. One works with built-in scalers, and the other with external *shaders* (a whole topic for another time, as it’s generally far above what normal users will bother with). We will cover some options for both here. Note that built-in scalers will require `vo=gpu` or `vo=opengl=cb` to be set.

These are the recommended settings depending on your needs (courtesy of eXmendiC’s post):


| PRESETS LUMA | UPSCALE LUMA | DOWNSCALE CHROMA | UP- & DOWNSCALE |
| ------------ | ------------ | ---------------- | --------------- |
| ***Experimental*** | FSRCNNX_x2_16-0-4-1 | SSimDownscaler | KrigBilateral |
| ***Very High*** | ravu | ewa_lanczossharp | ravu |
| ***High*** | ewa_lanczossharp | ewa_lanczos | ewa_lanczossoft |
| ***Medium*** | spline16 |	spline16 | sinc (with blackman) |
| ***Low*** | catmull_rom | catmull_rom | catmull_rom |
| ***Very Low*** | bicubic_fast	| bicubic_fast | bicubic_fast |

`scale` sets the scaler used for *upscaling*. There are only two scalers worth looking at here, which are `spline36` (used by default with `profile=gpu-hq`), (`ewa_`)`lanczos`. Both have their strengths and weaknesses, so compare the two and pick which one you prefer most.

`dscale` sets the scaler used for *downscaling*. The above options are still solid here, but `mitchell` also becomes useful here. `mitchell` is essentially bicubic, and the `b` and `c` values can be set with `scale-param1` and `scale-param2`.

`cscale` sets the scaler used for *the chroma*. I recommend `sinc`.

*My mpv.conf:*
```
# Resizer
scale=ewa_lanczos
# alternative dscale: mitchell
dscale=ewa_lanczos
cscale=sinc
cscale-window=blackman
cscale-radius=3
```

There are a couple of *shaders* that allow for better overall upscaling. These will have to be downloaded separately, and can be found [here](https://github.com/bjin/mpv-prescalers/tree/master). You can pick your shader based on the above table.

We won’t go into too much detail about shaders here (which offer even *more* options), but the basic gist of it is that you create a directory inside your mpv directory called "shaders". You then import that shader by setting `glsl-shader`.

*My mpv.conf:*

```
glsl-shader="~~/shaders/ravu-r4.hook"
```

You can check if your scalers work by opening up a video and pressing <kbd>shift</kbd>+<kbd>i</kbd>, and then <kbd>2</kbd>. If you’ve set everything up correctly, it should show them in the list. Note that some shaders will require the final resolution to be x times larger than the original resolution. 720p videos will generally serve well enough for this.

{% capture images %} {{ site.url }}/assets/res/2019-09-30-mpv-made-easy/SymphoXD_scalers.png {% endcapture %} {% include gallery images=images %}

<hr>


There’s a lot of amazing stuff you can do with mpv, and this only covers the basics for your average user. We haven’t even started talking about shaders, scaling, etc! If you are interested in these, you can check out the official documentation linked above, or read [eXmendiC’s post](https://iamscum.wordpress.com/guides/videoplayback-guide/mpv-conf/.)