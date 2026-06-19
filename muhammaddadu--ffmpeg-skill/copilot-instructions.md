## ffmpeg-skill

> Make polished videos with ffmpeg: a production pipeline plus vetted recipes for conversion, scaling, compositing, GIFs, captions, audio/voiceover, motion, and platform-ready export — for demos, ads, tutorials, launches, social clips, explainers, and pitches.


# ffmpeg Usage

## Overview

Two things in one skill: a **command library** of vetted ffmpeg recipes for everyday media jobs, and a **production pipeline** for making videos that actually look intentional. Treat ffmpeg as the final compositor and encoder — the renderer — and bring in helper tools for the parts ffmpeg is clumsy at (asset design, animation, captions, 3D). Use it whenever a request touches media: converting, resizing, building GIFs, pulling audio, editing, or producing a finished video for marketing, demos, tutorials, launches, social, explainers, app-store previews, or pitches.

**Version:** 2.0.0
**Needs:** ffmpeg ≥ 4.0; `ffprobe` recommended. Other tools (below) are optional and used only when they earn their place.

## When to apply

Trigger on requests to:

- Convert, rescale, or re-fit aspect ratio
- Build a GIF, extract or convert audio
- Edit clips — trim, merge, speed, rotate
- Work with subtitles or captions — burn-in, soft, extract, generate
- Compress, web-optimise, or format for a platform (YouTube, Instagram, TikTok, X)
- Grab thumbnails or frames; batch-process a folder
- **Plan and produce a finished video** — story beats, scene composition, animated text, framing, voiceover/music, and a repeatable render pipeline

## Prerequisites

```bash
brew install ffmpeg              # macOS
sudo apt-get install ffmpeg      # Debian/Ubuntu
choco install ffmpeg             # Windows
ffmpeg -version                  # confirm
```

---

# Command library

## 1. Format conversion

**MP4 → WebM:**
```bash
ffmpeg -i input.mp4 -c:v libvpx-vp9 -crf 30 -b:v 0 -c:a libopus output.webm
```

**MOV → MP4:**
```bash
ffmpeg -i input.mov -c:v libx264 -c:a aac -strict experimental output.mp4
```

**Anything → MP4 (broad compatibility):**
```bash
ffmpeg -i input.* -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k output.mp4
```

## 2. Resolution

Rescale while preserving aspect ratio.

**To 720p / 1080p:**
```bash
ffmpeg -i input.mp4 -vf scale=-1:720 -c:a copy output_720p.mp4
ffmpeg -i input.mp4 -vf scale=-1:1080 -c:a copy output_1080p.mp4
```

**Fixed width, auto height:**
```bash
ffmpeg -i input.mp4 -vf scale=1280:-1 -c:a copy output.mp4
```

**Fit with letterbox padding:**
```bash
ffmpeg -i input.mp4 -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" output.mp4
```

## 3. GIFs

Palette generation is what keeps GIFs sharp and small.

**Quick (10 fps):**
```bash
ffmpeg -i input.mp4 -vf "fps=10,scale=480:-1:flags=lanczos" output.gif
```

**Two-pass with palette:**
```bash
ffmpeg -i input.mp4 -vf "fps=10,scale=480:-1:flags=lanczos,palettegen" palette.png
ffmpeg -i input.mp4 -i palette.png -filter_complex "fps=10,scale=480:-1:flags=lanczos[x];[x][1:v]paletteuse" output.gif
```

**From a time range (single pass):**
```bash
ffmpeg -ss 00:00:10 -t 5 -i input.mp4 -vf "fps=10,scale=480:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" output.gif
```

## 4. Audio

**Extract to MP3 / WAV:**
```bash
ffmpeg -i input.mp4 -vn -acodec libmp3lame -q:a 2 output.mp3
ffmpeg -i input.mp4 -vn -acodec pcm_s16le -ar 44100 -ac 2 output.wav
```

