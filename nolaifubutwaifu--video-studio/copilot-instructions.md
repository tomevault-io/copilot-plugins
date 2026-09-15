## video-studio

> A pipeline for making social videos that match a learned style: analyze

# video-studio

A pipeline for making social videos that match a learned style: analyze
reference videos once, then for each new video generate a script, cut the raw
take, and render it with captions and graphics.

## The shape of it

```
references ──/style-learn──> styles/<name>/profile.json   ← single source of truth
topic ───────/script────────> projects/<slug>/script.md
raw take ────/edit──────────> projects/<slug>/edl.json → work/trimmed.mp4
             /render────────> projects/<slug>/out/final.mp4 + Remotion project
```

`profile.json` is the point of the whole system. Every downstream step reads it,
so a style change is a data edit rather than a code change, and video #20 looks
like video #1 without re-explaining anything.

## Environment constraints that shaped the design

- **ffmpeg here has no libass, drawtext or subtitles filter** (verified: 0 of
  489 filters). All text is drawn by Remotion. Never reach for
  `-vf subtitles=` or `drawtext` — they will fail.
- **Cuts drift.** The encoder snaps every cut to a frame boundary and runs
  28-82ms long per segment. It is invisible per segment and strictly
  accumulates — 187ms across four segments in testing. `pretrim.py` therefore
  builds its timeline from *measured* segment durations, not requested ones.
- **Whisper model choice matters.** `tiny.en` mishears disfluencies ("um" →
  "Bum"), so the filler pass silently misses them. Use `small.en`. First run of
  any model downloads it (~460MB for small.en); after that a 20s clip takes a
  few seconds.

## Tools

All stdlib-only Python wrappers over ffmpeg/whisper/yt-dlp. Each takes a file
and prints JSON; `--out` also writes it.

| Tool | Does |
|---|---|
| `ingest.py` | yt-dlp download or local copy; cropdetect for letterboxed screen recordings |
| `probe.py` | dimensions, fps, duration, aspect, audio presence |
| `transcribe.py` | word-level whisper transcript |
| `shots.py` | scdet cut detection → cuts/min, shot lengths |
| `audio.py` | silencedetect pause rhythm + loudnorm measurement |
| `frames.py` | labelled contact sheets (Pillow, since ffmpeg cannot draw text) |
| `edl.py` | retake / filler / dead-air detection → edit decision list |
| `pretrim.py` | applies the EDL, remaps the transcript onto the new timeline |
| `render.py` | stages media, writes Remotion props, renders, normalises loudness |

## Rules

- **Always pass the remapped `work/words.json` to the renderer**, never the
  original transcript. The original's timestamps refer to the uncut take, so
  every caption lands late by the amount cut before it.
- **Get approval on the EDL before rendering.** A wrong cut destroys a take; a
  missed one costs a second of runtime.
- **This repo is public.** Media, takes and downloaded references are
  gitignored and must stay that way. Downloaded references are third-party
  copyrighted material: analysis input only, never committed. Contact sheets
  are stills from those videos and are gitignored too — only the derived
  `profile.json` and `style-notes.md` are committed.
- **Platform UI is not style.** Screen-recorded references contain TikTok's or
  Instagram's own interface. Never let it into a profile; see the
  `video-style-analyst` agent for the full separation rules.

## Verifying a change

There is a synthetic take for smoke-testing — speech containing a deliberate
retake, two fillers and a 2s pause:

```bash
say -v Karen -o speech.aiff "Here are three things nobody tells you about cold email. Um, number one. Number one, your subject line is doing ninety percent of the work. [[slnc 2000]] Number two, uh, you should never, like, pitch in the first message. Number three, follow up twice. Follow me for more."
ffmpeg -y -f lavfi -i "testsrc2=size=1080x1920:rate=30" -i speech.aiff -shortest \
  -c:v libx264 -crf 28 -pix_fmt yuv420p -c:a aac take.mp4
```

A correct run removes exactly those four things and nothing else. To check
caption sync, sample a frame at a word's midpoint from `words.json` and confirm
that word is the highlighted one.

---
> Source: [Nolaifubutwaifu/video-studio](https://github.com/Nolaifubutwaifu/video-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
