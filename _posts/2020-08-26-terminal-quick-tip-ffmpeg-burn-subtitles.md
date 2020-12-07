---
layout: "post"
author: "jimdoescode"
title: "Terminal Quick Tip - Burn Subtitles into a video stream with ffmpeg"
date: "2020-08-26 13:50:14"
tags: [quicktip,terminal,ffmpeg,bash]
---

Say you've got a video file with subtitles in it. But streaming said video doesn't let you display the subtitles. Burning the subtitles into the video stream of the file will fix this.

```sh
$ ffmpeg -i Video\ with\ Subtitles.mkv -vcodec copy -acodec copy -vf subtitles=Video\ with\ Subtitles.mkv New\ Video\ with\ Subtitles.mkv
```

There are a couple things to note with this command.
 * I added the `vcodec` and `acodec` sections jic you want to change the encoding on either of those streams in the file.
 * To use an alternate subtitle stream: `subtitles=video.mkv:si=X` where `X` is the zero based index of all the subtitle streams in the video file.  
 * It's probably most helpful to first run an `ffprobe` on the video file to see where all the streams are.

For more options on how to burn subtitles into a video stream check out the ffmpeg wiki [http://trac.ffmpeg.org/wiki/HowToBurnSubtitlesIntoVideo]

Also check out the ffmpeg docs on the subtitles filter [http://ffmpeg.org/ffmpeg-filters.html#subtitles-1]