**Convert format:**
```bash
ffmpeg -i input.wav -c:a aac -b:a 192k output.m4a
```

**Replace audio with a track / mix two together:**
```bash
ffmpeg -i video.mp4 -i music.mp3 -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 -shortest output.mp4
ffmpeg -i video.mp4 -i music.mp3 -filter_complex "[0:a][1:a]amix=inputs=2:duration=first" -c:v copy output.mp4
```

## 5. Editing

**Trim (stream-copy, no re-encode):**
```bash
ffmpeg -i input.mp4 -ss 00:00:10 -to 00:00:30 -c copy output.mp4   # 10s–30s
ffmpeg -i input.mp4 -ss 00:00:05 -t 10 -c copy output.mp4          # 10s from 5s
```

**Concatenate** — pick by container:

```bash
# Protocol — TS/MPEG/MP3/AAC, no list file needed:
ffmpeg -i "concat:file1.mp3|file2.mp3|file3.mp3" -c copy output.mp3

# Demuxer — MP4/MOV/MKV, via process substitution:
ffmpeg -f concat -safe 0 -i <(printf "file '%s'\n" video1.mp4 video2.mp4 video3.mp4) -c copy output.mp4
# Fallback if the shell lacks process substitution:
printf "file '%s'\n" video1.mp4 video2.mp4 video3.mp4 > list.txt
ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp4 && rm list.txt

# Filter — mismatched codecs/resolutions, re-encodes:
ffmpeg -i video1.mp4 -i video2.mp4 -i video3.mp4 \
  -filter_complex "[0:v][0:a][1:v][1:a][2:v][2:a]concat=n=3:v=1:a=1[v][a]" \
  -map "[v]" -map "[a]" output.mp4
```

Decision: MP3/AAC/TS/MPEG → protocol; MP4/MOV/MKV → demuxer; differing codecs or sizes → filter.

**Speed / rotate:**
```bash
ffmpeg -i input.mp4 -filter:v "setpts=0.5*PTS" -an output.mp4   # 2x
ffmpeg -i input.mp4 -filter:v "setpts=2.0*PTS" output.mp4       # 0.5x
ffmpeg -i input.mp4 -vf "transpose=1" output.mp4               # 90° CW
ffmpeg -i input.mp4 -vf "transpose=2,transpose=2" output.mp4   # 180°
```

## 6. Subtitles

```bash
ffmpeg -i input.mp4 -vf subtitles=subtitles.srt output.mp4       # burn in
ffmpeg -i input.mp4 -i subtitles.srt -c copy -c:s mov_text output.mp4   # soft subs
ffmpeg -i input.mp4 -map 0:s:0 subtitles.srt                     # extract
```

## 7. Thumbnails

```bash
ffmpeg -i input.mp4 -ss 00:00:05 -vframes 1 thumbnail.jpg   # one frame
ffmpeg -i input.mp4 -vf fps=1/10 thumb%04d.jpg              # every 10s
ffmpeg -i input.mp4 -vframes 10 frame%04d.png              # first 10 frames
```

## 8. Compression & web

```bash
ffmpeg -i input.mp4 -c:v libx264 -crf 23 -preset medium -c:a aac -b:a 128k output.mp4    # balanced
ffmpeg -i input.mp4 -c:v libx264 -crf 28 -preset veryslow -c:a aac -b:a 96k output.mp4   # smaller
ffmpeg -i input.mp4 -c:v libx264 -preset medium -crf 23 -movflags +faststart -c:a aac -b:a 128k output.mp4   # web
```

## Platform presets

**YouTube:**
```bash
ffmpeg -i input.mp4 -c:v libx264 -preset slow -crf 18 -c:a aac -b:a 192k -pix_fmt yuv420p -movflags +faststart youtube.mp4
```

**Instagram Story (9:16, ≤15s):**
```bash
ffmpeg -i input.mp4 -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2" -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k -t 15 instagram_story.mp4
```

