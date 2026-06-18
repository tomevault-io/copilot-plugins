## ec-watch

> Watch a video (URL or local path). Downloads with yt-dlp, extracts auto-scaled frames with ffmpeg, pulls the transcript from captions (or local faster-whisper fallback), and hands the result to Claude so it can answer questions about what's in the video.


# /ec-watch — Claude watches a video

You don't have a video input; this skill gives you one. A Python script downloads the video, extracts frames as JPEGs, gets a timestamped transcript (native captions first, then local faster-whisper fallback), and prints frame paths. You then analyze each frame path using the MiniMax understanding_image MCP tool to see the images and combine them with the transcript to answer the user.

## Step 0 — First-run setup (runs once only)

The very first time `/ec-watch` is invoked, setup runs automatically and sets up faster-whisper, verifies binaries, and marks setup complete. After that, Step 0 is skipped entirely — subsequent invocations go straight to Step 1.

**To re-run setup manually (only if needed):**

```bash
python "C:/Users/Emir/.claude/skills/ec-watch/scripts/setup.py"
```

## When to use

- User pastes a video URL (YouTube, Vimeo, X, TikTok, Twitch clip, most yt-dlp-supported sites) and asks about it.
- User points at a local video file (`.mp4`, `.mov`, `.mkv`, `.webm`, etc.) and asks about it.
- User types `/ec-watch <url-or-path> [question]`.

## Recommended limits

- **Best accuracy: videos under 10 minutes.** Frame coverage scales inversely with duration.
- **Hard caps: 100 frames total and 2 fps.** Token cost grows with frame count:
  - ≤30s → ~1-2 fps (up to 30 frames)
  - 30s-1min → ~40 frames
  - 1-3min → ~60 frames
  - 3-10min → ~80 frames
  - \>10min → 100 frames, sparsely spaced (warning printed)
- For long videos, ask if they want to focus on a specific section.

## How to invoke

**Step 1 — parse the user input.** Separate the video source (URL or path) from any question the user asked. Example: `/ec-watch https://youtu.be/abc what language is this in?` → source = `https://youtu.be/abc`, question = `what language is this in?`.

**Step 2 — run the watch script.** Use `python` on Windows:

```bash
python "C:/Users/Emir/.claude/skills/ec-watch/scripts/watch.py" "<source>"
```

Optional flags:
- `--start T` / `--end T` — focus on a section (SS, MM:SS, or HH:MM:SS)
- `--max-frames N` — lower the cap (default 80, max 100)
- `--resolution W` — frame width in px (default 512)
- `--fps F` — override auto-fps (clamped to 2 fps max)
- `--out-dir DIR` — working directory (default: tmp)
- `--whisper local` — force local faster-whisper (default, no API key needed)
- `--no-whisper` — disable Whisper fallback (frames-only)

Examples:
```bash
# Last 10 seconds
python "C:/Users/Emir/.claude/skills/ec-watch/scripts/watch.py" video.mp4 --start 50 --end 60

# Zoom into 2:15 → 2:45
python "C:/Users/Emir/.claude/skills/ec-watch/scripts/watch.py" "$URL" --start 2:15 --end 2:45

# From 1h12m to end
python "C:/Users/Emir/.claude/skills/ec-watch/scripts/watch.py" "$URL" --start 1:12:00
```

**Step 3 — Analyze frames with MiniMax MCP.**

After frames are extracted, use the MiniMax understanding_image MCP tool to analyze each frame. The tool name is `mcp__minimax__understanding_image` and it accepts:
- `prompt`: "Describe what's on screen, any UI elements, text visible, and the overall context."
- `image_url`: The file path to the JPEG frame

Process frames in batches of 5-10 for efficiency. Combine the visual analysis with the transcript to answer the user.

**Step 4 — answer the user.** You now have:
- **Frames** — what's on screen at each timestamp
- **Transcript** — what's said at each timestamp (via `whisper (local)` or `captions`)

If the user asked a specific question, answer citing timestamps. If they didn't ask, summarize the video — key moments, visuals, spoken content.

**Step 5 — clean up.** Delete the working directory if user won't ask follow-ups:
```bash
rm -rf "C:/Users/Emir/AppData/Local/Temp/watch-xxx"
```

## Transcription

The script gets a timestamped transcript in one of two ways:

1. **Native captions (free, preferred).** yt-dlp pulls subtitles from the source.
2. **Local faster-whisper fallback (free, no API key).** Extracts audio and transcribes locally using faster-whisper on CPU.

No API key needed for local transcription.

## Failure modes

- **Setup preflight failed** → run `python "C:/Users/Emir/.claude/skills/ec-watch/scripts/setup.py"`
- **No transcript** → captions missing AND Whisper unavailable. Proceed frames-only.
- **Long video warning** → offer to re-run focused on a section with `--start`/`--end`
- **Download fails** → tell the user plainly; do not retry

## Token efficiency

- 80 frames at 512px ≈ 50-80k image tokens
- Transcript is cheap (a few thousand tokens max)
- Only bump resolution to 1024 when user needs to read on-screen text

If user asks a follow-up, do **not** re-run the script — you already have frames and transcript in context.

## Security

- Video downloaded locally via yt-dlp
- Audio transcribed locally via faster-whisper (no data leaves your machine)
- No API keys stored or transmitted
- Working directory auto-deleted when done

---
> Source: [emir-can-tr/ec-watch](https://github.com/emir-can-tr/ec-watch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
