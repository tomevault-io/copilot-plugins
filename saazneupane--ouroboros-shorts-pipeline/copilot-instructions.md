## ouroboros-shorts-pipeline

> **Ouroboros** — AI-native vertical video engine that researches a topic, writes a script, sources b-roll, generates voiceover + captions, mixes music, assembles a 9:16 Short, and uploads to YouTube (and optionally Facebook).

# CLAUDE.md — Ouroboros

**Ouroboros** — AI-native vertical video engine that researches a topic, writes a script, sources b-roll, generates voiceover + captions, mixes music, assembles a 9:16 Short, and uploads to YouTube (and optionally Facebook).

---

## Tech Stack & Key Dependencies

| Layer | Tech |
|---|---|
| Language | Python 3.10+ |
| LLM | Anthropic Claude (default `claude-sonnet-5`), Gemini, OpenAI, Ollama |
| TTS | Edge TTS (free default, 300+ voices), ElevenLabs (premium) |
| Captions | OpenAI Whisper (`base` model) → ASS burn-in + SRT for upload |
| Video | FFmpeg + ffprobe (must be on PATH) |
| B-roll / thumbnails | Pexels API (free tier: 200/hr, 20k/mo) |
| Upload | YouTube Data API v3 + YouTube Analytics API v2 (OAuth2) |
| Social | Facebook Graph API (optional, non-fatal if unconfigured) |
| UI | Gradio (`pip install ouroboros[ui]`) |
| Config | `~/.ouroboros/config.json` — keys live here or in env vars |

---

## Package Structure

```
ouroboros/                  # main package (import as `ouroboros`)
  __main__.py               # CLI entry point: python -m ouroboros <cmd>
  config.py                 # ALL paths, constants, key resolution, setup wizard
  state.py                  # PipelineState — stage completion tracking in draft JSON
  pipeline/
    runner.py               # Pipeline class — single source of truth for stage ordering
  draft.py                  # LLM script generation with hook competition
  broll.py                  # Pexels b-roll fetch + crop to 9:16
  tts.py                    # Edge TTS / ElevenLabs voiceover generation
  captions.py               # Whisper timestamps → ASS (burn-in) + SRT (upload)
  music.py                  # Track selection + speech-aware volume ducking
  assemble.py               # FFmpeg final video assembly
  thumbnail.py              # Pillow thumbnail generation
  upload.py                 # YouTube OAuth upload
  facebook.py               # Facebook Graph API upload (optional)
  analytics.py              # YouTube Analytics fetch + scorer weight updates
  research.py               # Topic research (web scrape / news API)
  niche.py                  # YAML niche profile loader + stage-specific config getters
  llm.py                    # Multi-provider LLM abstraction + Claude CLI/API backends
  retry.py                  # @with_retry exponential backoff decorator
  scheduler.py              # Windows Task Scheduler runner (twice-daily automation)
  topic_memory.py           # Tracks seen topics in ~/.ouroboros/topic_memory.json
  topic_scorer.py           # Virality scorer (recency/engagement/relevance/emotion/novelty)
  sfx.py                    # Sound effects layer (whoosh/impact via FFmpeg lavfi)
  upload_timing.py          # Peak engagement window waiter (niche-specific UTC hours)
  topics/                   # Multi-source topic discovery
    base.py                 # TopicCandidate dataclass + TopicSource ABC
    engine.py               # TopicEngine — orchestrates all sources
    reddit.py / rss.py / google_trends.py / newsapi.py / youtube_trending.py
  reddit_pipeline.py        # Reddit storytime pipeline (separate from main pipeline)
  reddit_fetch.py / reddit_assemble.py
  ui.py                     # Gradio web UI
  utils/
    ffmpeg.py               # ALL ffmpeg/ffprobe subprocess calls live here
    http.py                 # Shared requests.Session with retry (get_session())
    pexels.py               # Pexels API client + quota tracker
    text.py                 # extract_keywords, text helpers
niches/                     # 16 YAML niche profiles (tech, gaming, finance, ...)
backgrounds/                # bg1-5.mp4 loop videos for Reddit pipeline
music/                      # *.mp3 background tracks (EMPTY by default — add your own)
scripts/
  setup_youtube_oauth.py    # First-run YouTube OAuth flow
  refresh_facebook_token.py # Refresh long-lived Facebook page token
tests/                      # pytest suite (188/188 pass)
```