**X / Twitter (16:9, ≤2:20):**
```bash
ffmpeg -i input.mp4 -vf scale=1280:720 -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k -t 140 twitter.mp4
```

**TikTok (9:16, ≤60s):**
```bash
ffmpeg -i input.mp4 -vf "scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2" -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k -t 60 tiktok.mp4
```

## Quick recipes

```bash
# Shrink a screen recording
ffmpeg -i screen_recording.mov -c:v libx264 -preset medium -crf 23 -vf "scale=1920:-1" -c:a aac -b:a 128k optimised.mp4

# Batch-convert a folder
for i in *.mov; do ffmpeg -i "$i" -c:v libx264 -crf 23 -c:a aac "${i%.mov}.mp4"; done

# Images → video
ffmpeg -framerate 30 -pattern_type glob -i '*.jpg' -c:v libx264 -pix_fmt yuv420p output.mp4   # sequence
ffmpeg -loop 1 -i image.jpg -c:v libx264 -t 5 -pix_fmt yuv420p output.mp4                      # 5s from one image
```

---

# Polished video production workflow

For good-looking videos, use ffmpeg as the final compositor/encoder, not as the only creative tool.

1. **Define the story beats.** One clear idea per beat: hook, problem, solution, proof, call to action. Write the through-line before touching a command.
2. **Gather source media** — screen recordings, product shots, images, AI/stock clips, or generated backgrounds. Capture clean footage: use a release build, hide debug overlays, cursors, and notifications, and record at the target resolution or higher.
3. **Normalise all media** to the target format — usually `1080x1920` vertical or `1920x1080` landscape, a consistent frame rate (30 fps is a safe default), and `yuv420p`.
4. **Create reusable assets** — transparent text overlays, frames, masks, backgrounds, captions, logos.
5. **Compose scene-by-scene** with ffmpeg, one short clip per beat.
6. **Add motion** — pan, zoom, scroll, parallax, easing, transitions.
7. **Add audio** — voiceover, music, sound effects, captions, and loudness normalisation.
8. **Render preview frames / contact sheets** before the final export.
9. **Verify with `ffprobe`.**

Prefer several short scenes over one long continuous recording — easier to time, replace, review, and reuse.

**Normalise clip timing** with `setpts=(target/actual)*PTS`, e.g. retime an 8s clip to 6s:
```bash
ffmpeg -y -i clip-raw.mp4 -vf "setpts=0.75*PTS,fps=30,format=yuv420p" -an clip-normalised.mp4
```

# Useful libraries and tools

ffmpeg is excellent for encoding, compositing, filters, audio mixing, scaling, and final export. Other tools are better at generating assets, animation, captions, and visual design. Reach for them only when ffmpeg gets awkward.

### Visual composition

- **ffmpeg** — final composition, encoding, A/V sync, transitions, overlays, scaling, cropping, filters.
- **ImageMagick** — quick image conversion, resizing, trimming, masks, contact sheets.
- **Pillow / PIL** — generate transparent PNG text overlays, caption cards, thumbnails, title frames, gradients, simple layouts.
- **OpenCV** — frame analysis, crop detection, motion tracking, masking, scene processing.
- **PySceneDetect** — auto-detect scene changes in long videos.
- **Blender** — 3D product/device renders, camera movement, lighting, shadows.
- **Three.js / React Three Fiber** — web-based 3D scenes and animated product mockups.
- **Remotion** — React-based programmatic video; reusable branded templates and dynamic videos.
- **Manim** — animated diagrams and technical/educational explainers.
- **Lottie / Bodymovin** — lightweight vector animations from After Effects or JSON.

**Selection rule:**

- ffmpeg for the final render and media processing.
- Pillow / HTML+CSS for designed text overlays.
- Remotion when the layout is mostly UI, components, and text.
- Blender or Three.js when the video needs real 3D depth.
- OpenCV when it needs detection, tracking, cropping, or frame analysis.

