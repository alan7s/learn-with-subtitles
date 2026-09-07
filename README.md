# Learn with Subtitles

A video player for learning a language by typing its subtitles: watch the
line, type the primary subtitle to unlock the rest of the video, and read
the secondary subtitle overlaid on it. Works with any language pair — it's
not tied to any specific languages.

## Usage (no server)

Open `index.html` directly in your browser (double-click it) and pick the
files:

- **Video** — any format your browser can play (mkv, mp4…)
- **Primary subtitle** (`.srt`) — the one you'll type
- **Secondary subtitle** (`.srt`) — the one shown overlaid on the video
- **Alternate audio** (optional, you can pick more than one file) — only
  needed if you want to switch the audio track/language (see why below).
  Once loaded, an "Active audio track" dropdown appears to switch between
  the video's original audio and the alternates you loaded.

The video starts playing automatically once a video and a primary subtitle
are loaded. You can drag the player's native progress bar to jump forward or
back at any time — the app resyncs on its own which line it's gating on.

## How it works

- The video plays normally until the end of each primary-subtitle line, then
  pauses and asks you to type the exact text (IME composition is supported,
  e.g. Hangul, Kana, Pinyin). Get it right, or press **Tab**/"Skip line", and
  the video plays on to the end of the next line.
- The secondary subtitle is overlaid on the video the whole time, with no
  gating.
- Chrome doesn't expose extra audio tracks from containers like MKV through
  a native `<video>` element (it only plays the file's default track). So to
  switch audio language, extract the track you want into a separate file and
  load it in the "Alternate audio" field — the app plays that file in sync
  with the video (which is muted while it does).

## Extracting subtitles and audio from an .mkv file

An `.mkv` can bundle several subtitle/audio tracks. Use
[ffmpeg](https://ffmpeg.org/) to see what's inside and pull out what you need:

**1. List the file's tracks:**

```
ffprobe -v error -show_entries stream=index,codec_type,codec_name:stream_tags=language,title video.mkv
```

This shows the `index` (number), `codec_type` (`audio`/`subtitle`/`video`),
and language/title of each track. Note the `index` of the ones you want.

**2. Extract a subtitle to `.srt`** (replace `N` with the track's index):

```
ffmpeg -i video.mkv -map 0:N subtitle.srt
```

Works for text-based tracks (SubRip, ASS/SSA). Image-based subtitle tracks
(PGS/VobSub) can't be converted to text — those won't work here.

**3. Extract an audio track to `.m4a`** (no re-encoding — fast and lossless):

```
ffmpeg -i video.mkv -map 0:N -c copy -bsf:a aac_adtstoasc alt_audio.m4a
```

`-bsf:a aac_adtstoasc` is only needed if the track is AAC (the most common
case in mkv files); if it errors, try without it. If the track uses another
codec (e.g. AC3/DTS) and the browser won't play the resulting `.m4a`,
re-encode to AAC instead:

```
ffmpeg -i video.mkv -map 0:N -c:a aac -b:a 192k alt_audio.m4a
```

Then just load the generated `.srt`/`.m4a` files in the matching fields of
`index.html`.