---

## Architectural Rules

### Stage ordering
Defined in `pipeline/runner.py:_ALL_STAGES`. Never reorder or skip in production:

```
draft → broll → voiceover → captions → music → assemble → thumbnail → upload
```

`state.py:STAGES` is the authoritative list including the internal `research` stage.

### How `Pipeline` works (`pipeline/runner.py`)
- `Pipeline(niche, provider, voice, lang, privacy, platform, channel_context, ab_test, require_research, visuals_mode)` — construct once, call `run(topic)`.
- `run()` returns `(success: bool, youtube_url: str, draft_path: Path)`.
- Partial execution: `draft_only(topic)`, `produce(draft_path, force=, script_override=)`, `upload(draft_path, force=, privacy=)`. The CLI commands and the Gradio UI are thin wrappers over these — NEVER re-implement stage logic outside this class (it used to exist in four copies and they diverged).
- Each stage checks `PipelineState.is_done(stage)` before executing — safe to call multiple times.
- On failure, `draft_path` is returned so the caller can invoke `resume_from(draft_path)`.
- `_try_facebook()` is non-fatal — Facebook upload failure never blocks YouTube success.
- Visuals mode resolution goes through `niche.resolve_visuals_mode(cli, draft, profile)` — one precedence chain (CLI > draft JSON > YAML), everywhere.
- Whisper word timestamps from the captions stage are persisted as a stage artifact (`words_path`) and passed to `select_and_prepare_music(words=...)` — Whisper runs once per video.
- Topics are recorded in topic memory at successful **upload**, never at draft time.

### How `resume_from` works
`pipeline.resume_from(draft_path)` reads the draft JSON (which embeds `_pipeline_state`), finds the first incomplete produce stage, and continues. Already-completed stages are skipped entirely — no redundant API calls, no re-running Whisper.

### Where paths live
ALL paths are defined in `ouroboros/config.py`. Never hardcode them elsewhere:

```python
SKILL_DIR  = Path.home() / ".ouroboros"   # root data dir
DRAFTS_DIR = SKILL_DIR / "drafts"         # draft JSONs (one per job)
MEDIA_DIR  = SKILL_DIR / "media"          # produced videos, SRTs, thumbnails
LOGS_DIR   = SKILL_DIR / "logs"           # scheduler + pipeline logs
```

Work dirs are ephemeral: `MEDIA_DIR / f"work_{job_id}_{lang}"`. Cleaned up on success.

### Constants location (`config.py`)
- `VIDEO_WIDTH = 1080`, `VIDEO_HEIGHT = 1920` — never hardcode dimensions elsewhere.
- `DEFAULT_CLAUDE_MODEL = "claude-sonnet-5"` — update here when upgrading model.
- `PLATFORM_CONFIGS` — `shorts`/`reels`/`tiktok` dimension + script length hints.
- `NICHE_TO_SUBREDDITS` — niche → subreddit list mapping for topic discovery.

### API key resolution
`config.py::_get_key(name)`: env var first, then `~/.ouroboros/config.json`. Never hardcode keys.

### Niche profiles
YAML files in `niches/`. Each profile configures voice, captions, music, thumbnail, hooks, and topic sources per niche. Load via `niche.py::load_niche(name)`. Access stage config via `get_voice_config()`, `get_caption_config()`, `get_music_config()`.

### Analytics feedback loop
`analytics.py::run_analytics()` — fetches YouTube stats, stores in `~/.ouroboros/analytics.db` (SQLite), and updates `~/.ouroboros/scorer_weights.json` so `topic_scorer.py` learns from actual performance. Needs 5+ uploaded videos before weights update.