### Text, captions, and typography

For premium text, prefer generating transparent PNG overlays over relying only on ffmpeg `drawtext` (which is fragile around quoting and font discovery).

- **Pillow** — reliable text rendering, wrapping, shadows, gradients, tracking, caption cards.
- **HTML/CSS screenshots via Playwright** — best when text needs web-quality typography: flexbox/grid, custom fonts, gradients, responsive layout.
- **ASS subtitles** — styled and karaoke-style captions.
- **SRT/VTT** — simple subtitles with broad platform support.
- **Whisper / faster-whisper** — transcribe voiceover into captions.
- **aeneas / Gentle / WhisperX** — forced alignment for accurate word timing.

Guidelines: keep headlines short; keep captions clear of important content; respect mobile safe margins; avoid flashing text unless it's a deliberate, readable rhythm; animate text once, then hold steady; generate preview frames to check overlap and readability. Prefer a condensed bold display font for headlines.

### Audio, voiceover, and music

Good audio reads as more professional than extra visual effects.

Voiceover:

- **Kokoro TTS / Piper** — lightweight local open-source TTS; good for repeatable, private generation.
- **Coqui XTTS** — expressive TTS and voice cloning; check licensing and model availability before commercial use.
- **OpenAI TTS / ElevenLabs / PlayHT** — higher-quality hosted voices when commercial polish matters; need API keys and cost control.

Music and sound: use open-licensed music when publishing commercially and keep the licence/attribution file beside the asset; prefer instrumental tracks under voiceover; duck music under speech; add subtle effects for transitions, taps, reveals, and success states; normalise final loudness.

```bash
# Normalise loudness
-af "loudnorm=I=-16:TP=-1.5:LRA=11"

# Fade music in and out
-af "afade=t=in:ss=0:d=1,afade=t=out:st=27:d=1"

# Mix voice and music
-filter_complex "[voice][music]amix=inputs=2:duration=first"

# Duck (sidechain) music under voice
-filter_complex "[music][voice]sidechaincompress=threshold=0.04:ratio=8:attack=20:release=250[ducked];[voice][ducked]amix=inputs=2"

# Loop music to fit the video without retiming the picture
ffmpeg -y -i silent.mp4 -stream_loop -1 -i music.mp3 -map 0:v:0 -map 1:a:0 \
  -shortest -c:v copy -c:a aac -b:a 192k -movflags +faststart final.mp4
```

Voiceover guidance: one short line per scene, matched to what's on screen; use punctuation to control rhythm; nudge TTS speed up slightly if it sounds flat; leave pauses between beats; generate captions from the **final** voiceover, not the draft script.

### Motion and animation

Static compositions feel like slideshows. Add subtle motion to every scene: slow push-in, gentle pan, device parallax, background drift, screen scroll, text ease-in then hold, fast-but-readable transitions on the beat, light zoom for emphasis. Prefer smooth easing over linear movement; animate once at a beat's start, then hold.

```bash
# Ease-out slide-in for a text overlay PNG (smooth, no flicker)
ffmpeg -y -i base.mp4 -i title.png \
  -filter_complex "[0:v][1:v]overlay=x='(W-w)/2':y='if(lt(t,0.35),-h+(220+h)*sin(t/0.35*PI/2),220)':enable='between(t,0,3)'[v]" \
  -map "[v]" -map 0:a? -c:v libx264 -crf 18 -preset medium -pix_fmt yuv420p \
  -c:a aac -b:a 192k -movflags +faststart out.mp4

# Ken Burns push-in
zoompan=z='min(zoom+0.0015,1.08)':d=150:x='iw/2-(iw/zoom/2)':y='ih/2-(ih/zoom/2)'
```

# Framing products, devices, and mockups

When footage shows an app, website, or product, frame it so it reads as a real object rather than a flat rectangle.

