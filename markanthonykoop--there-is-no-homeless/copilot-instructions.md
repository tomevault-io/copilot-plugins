## there-is-no-homeless

> Norms for whoever (AI or human) modifies this project next. Read `README.md`

# CLAUDE.md — there_is_no_homeless

Norms for whoever (AI or human) modifies this project next. Read `README.md`
first — this file only adds what a future maintainer would otherwise have to
discover the hard way.

Universal rules: `~/CLAUDE.md`. Machine-wide notes: `~/claude/CLAUDE.md`.

## Identity

Same as every other release on this machine: **Mark Nadon · MiddleMatter Music ·
markanthonykoop@gmail.com**. Don't re-hardcode these — import the constants from
`youtube_publisher.metadata` if you need them.

## Cached credentials — do NOT trigger re-auth

- `~/.cache/youtube_publisher/token_677495352.pickle` — YouTube + Drive scopes
  already granted (`youtube.upload`, `drive.file`). Use
  `youtube_publisher.auth.get_credentials(scopes=…)` and it'll silent-refresh.

## The asset directory is a symlink

`./assets/` → `/mnt/d/there_is_no_homeless/`. Anything heavy (raw phone pulls,
intermediate renders, finals) goes there. **Never** commit a media file under
the project root — it'll sit on `C:` and the user has explicitly asked us not
to do that.

## Phone pull — current MTP gotcha

The user's phone is a **Motorola Moto G 2026** (USB VID_22B8 — Motorola).
It only exposes its filesystem when it's in **MTP / File-transfer** mode.
By default Android plugs in as charging-only; swipe down the USB notification
and select "File transfer" to enable MTP.

**Shell.Application does NOT work for MTP enumeration on this phone** — it
returns 0 items even when the device is visible in Explorer. Don't waste time
on that path.

**ADB is the reliable pull method.** USB debugging path on Moto G 2026 is NOT
via the standard "Build number" tap — the exact path is TBD (user confirmed
no Build number under About phone; check Software version or System submenu).
Once USB debugging is on, approve the dialog on the phone, then:
```
ADB="/mnt/c/Program Files (x86)/Android/android-sdk/platform-tools/adb.exe"
"$ADB" pull /sdcard/DCIM/Camera /mnt/d/downloads/there_is_no_homeless
```

## Episode scripts

Each episode is one `.md` in `episodes/` containing: outline, transcripts,
credit list, edit decisions, and the final YouTube description. The
`video_composer` package consumes a `## Timeline` section in the same file —
see `~/claude/video_composer/README.md` for the schema. Keep narrative text
*outside* that section; the parser only looks at `### seg-XX` blocks.

## Where the raw data actually lives

The phone pull has already happened. Don't ask the user to redo it.

- **Phone videos:** `/mnt/d/downloads/there_is_no_homeless/` —
  132 × `VID_YYYYMMDD_HHMMSS_*.mp4`, ~74 GB, 9.6 h total runtime. 46 JPGs
  in the same dir.
- **WhatsApp re-encodes:** `…/there_is_no_homeless/whatsapp/` — 5 mp4s
  dated 2026-05-10. These are mostly re-encodes of phone clips; use
  `transcriptions/tools/compare_videos/` to dedup by duration.
- **Sound recordings (Windows Voice Recorder):**
  `/mnt/c/Users/x/Documents/Sound recordings/` — 151 × `.m4a`, 7.4 GB.
  The path has a **space**; quote it. Files named `Recording (NNN).m4a`
  by default; user-renamed favorites include `ez.m4a` and
  `Recording (140).m4a`.

## Two source files you should know exist

- `…/Sound recordings/Recording (140).m4a` — Mark's **full episode-1
  narration**, 3:32. Opens *"Hey y'all. I'm Mark. I'll be your humble
  narrator."* Introduces the cast (Obiwana, Hunter, T.K., John).
  Transcribed → `transcriptions/audio/Recording (140).md`.
- `…/Sound recordings/ez.m4a` — the "ez pz" marker. Whisper transcribes
  it as **"easy, easy, easy, easy. Easy. It's so easy peasy."** When
  searching for the marker, grep `easy` not `ez pz` — that's the literal
  audio.

## Transcription store: separate documented subtree

`/mnt/d/downloads/there_is_no_homeless/transcriptions/` has its own
`README.md` + `CLAUDE.md`. **Read those before touching transcripts.** It
documents:
- The manifest / database / per-file artifact layout.
- The three tools (`transcribe_recent`, `build_database`, `compare_videos`).
- The "rebuilt from scratch" rule for `database.md`.
- The DrvFs cosmetic permission warnings.

## Tooling reaches

- **`video_composer`** (sibling) — turns the `## Timeline` block into MP4.
  Per-segment cache on `/mnt/d/cache/video_composer/`. Needs an ffmpeg build
  with `drawtext` (we use BtbN master, symlinked into `~/.local/bin/`).
- **`runway_client`** (sibling) — fills `[GAP]` segments via Runway's
  `text_to_image` → `image_to_video` pipeline. Reads
  `video_composer gaps --json` output. Cache key required at
  `~/.cache/runway_client/api_key`.
- **`speech_transcriber`** (sibling) — faster-whisper on the user's recordings.
  CUDA on this machine; default `distil-large-v3` (English). Use
  `speech_transcriber find` to locate marker phrases like "ez pz" without
  listening through the whole recording.
- **`youtube_publisher`** (sibling) — final upload step; OAuth already cached.

## Smoke test before declaring an episode done

1. The final MP4 plays in `mpv` start-to-finish without error.
2. `ffprobe` reports the right duration (intro + footage + outro).
3. Description draft is in `episodes/<id>.md` under `## Description`.
4. Thumbnail exists at `assets/episodes/<id>/final/thumb.jpg`.
5. Show the user the description before any upload — never auto-publish.

## Don't grow this project sideways

If you're tempted to add a generic "audio normalizer" or "subtitle burner"
helper, stop. Those go in a sibling (`audio_fx/`, `video_composer/`). This
project is *just* the per-episode glue.

---
> Source: [MarkAnthonyKoop/there_is_no_homeless](https://github.com/MarkAnthonyKoop/there_is_no_homeless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