---

## Common Commands

```bash
# Full pipeline (topic → upload)
python -m ouroboros run --news "Topic headline here" --niche tech

# Full pipeline with auto topic discovery
python -m ouroboros run --discover --auto-pick --niche gaming

# Step 1: draft only
python -m ouroboros draft --news "Topic" --niche finance

# Step 2: produce video from draft
python -m ouroboros produce --draft ~/.ouroboros/drafts/1234567890.json --lang en

# Step 3: upload
python -m ouroboros upload --draft ~/.ouroboros/drafts/1234567890.json --lang en

# Override visuals mode for one run (choices: stock_photo, motion_graphics)
# When omitted, falls back to niche YAML's visuals.mode (gaming default: stock_photo)
python -m ouroboros run --news "Topic" --niche gaming --visuals-mode motion_graphics
python -m ouroboros draft --news "Topic" --niche gaming --visuals-mode motion_graphics

# Discover trending topics without running pipeline
python -m ouroboros topics --niche tech --limit 20

# Reddit storytime pipeline
python -m ouroboros reddit --auto --mode both

# Launch Gradio web UI
python -m ouroboros ui

# Run YouTube analytics + update scorer weights
python -c "from ouroboros.analytics import run_analytics; run_analytics()"
```

---

## Things to NEVER Do

1. **Don't hardcode `~/.ouroboros` paths.** Use `SKILL_DIR`, `DRAFTS_DIR`, `MEDIA_DIR` from `config.py`.

2. **Don't add new `subprocess` calls outside `utils/ffmpeg.py`.** All ffmpeg/ffprobe calls go through `run_ffmpeg()` and `ffprobe_duration()`. The Windows ASS path escaping (`C\\:/...`) is handled there — don't replicate it.

3. **Don't add new `requests.get()` / `requests.post()` calls outside `utils/http.py`.** All outbound HTTP goes through `get_session()` which provides retry logic (3x backoff on 429/5xx), consistent User-Agent, and connection pooling.

4. **Don't read/write `analytics.db` directly.** All SQLite access is in `analytics.py`. No other file should `import sqlite3`.

5. **Don't touch `~/.ouroboros/youtube_token.json` programmatically.** Token refresh happens in `upload.py` via google-auth library. The token is OAuth2 — never overwrite it manually.

6. **Don't add TTS logic outside `tts.py`.** Edge TTS async event loop handling is subtle (detects running loop, spawns thread). Replicate and it will break under Gradio/async contexts.

7. **Don't import `ouroboros.music` for Whisper timestamps.** Use `captions._whisper_word_timestamps()` directly — `music.py` calls it internally. Avoid double-loading the Whisper model.

---

## Known Issues

| Issue | Details | Workaround |
|---|---|---|
| Edge TTS 403 | Microsoft throttles edge-tts rarely; raises `RuntimeError("Edge TTS failed: ...")` | Falls back to ElevenLabs if `ELEVENLABS_API_KEY` set, else fails |
| Whisper FP16 CPU warning | `UserWarning: FP16 is not supported on CPU; using FP32 instead` from Whisper on CPU-only machines | Harmless — Whisper auto-downgrades, captions work correctly |
| `music/` folder empty | No tracks bundled with the repo (licensing). `select_and_prepare_music()` silently returns `{}` and assemble skips music | Drop `.mp3` files into `music/` at project root |

---

## Test Suite

**Run:** `pytest tests/`

**Current pass rate: 188/188.** Lint: `ruff check ouroboros tests` must stay clean (CI enforces both — `.github/workflows/ci.yml`).

`tests/test_runner.py` covers Pipeline wiring (flag pass-through, visuals-mode propagation, stage skipping, Whisper-words reuse). When touching `pipeline/runner.py` or `__main__.py`, extend it.

---
> Source: [SaazNeupane/Ouroboros-Shorts-Pipeline](https://github.com/SaazNeupane/Ouroboros-Shorts-Pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
