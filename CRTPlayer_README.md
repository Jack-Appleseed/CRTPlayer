# CRTPlayer for Miyoo Mini Plus (OnionOS)

A drop-in companion app for the OnionOS FFplay video player that adds a
CRT-style visual filter — scanlines, color pop, vignette, and film grain —
so your videos look like they're playing on an old tube TV.

Runs as a **separate app** alongside the normal Video Player, so you can
switch between clean playback and CRT mode from the main menu.



**Extract the zip contents into a new folder named CRTPlayer**, then place that folder in /App/ on your SD card.



Videos should be cropped to 640x480 in HandBrake for best results, image of HandBrake settings included in folder.

## 

## What it does

* Light scanlines (tuned to avoid color fringing on chroma-subsampled video)
* Color/saturation boost
* Vignette (darkened corners, like tube glass curvature)
* Film grain
* Subtle color balance shift for a warmer, less green-cast look

All done via ffplay's built-in `-vf` filtergraph — no source modification,
no recompiling, just a different launch command.

## 

## Requirements

* Miyoo Mini Plus running OnionOS
* The standard **FFplay / Video Player** app already installed
(this mod uses the same shared system ffplay binary — it does not bundle
its own)
* Credit to [Steward-fu](https://steward-fu.github.io/website/handheld/miyoo-mini/parasyte_build_ffplay.htm)
and [bobotrax](https://github.com/bobotrax/ffplay_Miyoo) for the original
FFplay port this builds on

## 

## Installation

1. Download this zip and extract its contents into a new folder named
`CRTPlayer`.
2. Copy that `CRTPlayer` folder into `/App/` at the root of your SD card,
so you end up with `/App/CRTPlayer/` containing `launch.sh`,
`config.json`, and `icon.png` directly (not nested another level deep).
3. Restart the device (or use Onion's app refresh if available).
4. You should see a new **CRTPlayer** icon in your Apps menu, alongside
the regular Video Player.
5. Videos are read from the same `Media/Videos` folder as the normal
player — no need to duplicate your files.

## 

## The filter chain (for tinkerers)

If you want to tweak the look yourself, this is the line to edit in
`launch.sh`:

```sh
./bin/ffplay -autoexit -vf "hflip,vflip,drawgrid=w=iw:h=4:t=1:c=black@0.2,hue=s=1.4,vignette,noise=alls=7:allf=t,colorbalance=rs=0.05:gs=0.04:bs=-0.05" -i "$selected\_file" $startTimer
```

Breakdown:

|Filter|What it does|Easy tweaks|
|-|-|-|
|`hflip,vflip`|Corrects screen orientation (device-specific, leave as-is)|—|
|`drawgrid=w=iw:h=4:t=1:c=black@0.2`|Scanlines|`h=` = spacing (higher = fewer lines), `@0.2` = opacity. Tighter spacing (`h=3`) looks more authentic but can cause red color fringing at higher opacity — see note below|
|`hue=s=1.4`|Saturation boost|`s=1.0` is neutral, higher = more color pop|
|`vignette`|Darkened corners|leave default, or see ffmpeg docs for angle/opacity params|
|`noise=alls=7:allf=t`|Film grain|`alls=` = strength (try 5–15)|
|`colorbalance=rs=0.05:gs=0.04:bs=-0.05`|Warm tint, corrects a slight green cast|small values only, this adds up fast|



**A note on scanline spacing vs. color fringing:** video on this device
decodes in 4:2:0 chroma-subsampled format, meaning color info is shared
across pairs of rows while brightness is per-row. Tighter/darker scanlines
(`h=3` and up in opacity) can create visible red/green color fringing as a
result. Lower opacity (`@0.2` and below) mostly avoids this even at tighter
spacing — if you want to try `h=3` for a denser look, start low on opacity
and increase gradually while watching for fringing on red content.



**Known limits of this build:** the OnionOS ffplay binary is a stripped
build and does *not* include several common ffmpeg filters — `eq`,
`rgbashift`, and `tmix` all fail to launch (silent crash back to the file
browser). `lutyuv` works and is a cheap way to add a highlight glow if you
want it: `,lutyuv=y='val\*1.05+5'`. A `boxblur`+`blend` bloom effect also
works and launches fine, but costs enough CPU to cause audio/video sync
drift — left out of the default build for that reason, but documented here
in case you want to experiment:
`,split\[a]\[b];\[b]boxblur=2:1\[bl];\[a]\[bl]blend=all\_mode=screen:all\_opacity=0.25`



**Performance note:** this is a single-core \~1.2GHz device. Heavier
filters (blur, format conversion for full chroma resolution, etc.) can
cause audio/video sync drift. If you add filters and notice lag or lip
sync issues, that's your signal to back off.

## Known limitations

* No live in-app toggle between filtered/unfiltered — this is a separate
app you switch to from the menu, not a runtime effect switch (ffplay's
built-in SELECT-button display modes are hardcoded in its source and
can't be extended without recompiling ffplay itself)
* Filter is baked in at launch, not adjustable per-video without editing
the script

## Credits

Filter chain and app packaging by \[Jack\_Appleseed].
Built on the OnionOS FFplay port originally by Steward-fu, packaged for
OnionOS by bobotrax and the OnionUI team.

