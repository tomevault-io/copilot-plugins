## autocliper-ai

> |


# AutoCliper.AI — Agent Skill

Convert one long YouTube video into 5–15 short clips with Chinese packaging and hard-burned subtitles.

This document is the **single source of truth** for any AI agent executing the AutoCliper pipeline. Follow every numbered step in order. Do not skip steps. Do not guess file names — inspect the filesystem after each write operation.

---

## 0. Pre-flight Checks

Before doing anything else, verify the environment. Report the **exact blocker** and stop if any check fails.

| Check | Command | Pass Condition |
|-------|---------|----------------|
| yt-dlp installed | `which yt-dlp` | returns a path |
| ffmpeg installed | `which ffmpeg` | returns a path; if missing, try `python3 -c "import imageio_ffmpeg; print(imageio_ffmpeg.get_ffmpeg_exe())"` |
| Python 3.9+ | `python3 --version` | >= 3.9 |
| Skill repo available | check that `corekit/` directory exists relative to the skill root | `corekit/__init__.py` is present |

Set the environment for all subsequent commands:

```bash
export PYTHONPATH="<path-to-AutoCliper.AI-repo>"
```

All `python3 -m corekit.*` commands below assume this `PYTHONPATH` is set.

---

## 1. Create Workspace

Create the output directory structure for this video. Derive `<slug>` from the video title or ID — lowercase, hyphens, no spaces, no special characters.

```bash
mkdir -p "studio/<slug>/intake"
mkdir -p "studio/<slug>/intel"
mkdir -p "studio/<slug>/exports"
```

Expected result:

```
studio/<slug>/
├── intake/       ← raw video + subtitles land here
├── intel/        ← analysis artifacts land here
└── exports/      ← per-clip folders land here
```

---

## 2. Fetch Source Assets

Download the video and subtitles from YouTube:

```bash
python3 -m corekit.fetch_source "<YouTube_URL>" "studio/<slug>/intake"
```

**What happens inside**: the downloader tries English subtitles first (`en`, `en-US`, `en-orig`), falls back to Chinese (`zh-Hans`, `zh-CN`, `zh`). It uses `--cookies-from-browser chrome` for authenticated access.

**After the command finishes**, list the intake folder and identify:

1. The `.mp4` video file (may contain the video title and ID in the filename)
2. The `.srt` subtitle file (may have a language tag like `.en.srt` or `.zh-Hans.srt`)
3. Any sidecar files (`.ytdl`, `.jpg`, etc.) — note them but they are not needed

**If the download fails**:
- Cookie error → tell the user to refresh their Chrome YouTube login, then retry
- No subtitles found → report "no subtitles available for this video" and stop
- Video download error → retry once with a different format; if still failing, report the error

**Record** the exact filenames for the next steps. Do not guess or hardcode names.

---

## 3. Parse Subtitles into JSON

Convert the SRT subtitle file into structured JSON for easier analysis:

```bash
python3 -m corekit.subtitle_to_json \
  "studio/<slug>/intake/<exact-srt-filename>" \
  "studio/<slug>/intel/transcript.json"
```

**Verify** the output file exists and contains an array of cue objects. Each cue has:

```json
{
  "index": 1,
  "start": "00:00:01,234",
  "end": "00:00:03,456",
  "start_seconds": 1.234,
  "end_seconds": 3.456,
  "text": "the spoken content"
}
```

If the transcript is empty or has fewer than 10 cues, report "transcript too short or corrupted" and stop.

---

## 4. Analyze Transcript and Propose Candidates

This is the most important intellectual step. Read the following files **before** starting analysis:

1. **[playbooks/clip-contract.md](playbooks/clip-contract.md)** — defines the exact JSON schema for `selected_clips.json`
2. **[playbooks/content-analysis-playbook.md](playbooks/content-analysis-playbook.md)** — defines the multi-pass analysis method, scoring rubric, and review formatting

### 4a. Multi-Pass Analysis

Do NOT try to pick clips in a single pass. Follow this sequence:

**Pass 1 — Skim and flag**: Read through `transcript.json` looking for stretches that contain memorable, information-dense, opinionated, counterintuitive, or emotionally sharp content. Flag generously — it is better to flag too many than too few.

