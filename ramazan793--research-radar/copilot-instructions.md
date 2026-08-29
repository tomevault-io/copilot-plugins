## research-radar

> Daily arXiv digest daemon. It fetches each day's new papers in your chosen categories,

# Research Radar

Daily arXiv digest daemon. It fetches each day's new papers in your chosen categories,
scores them against *your* research interests, deep-reads the most promising ones, and
writes an HTML/Markdown digest (with an optional Telegram ping). Field-agnostic: what to
care about lives entirely in `context/research_context.md`, not in the code. Typically
run from cron on an always-on machine.

## Pipeline

```
cron (daily; run time set by your crontab, digest dates use timezone_offset_hours)
  -> src/main.py
       1. Load config.yaml + context/research_context.md
       2. Fetch papers: per category, RSS (today) + API lookback window, merged
          - API lookback always runs (backfills missed runs/weekends; dedup makes it cheap)
          - Dedup across categories by arxiv_id
          - Reports per-category fetch outages (RSS+API both errored) distinctly from quiet days
       3. Filter against data/seen_papers.json (skip already-seen)
       4. PASS 1 — Abstract scoring (scorer.py)
          - Batches of batch_size papers -> SCORING backend/model (cheap/fast)
          - Title + abstract only, scored 1-10 against research_context.md
          - Returns JSON: [{arxiv_id, score, reason}]. Failures isolate per-PAPER: each
            retry re-asks only the still-unscored papers; any left over after retries are
            recorded (with a reason) to data/digests/YYYY-MM-DD_failures.json, not dropped
       5. PASS 2 — Deep read (deep_reader.py)
          - Papers scoring >= deep_read_min_score (default 7), capped at max_deep_reads
          - Downloads full PDF + extracts text via pymupdf (deterministic, no model)
          - Sends full paper to DEEP_READ backend/model (strong/slow)
          - Returns: summary, what_it_opens, key_insights, ideas, limitations,
            refined_score, verdict
          - Saved to data/digests/YYYY-MM-DD_deep.json
          - Retryable failures (PDF download/extract error, or model failure after retries)
            are held for retry like scoring failures; text-too-short is skipped permanently
       6. Generate HTML + markdown digests via Jinja2 templates
       7. Update seen_papers.json (atomic write; after the digest; skipped on --dry-run).
          Papers that failed scoring OR deep-read are held back (left unseen) so the next
          run retries them
       8. Optional: Telegram notification with top papers (clickable arXiv links)
          + digest link  (skipped if Telegram is not configured)
```

Both model passes go through the model-agnostic `backends/` layer. `main.resolve_task()`
reads the `tasks:` section of `config.yaml` and resolves a `(backend, model, effort)`
triple **per task** — so scoring can run on a cheap model and deep-read on a strong one,
even across different providers.

## File Map

