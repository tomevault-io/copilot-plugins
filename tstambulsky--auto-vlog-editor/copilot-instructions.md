## auto-vlog-editor

> Operational rules and non-obvious gotchas for anyone (human or AI agent) editing

# AGENTS.md — working in this codebase

Operational rules and non-obvious gotchas for anyone (human or AI agent) editing
this repo. Architecture and feature explanations live in [`docs/`](docs/index.md).

> **Rule of thumb:** info needed to NOT break things → here. Info to understand
> how it works → `docs/`.

## Knowledge map

| Topic | Doc |
|---|---|
| Catalog / index | [docs/index.md](docs/index.md) |
| Pipeline & beat-sync architecture | [docs/architecture/pipeline.md](docs/architecture/pipeline.md) |
| Rendering (ffmpeg, HDR, audio, subtitles) | [docs/architecture/rendering.md](docs/architecture/rendering.md) |
| Publishing API (Zernio) | [docs/integrations/zernio.md](docs/integrations/zernio.md) |
| Voiceover (ElevenLabs) | [docs/integrations/elevenlabs.md](docs/integrations/elevenlabs.md) |
| Telegram bot UX & approval flow | [docs/integrations/telegram.md](docs/integrations/telegram.md) |
| Deployment | [docs/operations/deploy.md](docs/operations/deploy.md) |
| Web dashboard (read-only) | [docs/operations/dashboard.md](docs/operations/dashboard.md) |
| Design decisions | [docs/product/decisions.md](docs/product/decisions.md) |

## Hard rules

- **Never publish without approval.** Only a day in the `approved` state can create
  posts; the single gate is `publishing.publish_day()`. Don't add publish paths
  anywhere else.
- **Never call the Claude CLI outside `app/claude_cli.py`.** It's the swappable
  reasoning layer (`CLAUDE_CODE_COMMAND`): serialized, schema-validated, read-only
  tools. Everything that needs the model goes through `run_structured`.
- **Secrets never go into prompts or git.** Real keys live only in `.env` (which is
  gitignored). `.env.example` documents the variables with placeholders.

## Critical gotchas

### Subtitles/beats: ASR gives TIMING, the script gives TEXT
Word timings (`words_v{N}_{lang}.json`, which drive BOTH subtitles and beats) come
from `app/services/align.py`: faster-whisper transcribes the mp3, wav2vec2 refines
per-word timing. The transcription's TEXT is often mis-heard, so `retext_to_script`
re-maps the timed words onto the KNOWN script (difflib) and emits the script tokens
verbatim — **the subtitle word TEXT is ALWAYS the script, never the ASR.** This runs
on every path, including fallbacks (WhisperX align → free-form faster-whisper →
char alignment). Don't remove the re-text step or let any path set word text from
ASR. torch is CPU-only and lives in the Dockerfile `prod` stage (the `dev` image has
none → exercises the fallback). See [docs/architecture/rendering.md](docs/architecture/rendering.md).

### Beat-sync is a contract — don't flatten the script
Revisions and the humanizer pass rewrite `edl[].narration` **in place** (same count
and order). Never collapse the per-beat script into one block, or beat-sync breaks.

### Voice speed changes the script math
`VOICE_SPEED` scales words/second in the drafting prompt. If you change it, video
lengths shift with it.

### Video length knobs
`SCRIPT_WORDS_MIN`/`SCRIPT_WORDS_MAX` feed the drafting prompt; the render HARD-CAPS
at `MAX_VIDEO_SECONDS` and raises if the final video exceeds it. Keep
`SCRIPT_WORDS_MAX ÷ (≈2.6 · VOICE_SPEED)` comfortably under `MAX_VIDEO_SECONDS`.

### Telegram limits: 50MB out / 20MB in
Bots can **send** ≤50MB video; larger renders get an automatic CRF-28 preview
re-encode (don't raise the final CRF without checking sizes). Bots can only
**download** ≤20MB, so large 4K/HDR clips can't be uploaded in-chat — those go via
Drive. Sending "as video" makes Telegram recompress (kills HDR); send "as a file"
for full quality.

### iPhone / HDR footage
HDR (Dolby Vision/HLG) clips are tone-mapped (`zscale`+`tonemap=hable`) — needs
ffmpeg ≥6 with libzimg (the image ships ffmpeg 7.x). Clips with no audio stream skip
whisper (it crashes on them — handled in `stage_analyze`).

### Photos OFF by default (`INCLUDE_PHOTOS=false`)
Photos in the Drive folder are ignored — the video is clips-only. Set
`INCLUDE_PHOTOS=true` to re-enable Ken Burns stills (HEIC→JPEG via `pillow-heif`).

### Extras that apply at publish/preview time, NOT in the Draft/EDL
- **Shoutouts** (`/shoutout`, `app/shoutout.py`): per-day X handles in
  `<day>/shoutouts.json`; a deterministic line is appended to the **X** caption only.
- **Sign-off footer** (`app/signature.py`): a fixed footer appended to every caption,
  configured via `POST_SIGNATURE_*` (empty = none).
- **First-frame hook** (`FIRST_FRAME_HOOK`, `app/hook.py`): a 1–2s cold-open the user
  picks in the script-review gate; stored in `<day>/hook.json` and copy-concatted in
  front at render time. It is NOT a beat (invisible to beat-sync). Keep the selection
  out of the Draft/EDL.

### Pipeline toggles (all default on)
`BURN_SUBTITLES`, `AUTO_PIPELINE` (off = nightly check-in instead of auto-build),
`FIRST_FRAME_HOOK`, `SCRIPT_REVIEW` (gate the script before spending a render),
`HUMANIZE` (strip AI-writing patterns from fresh narration + captions, EN only),
`BILINGUAL` (also produce a second-language voiceover + video + captions).

### Script-review gate (`SCRIPT_REVIEW`, default true)
`run_pipeline` STOPS at the `script_review` state and returns `(draft, version, None)`
— **callers MUST handle `result is None`** (send the script, don't render).
Voiceover + render run only via `pipeline.render_phase(d)` on "✅ Approve & build".

## Commands

```bash
# tests (Docker-only, no local venv needed)
docker build --target dev -t auto-vlog-dev . && docker run --rm auto-vlog-dev

# render smoke test with synthetic clips (incl. HDR + freeze cases)
docker run --rm -v "$PWD":/srv/app -e DATA_DIR=/tmp/vj -e PYTHONPATH=/srv/app \
  auto-vlog-dev python scripts/smoke_render.py

# full day run / re-voice / re-render / voice A-B
docker compose run --rm --no-deps -e PYTHONPATH=/srv/app auto-vlog python scripts/run_day.py [YYYY-MM-DD]
docker compose run --rm --no-deps -e PYTHONPATH=/srv/app auto-vlog python scripts/regen_vo.py [date]
docker compose run --rm --no-deps -e PYTHONPATH=/srv/app auto-vlog python scripts/rerender.py [date]
docker compose run --rm --no-deps -e PYTHONPATH=/srv/app auto-vlog python scripts/voice_test.py [date]
```

## Knowledge persistence

Short rules/gotchas/commands → this file. Explanations, flows, and API details →
[`docs/`](docs/index.md) (update `docs/index.md` and the knowledge map above when you
add a page).

---
> Source: [tstambulsky/auto-vlog-editor](https://github.com/tstambulsky/auto-vlog-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