**Pass 2 — Boundary refinement**: For each flagged stretch, re-read the surrounding cues (5–10 cues before and after). Choose a clean start and end:
- The **start** must land on a line that hooks the viewer within the first 3 seconds of the clip
- The **end** must land *after* the speaker has finished the thought — never mid-sentence
- If a promising moment needs a few seconds of context, extend the start earlier
- If a thought trails off, extend the end until it resolves

**Pass 3 — Completeness check**: For each candidate, verify:
- [ ] It is 20–180 seconds long (can stretch to 3 min if payoff justifies it)
- [ ] It contains exactly one clear idea
- [ ] The opening line is a hook, not filler
- [ ] The closing line resolves the thought
- [ ] It is understandable without watching the rest of the video
- [ ] It is NOT a greeting, sponsor read, Q&A preamble, or long digression

**Pass 4 — Score and rank**: Score each candidate on four dimensions (1–5):
- `hook` — how compelling is the opening in the first 3 seconds?
- `clarity` — is it one clean idea with low ambiguity?
- `standalone` — can it stand alone without prior context?
- `payoff` — is the ending useful, memorable, or share-worthy?

Rank by weighted total: `hook × 0.35 + clarity × 0.25 + standalone × 0.2 + payoff × 0.2`

### 4b. Candidate Count Guidance

| Source Duration | Target Candidates |
|-----------------|-------------------|
| < 20 min | 5 – 8 |
| 20 – 60 min | 8 – 12 |
| > 60 min | 10 – 15 |

**The default failure mode should be "too few candidates," not "too many."** When in doubt, include more. The user will prune.

### 4c. Handling Auto-Generated (Noisy) Subtitles

YouTube auto-captions often contain:
- Repeated fragments (same phrase appearing in consecutive cues)
- Missing punctuation and capitalization
- Misheard words (especially names, technical terms)

When the source subtitles are auto-generated:
- Infer the intended meaning conservatively — do not invent claims
- Ignore duplicate fragments when judging clip boundaries
- Use surrounding context to determine where sentences actually end
- Note in the candidate summary if the source quality is low

### 4d. Write Outputs

Write two files:

1. **`studio/<slug>/intel/selected_clips.json`** — the machine-readable clip decisions. Must follow the schema in `playbooks/clip-contract.md` exactly.

2. **`studio/<slug>/intel/candidate-board.md`** — the human-readable review table.

---

## 5. Present Candidates for User Review

Show the candidate list in a **review-friendly format**. For each candidate, display:

| Field | Format | Example |
|-------|--------|---------|
| ID | `clip-XX` | `clip-01` |
| Time range | `HH:MM:SS → HH:MM:SS` | `00:12:03 → 00:13:22` |
| Duration | human-readable | `1m 19s` |
| Title | one provocative Chinese title, ≤12 characters | `AI终将取代一切？` |
| Summary | exactly two sentences in Chinese | 句1: 在讲什么。句2: 为什么值得看。 |

Present as a numbered table so the user can reply with IDs.

**If the user says "proceed" or "auto-pick"**: select the top-ranked candidates yourself. For a 1-hour video, export at least 8.

**If the user picks specific IDs**: export only those.

---

## 6. Export Each Chosen Clip

For **each** clip the user chose (or you auto-picked), execute steps 6a through 6f **in order**. Complete all sub-steps for one clip before moving to the next.

### 6a. Cut the Video Segment

```bash
python3 -m corekit.cut_video \
  "studio/<slug>/intake/<exact-mp4-filename>" \
  <start_seconds> <end_seconds> \
  "studio/<slug>/exports/<clip-folder>/clip.mp4"
```

The `<clip-folder>` naming convention is `XX-<title-slug>`, e.g., `01-ai-will-replace`.

**Verify** the output file exists and has a non-zero size.

### 6b. Window the Source Subtitle

```bash
python3 -m corekit.window_subtitles \
  "studio/<slug>/intake/<exact-srt-filename>" \
  <start_seconds> <end_seconds> \
  "studio/<slug>/exports/<clip-folder>/clip.src.srt"
```

This extracts only the cues that overlap the clip window and shifts all timestamps so the clip starts at `00:00:00,000`.