- Compose on the target canvas (e.g. `1080x1920`) and keep text in top/bottom bands, off the footage.
- Fit footage into the screen area, preserving aspect ratio:
  ```bash
  ffmpeg -y -i clip.mp4 -vf "scale=564:1284:force_original_aspect_ratio=decrease,pad=564:1284:(ow-iw)/2:(oh-ih)/2:black,fps=30,format=rgba" screen.mp4
  ```
- Make the device smaller than the canvas, with rounded body, bezel, notch/island, side buttons, and shadow; keep the content in a transparent screen cutout.
- Give the device a **stage** — warm rim light, soft shadow, gentle glow, faint outline. Never sit a black device on a black background.
- For consumer work, a cinematic dark background with warm white/gold type usually beats neon accent bars. Pick **one** accent colour and keep it.
- Accent key words within a line (white text, final keyword in the accent) instead of one flat colour.
- Write beat-specific support copy that explains the moment — never one generic line under every scene.
- Keep texture subtle; a little grain adds depth, visible noise cheapens it.

A solid vertical layout to start from:
```text
canvas: 1080x1920
media stage: x=40, y=235, w=1000, h=1420
device body: w=600, h=1298
device stage: dark rounded rectangle behind the device, low-opacity warm outline
headline: y=72, condensed uppercase, wide tracking
bottom tagline: y=1708, condensed uppercase, final keyword in accent
support copy: y=1814, smaller light text
accent: one warm colour, used consistently
```

# Video QA checklist

Before final delivery:

- `ffprobe` to confirm dimensions, duration, codecs, pixel format, and audio:
  ```bash
  ffprobe -v error -select_streams v:0 \
    -show_entries stream=width,height,r_frame_rate,duration \
    -show_entries format=duration,size -of default=nw=1 final.mp4
  ```
- Export preview frames at key timestamps and build a contact sheet for review:
  ```bash
  ffmpeg -y -ss 00:00:03 -i final.mp4 -frames:v 1 -update 1 preview-03.png
  ```
- Check text readability on mobile and that captions don't overlap important content.
- Check audio starts and ends cleanly and that voiceover matches what's on screen.
- Check for debug overlays, reload flashes, cursor artifacts, or stale screens.
- Confirm the export uses `yuv420p` and `+faststart`.

Recommended final export:
```bash
ffmpeg -i input.mp4 \
  -c:v libx264 -crf 18 -preset medium -pix_fmt yuv420p \
  -c:a aac -b:a 192k \
  -movflags +faststart \
  output.mp4
```

---

## Best practices

- **Inspect first:** `ffprobe -v quiet -print_format json -show_format -show_streams input.mp4`
- **`-c copy` whenever you can** to skip re-encoding.
- **Test on a slice** with `-t 10` before committing to a long render.
- **CRF guide:** 18 ≈ visually lossless, 23 = good default, 28 = smaller and acceptable (range 0–51).
- **`-movflags +faststart`** on web video for progressive playback.

## Guidelines for the agent

On a media request: identify the task, pick the matching recipe, confirm prerequisites (ffmpeg present, input exists), explain the command, run it with error handling, verify the output, and suggest optimisations where they help. For a finished video, work the pipeline above — beats first, then source, compose, motion, audio, QA — and break complex jobs into explained steps. Reach for a helper tool only when ffmpeg alone is awkward, and say why.

For concatenation, prefer process substitution with `printf` over a temporary `list.txt`; fall back to the file only when the shell can't do it.

Always: confirm the input exists and is readable, check ffmpeg is installed, ensure the output path is writable, fail with a clear message, and show progress on long renders.

## References

- Docs: https://ffmpeg.org/documentation.html
- Wiki: https://trac.ffmpeg.org/wiki
- Codecs: https://ffmpeg.org/ffmpeg-codecs.html
- Filters: https://ffmpeg.org/ffmpeg-filters.html

---
> Source: [muhammaddadu/ffmpeg-skill](https://github.com/muhammaddadu/ffmpeg-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
