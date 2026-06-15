## claude-watch

> Watch a tutorial or lecture video (URL or local path) and produce structured study notes. Downloads with yt-dlp, detects scene changes with ffmpeg, pulls a timestamped transcript (captions or Whisper API fallback), and writes a section-by-section markdown notes file with embedded screenshots to ~/claude-watch/library/<slug>/.


# /claude-watch — Claude turns a video into study notes

You don't have a video input. This skill gives you one *and* turns each viewing into a saved notes artifact.

## Step 0 — Setup preflight (silent on success)

Run on every `/claude-watch` invocation:

```bash
python3 "${CLAUDE_SKILL_DIR}/scripts/setup.py" --check
```

Exit codes: `0` ready (silent — proceed), `2` missing binaries, `3` missing API key, `4` both. On non-zero, run the installer:

```bash
python3 "${CLAUDE_SKILL_DIR}/scripts/setup.py"
```

On macOS this auto-`brew install`s ffmpeg + yt-dlp. On Linux/Windows it prints the right commands. It scaffolds `~/.config/claude-watch/.env` (mode 0600) with commented placeholders.

If a Whisper key is still missing afterwards, use `AskUserQuestion` to ask whether the user has a Groq key (preferred — cheaper, faster) or an OpenAI key, and write it to `~/.config/claude-watch/.env`. If they don't want to, run with `--no-whisper`; videos without native captions will come back frames-only.

## When to use

- User pastes a tutorial / lecture / talk URL and asks to study it
- User points at a local screen recording or video and wants notes
- User types `/claude-watch <url-or-path> [topic]`

## How to invoke

**Step 1 — parse input.** Separate the source (URL or path) from any topic the user mentioned. The topic shapes which sections you emphasize in the notes — pass it through to your synthesis, not to the script.

**Step 2 — run the watch script.**

```bash
python3 "${CLAUDE_SKILL_DIR}/scripts/watch.py" "<source>"
```

Optional flags:
- `--start T` / `--end T` — focus on a section (`SS`, `MM:SS`, or `HH:MM:SS`)
- `--max-frames N` — lower budget (default 80)
- `--resolution W` — bump frame width to 1024 px when on-screen text is tiny
- `--scene-threshold X` — sensitivity (default 0.30; raise for fewer cuts, lower for more)
- `--max-gap S` — coverage floor in seconds (default 45)
- `--whisper groq|openai` — force backend
- `--no-whisper` — disable Whisper entirely
- `--out-dir DIR` — override library root

**Step 3 — read every frame.** The script ends with a structured `=== frames ===` block listing each frame's path and timestamp. `Read` them all in parallel — they render as images in your context.

**Step 4 — load the transcript.** The `=== transcript ===` block points to `transcript.json` (or `transcript.window.json` for focused mode). `Read` it — it's a list of `{t_start, t_end, text, speaker_break}`.

**Step 5 — write `notes.md` to the library directory.** Use the **strict template** below. Save to `<library_dir>/notes.md`. Then print a 3-line summary to chat:
1. Title and slug
2. Number of sections + key concepts
3. Path to the notes file

Do **not** delete the library dir. It is the artifact.

## Notes template (non-negotiable structure)

````markdown
# <Video Title>

**Source:** <URL or path>  ·  **Duration:** MM:SS  ·  **Watched:** YYYY-MM-DD

## TLDR
<3-4 sentences: what the video is about and the single most important takeaway.>

## Key Concepts
- **<concept>** — <one-line definition> · `[t=MM:SS]`
- ...

## Notes

### [t=00:04] <Section title you derive from on-screen + spoken content>

![](frames/0001_t00-04.jpg)

**On screen:** <Transcribe / describe the slide, code, diagram. If code, transcribe verbatim.>

**Said:** <Relevant transcript excerpt for this scene, lightly cleaned.>

**Synthesis:** <Your connection — what this section is teaching, how it links to prior section.>

### [t=00:31] <next section>
...

## Code & Commands
<every code-on-screen frame's content as a runnable fenced block, language-tagged, with [t=MM:SS] back-link>

```python
# [t=03:45]
def forward(x):
    return x @ W + b
```

## Diagrams Referenced
- `[t=02:10]` — <one-line description of the diagram in frame 0008>
- ...

## Open Questions
- <things mentioned but not fully covered, or follow-ups to explore>
````

## Rules baked into the template

- **One scene = one section.** Use the `t=MM:SS` from each frame as the section anchor.
- **Adjacent scenes that are clearly the same topic** can be merged. When you do, mention it parenthetically: *(merged scenes at t=02:10 and t=02:42)*
- **Code blocks must be fenced** with the right language tag, transcribed verbatim from the frame.
- **The "On screen" block is required even for title slides.** Keeps the structure parallel.
- **Timestamps are absolute** (real video timeline) — for YouTube sources, a viewer can paste `<URL>&t=<seconds>` to jump there.

## Re-runs

If the user re-watches the same URL, the script reuses the cached download, transcript, and scenes. Only frames + notes regenerate. To force a full re-run, delete `<library_dir>/meta.json` first.

## Failure modes

- **Setup preflight non-zero** → run `setup.py`, then ask for a key via `AskUserQuestion`.
- **No transcript** → script emits `transcript_source: none`. Generate notes frames-only and tell the user.
- **Long video sparse-scan warning** → offer to re-run with `--start`/`--end` focused on the part the user cares about.
- **Whisper failure** → retry with `--whisper openai` (if Groq failed) or vice versa.

## Token budget

Frames dominate cost (~50-80k input tokens for 60 frames at 512 px). Transcripts are cheap. `--resolution 1024` quadruples per-frame cost — only when the user must read tiny on-screen text.

If the user asks a follow-up about a video you already watched in this session, do NOT re-run the script. The library directory is on disk; re-`Read` only the frames you need.

## Security

- Runs `yt-dlp`, `ffmpeg`, `ffprobe` locally
- Sends extracted mono 16 kHz audio to Groq (preferred) or OpenAI Whisper API only when captions are missing
- Reads/writes `~/.config/claude-watch/.env` (mode 0600) for keys
- Persists artifacts to `~/claude-watch/library/<slug>/` — review the directory after first run if you're cautious
- Does NOT log or transmit API keys, video files, or the original URL outside the audio-to-Whisper call

---
> Source: [devinilabs/claude-watch](https://github.com/devinilabs/claude-watch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