**Verify** the output SRT has at least one cue.

### 6c. Translate to Chinese Subtitle

Read `clip.src.srt` and translate it into simplified Chinese. Write the result to `clip.zh.srt` in the same folder.

**Translation rules** (follow strictly):

1. **Natural spoken Chinese** — not written/formal Chinese. The viewer is watching a short video, not reading a document.
2. **Preserve timestamps exactly** — unless you need to merge two cues for readability. Never arbitrarily shift timing.
3. **Keep named entities accurate** — names of people, companies, products, numbers, and quoted phrases must be faithfully retained.
4. **Concise lines** — prefer short Chinese phrases that fit short-form video pacing. Long scrolling subtitles kill retention.
5. **Rebalance across cues when needed** — if an English sentence is split across 3 cues, you may redistribute the Chinese text across those same 3 cues. Preserve cue order.
6. **Do NOT collapse cues** — a cue-dense English SRT with 40 cues should produce roughly 35–45 Chinese cues, not 10. The default failure mode is slightly too many cues, not far too few.
7. **Drop only true duplicates** — auto-captions often repeat the same fragment in consecutive cues. You may drop the duplicate, but keep the first occurrence.
8. **If source is already Chinese** — do not translate. Only clean obvious auto-caption noise (repeated fragments, broken characters).

### 6d. Generate Title and Description

For each clip, create:

**Title** (for the first-second overlay):
- Sharp, clickable, opinionated
- Target **≤ 12 Chinese characters** so it fits cleanly on screen at font size 48
- May be provocative or contrarian but must be faithful to the speaker's meaning
- Should create curiosity, tension, or an urge to keep watching

**Description** (for platform distribution):
- ≤ 140 Chinese characters
- Must mention: who is speaking, what show/interview this is from, what topic is discussed, what the key claim or takeaway is

Write both to `studio/<slug>/exports/<clip-folder>/metadata.txt`:

```
标题：AI终将取代一切？
描述：Sam Altman 在 Lex Fridman 播客中谈到 AGI 的时间表——他认为大多数人严重低估了 AI 的发展速度，未来三年将改变一切。
```

### 6e. Burn Chinese Subtitles + Title into Video

```bash
python3 -m corekit.render_hardsubs \
  "studio/<slug>/exports/<clip-folder>/clip.mp4" \
  "studio/<slug>/exports/<clip-folder>/clip.zh.srt" \
  "studio/<slug>/exports/<clip-folder>/clip.hardsub.mp4" \
  --title "<the-title-from-6d>"
```

**CRITICAL**: Pass the **Chinese** `clip.zh.srt` file, NOT `clip.src.srt`. This is the most common agent mistake — passing the source-language subtitle instead of the translation.

The burn step will:
- Auto-detect libass; if unavailable, fall back to drawtext rendering (no action needed from you)
- Auto-select the best H.264 encoder for the current machine
- Render the title centered at font size 48 with 3px black outline during the first second
- Render subtitle cues at the bottom of the frame for the rest of the clip

**Verify** the output `clip.hardsub.mp4` exists and has a larger file size than `clip.mp4` (the filter chain adds visual data).

### 6f. Append to Packaging Copy

After each clip is exported, append its title and description to the combined packaging file:

`studio/<slug>/intel/packaging-copy.md`

Format:

```markdown
## clip-01: AI终将取代一切？

**标题**: AI终将取代一切？
**描述**: Sam Altman 在 Lex Fridman 播客中谈到 AGI 的时间表...
**文件**: studio/<slug>/exports/01-ai-will-replace/clip.hardsub.mp4
**时长**: 1m 19s

---
```

---

## 7. Final Output

After all clips are exported, return to the user:

1. **Source asset folder**: `studio/<slug>/intake/`
2. **Candidate list**: the full table from step 5 (timestamps, duration, title, summary)
3. **Packaging file**: `studio/<slug>/intel/packaging-copy.md`
4. **Exported clips**: list each `clip.hardsub.mp4` path

---

## Workspace Layout Reference