```
src/
  main.py              # Entry point + orchestrator. resolve_task() picks per-task
                       #   backend/model/effort; accumulates token usage. Supports --dry-run.
  arxiv_fetcher.py     # RSS + API fetching, dedup, date filtering
  scorer.py            # Pass 1: batched abstract scoring; salvages partial batches, isolates
                       #   per-paper failures, retries only the unresolved papers
  deep_reader.py       # Pass 2: PDF download + pymupdf text extraction, then deep analysis
  digest_generator.py  # Jinja2 rendering of HTML/MD digests + index.html
  telegram_bot.py      # Optional Telegram notifications (digest, no-papers, errors)
  backends/            # Model-agnostic inference layer
    base.py            # LLMBackend ABC + complete(); Usage / CompletionResult dataclasses;
                       #   shared JSON extract/validate helpers (backend-agnostic)
    _cli.py            # Robust CLI executable resolution (cron-safe PATH lookup)
    codex.py           # Codex CLI backend (codex exec --json; ChatGPT plan credits)
    claude.py          # Claude Code CLI backend (claude -p --output-format json; Claude.ai plan)
    openai_compatible.py # Any OpenAI /chat/completions endpoint (Ollama/vLLM/OpenRouter/
                       #   OpenAI/...). Also the heavily-commented "write your own" template.
    __init__.py        # get_backend(name, config) factory + backend registry
  codex_cli.py         # Deprecated shim (re-exports CLI resolution; use backends/ instead)

config.yaml            # backends + per-task models + thresholds + categories + timezone
context/
  research_context.example.md  # generic template — copy it and edit
  research_context.md          # YOUR research profile + scoring guide. User-provided
                               #   (copied from .example, gitignored). Drives all scoring.
templates/
  digest.html.j2       # HTML digest (score badges, deep-analysis boxes)
  digest.md.j2         # Markdown digest
  index.html.j2        # Archive index page
data/
  seen_papers.json     # Dedup state: {arxiv_id: {title, date_first_seen, score}}
  digests/             # Generated digests: YYYY-MM-DD.html, .md, _deep.json,
                       #   _failures.json (papers that couldn't be scored), + index.html
  cron.log             # Cron output log

pyproject.toml         # uv app (package=false); deps: requests, feedparser, python-dotenv,
                       #   pyyaml, jinja2, pymupdf
.env                   # TELEGRAM_BOT_TOKEN, TELEGRAM_ARXIV_CHAT_ID, optional API keys (not in git)
install.sh             # Optional convenience setup (uv, venv, deps, CLI auth, web server)
setup_cron.sh          # Install the daily cron job (idempotent, won't duplicate)
```

## Setup

```bash
# 1. Install dependencies into .venv (from pyproject.toml + uv.lock)
uv sync

# 2. Tell it what you care about (this file drives all scoring)
cp context/research_context.example.md context/research_context.md
$EDITOR context/research_context.md

# 3. Pick your backends/models in config.yaml (claude / codex / openai), and sign in to
#    whichever CLI backend you use:
#      claude   -> run `claude` once and log in with your Claude.ai account
#      codex    -> run `codex login` once (ChatGPT account)
#      openai   -> set base_url + api_key_env in config.yaml / .env

# 4. Dry run: scores, deep-reads, writes digests — no Telegram, no state update
.venv/bin/python src/main.py --dry-run

# 5. (Optional) Telegram + daily cron
#    add TELEGRAM_BOT_TOKEN / TELEGRAM_ARXIV_CHAT_ID to .env, then:
bash setup_cron.sh
```

`install.sh` automates the boilerplate non-interactively (runs `uv sync`, scaffolds
`context/research_context.md` and `.env`, detects which backend CLIs you have, and with
`--with-web` sets up a systemd static-file server), but `uv sync` + a dry run is enough.

## Running

```bash
# Full run (scores, deep-reads, generates digest, sends Telegram if configured)
.venv/bin/python src/main.py

# Dry run (no Telegram, no seen_papers update, still generates digests)
.venv/bin/python src/main.py --dry-run
```

## Key Design Decisions

- **Model-agnostic backends.** All inference goes through `LLMBackend.complete()`. Three
  ship in the box: `claude` (Claude Code CLI) and `codex` (Codex CLI) authenticate via your
  subscription and draw on plan credits rather than a metered API key; `openai` hits any
  OpenAI-compatible HTTP endpoint (Ollama, vLLM, LM Studio, OpenRouter, OpenAI, ...). Add a
  provider by dropping a file in `backends/` and registering it in `_REGISTRY` — everything
  else (batching, JSON validation, retries, digest, Telegram) is provider-agnostic.
- **Per-task model selection.** Pass 1 scores many abstracts, so it uses a cheap/fast model;
  Pass 2 deep-reads a few full papers, so it uses a strong one. With Codex the quality lever
  is reasoning `effort`; with Claude it's mostly model choice (Sonnet vs Opus). Both are set
  in `tasks.scoring` / `tasks.deep_read` in `config.yaml`.