```
studio/<video-slug>/
├── intake/                              # raw assets from YouTube
│   ├── <title> [<id>].mp4              # source video (filename from yt-dlp)
│   └── <title> [<id>].<lang>.srt       # source subtitle
├── intel/                               # analysis artifacts
│   ├── transcript.json                  # structured subtitle cues (from step 3)
│   ├── selected_clips.json              # clip decisions (from step 4)
│   ├── candidate-board.md               # review table (from step 4)
│   └── packaging-copy.md                # all titles + descriptions (from step 6f)
└── exports/                             # one folder per exported clip
    └── 01-<slug>/
        ├── clip.mp4                     # raw cut (no subtitles)
        ├── clip.src.srt                 # windowed source-language subtitle
        ├── clip.zh.srt                  # translated Chinese subtitle
        ├── clip.hardsub.mp4             # final deliverable (burned subtitles + title)
        └── metadata.txt                 # title + description for this clip
```

---

## Module Reference

| Module | Invocation | Arguments |
|--------|-----------|-----------|
| `corekit.fetch_source` | `python3 -m corekit.fetch_source <url> <output_dir>` | YouTube URL, output directory |
| `corekit.subtitle_to_json` | `python3 -m corekit.subtitle_to_json <input.srt> <output.json>` | SRT file path, JSON output path |
| `corekit.cut_video` | `python3 -m corekit.cut_video <input.mp4> <start_sec> <end_sec> <output.mp4>` | source video, start seconds (float), end seconds (float), output path |
| `corekit.window_subtitles` | `python3 -m corekit.window_subtitles <input.srt> <start_sec> <end_sec> <output.srt>` | source SRT, start seconds (float), end seconds (float), output path |
| `corekit.render_hardsubs` | `python3 -m corekit.render_hardsubs <input.mp4> <input.srt> <output.mp4> --title "..."` | clip video, Chinese SRT, output path, optional title text |

Optional flags for `render_hardsubs`:
- `--fontfile <path>` — override the auto-detected Chinese font
- `--subtitle-fontsize <int>` — subtitle font size for drawtext fallback (default 28)

---

## Execution Safety Notes

1. **Paths with spaces or non-ASCII**: Always pass file paths as separate arguments to shell commands. Never build a command string with `f"ffmpeg -i {path}"` — the downloader produces filenames like `My Interview [abc123].mp4`. Use list-based `subprocess.run(cmd)`.

2. **Cookie failures**: The downloader uses `--cookies-from-browser chrome`. If it fails with a 403 or login-required error, tell the user to open YouTube in Chrome and verify they are logged in, then retry. Do not modify the download command.

3. **Verify after every write**: After every command that produces a file, check that the file exists and has non-zero size before proceeding. If a step produces no output, report it immediately.

4. **Vertical shorts**: If the user asks for vertical (9:16) shorts, complete the entire horizontal pipeline first, then crop/reframe as a post-processing step. Do not attempt vertical crop during the main pipeline.

5. **Encoder detection is automatic**: Do not hardcode `libx264` or `aac` in any manual ffmpeg commands. The corekit modules handle encoder selection. If you need to run ffmpeg directly for any reason, use `corekit.ffmpeg_locator.h264_encoder()` and `corekit.ffmpeg_locator.aac_encoder()` to get the right encoder names.

---

## Error Reporting

If the pipeline cannot complete, report the **exact** blocker using one of these categories:

| Blocker | What to tell the user |
|---------|----------------------|
| `COOKIES_EXPIRED` | "YouTube download failed due to expired cookies. Please log in to YouTube in Chrome and retry." |
| `NO_SUBTITLES` | "No English or Chinese subtitles found for this video. The pipeline requires subtitles." |
| `FFMPEG_MISSING` | "ffmpeg is not installed. Install via `brew install ffmpeg` or `pip install imageio-ffmpeg`." |
| `DOWNLOAD_FAILED` | "Video download failed. [include the error message from yt-dlp]" |
| `TRANSCRIPT_EMPTY` | "The subtitle file is empty or contains too few cues to analyze." |
| `LOW_QUALITY_TRANSCRIPT` | "The auto-generated subtitles are too noisy to produce reliable clips. Consider finding a manually transcribed version." |

Do not attempt workarounds for blockers. Report and stop.

---
> Source: [ZhanlinCui/AutoCliper.AI](https://github.com/ZhanlinCui/AutoCliper.AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