- **Field-agnostic.** No domain is hardcoded anywhere in the code. The scoring criteria for
  both passes come entirely from `context/research_context.md`.
- **RSS + always-on API backfill.** RSS carries today's new papers; the API lookback window
  (`arxiv_lookback_days`) is fetched every run too and merged (deduped), so a missed run or a
  weekend gets backfilled instead of silently lost. Run daily — once per category per run is
  gentle on the API. A category that errors on BOTH RSS and API is reported as a fetch failure
  (a distinct Telegram alert) rather than being mistaken for a quiet day.
- **Two-pass scoring.** Pass 1 cheaply triages every abstract; Pass 2 deep-reads only the top
  papers with full PDF text for rich analysis.
- **Deterministic where it can be.** PDF download/extraction, dedup, batching, and JSON
  parse/validate/retry are non-model code; only the actual scoring and analysis call a model.
- **Atomic writes.** `seen_papers.json` is written to a temp file then renamed — crash-safe.
- **Optional Telegram.** The pipeline runs fully without it; if the token/chat-id are blank,
  notifications are simply skipped. Configure via `.env`.
- **Configurable timezone.** `timezone_offset_hours` (default 0 = UTC) sets which calendar day
  a run is filed under, independent of the host clock or cron schedule.
- **Digest as static files.** Digests are plain HTML in `data/digests/`; `install.sh` can serve
  them with `python3 -m http.server` (host/port from `digest_host` / `digest_port`).

## Config Reference

| Key | Default | What |
|-----|---------|------|
| `arxiv_categories` | cs.CV, cs.GR | arXiv categories to monitor |
| `arxiv_lookback_days` | 3 | Days back for the always-on API backfill |
| `timezone_offset_hours` | 0 | Offset from UTC (hours) used to date digests |
| `backends` | claude, codex | Provider-level settings, keyed by backend name (e.g. `openai.base_url`, `openai.api_key_env`) |
| `tasks.scoring` | claude / sonnet / low | Pass 1 `{backend, model, effort}` — cheap/fast |
| `tasks.deep_read` | claude / opus / high | Pass 2 `{backend, model, effort}` — strong/slow |
| `batch_size` | 10 | Abstracts per scoring call (amortizes CLI overhead) |
| `min_relevance_score` | 5 | Minimum score to appear in the digest |
| `max_papers_in_digest` | 20 | Cap on papers in the digest |
| `deep_read_min_score` | 7 | Minimum score to qualify for a full deep-read |
| `max_deep_reads` | 10 | Cap on deep reads per run |
| `top_papers_in_telegram` | 5 | Papers shown in the Telegram message |
| `telegram_bot_token` | "" | Telegram bot token (override via `.env`; blank = skip) |
| `telegram_arxiv_chat_id` | "" | Telegram chat id (override via `.env`; blank = skip) |
| `digest_host` / `digest_port` | 0.0.0.0 / 8477 | Bind for the static digest server |
| `digest_base_url` | localhost:`digest_port` | Public URL used in Telegram digest links |

> Backward compatibility: if `tasks:` is absent, `resolve_task()` falls back to legacy
> top-level `codex_model` / `codex_reasoning_effort` keys on the `codex` backend.

## Extending

- **New backend / model provider:** Copy `backends/openai_compatible.py` (the template),
  implement `complete()`, and register the class in `backends/__init__.py`'s `_REGISTRY`.
- **New source (Semantic Scholar, X/Twitter, ...):** Add a fetcher that returns the same
  paper dict (`arxiv_id`/id, `title`, `abstract`, `url`) and call it from `main.py`. A
  second Telegram target is reserved via `telegram_twitter_chat_id`.
- **New categories:** Edit `arxiv_categories` in `config.yaml`.
- **Adjust scoring:** Edit `context/research_context.md` — its scoring guide drives both passes.

---
> Source: [ramazan793/research-radar](https://github.com/ramazan793/research-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
