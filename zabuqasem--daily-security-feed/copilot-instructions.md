## daily-security-feed

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
uv sync
npm install -g defuddle
npm install -g @github/copilot   # required by classify.py

# Run each pipeline step individually
MAX_ARTICLES=5 uv run python pipeline/fetch_feeds.py
# Classify with Copilot (default)
COPILOT_GITHUB_TOKEN=<fine-grained-pat> uv run python pipeline/classify.py
# Classify with Claude
CLASSIFIER_BACKEND=claude ANTHROPIC_API_KEY=<key> uv run python pipeline/classify.py
uv run python pipeline/write_vault.py
uv run python pipeline/persist_state.py

# Lint
uv run ruff check pipeline/
uv run ruff format pipeline/

# E2E tests (requires deployed GitHub Pages site)
uv run pytest tests/test_site.py -v
```

There are no unit tests. The staging files (`state/staged_articles.json`, `state/classified_articles.json`) are the primary debugging surface — run steps individually and inspect JSON output between runs.

## Architecture

A daily-scheduled AI-curated security research feed. `pipeline/` contains sequential Python modules; `site/` is a Jekyll static site deployed to GitHub Pages.

**Data flow** (each step writes a file consumed by the next):

```
fetch_feeds.py      →  state/staged_articles.json
classify.py         →  state/classified_articles.json
write_vault.py      →  site/_posts/YYYY-MM-DD-slug.md
persist_state.py    →  git push  →  triggers pages.yml  →  GitHub Pages rebuild
notify_discord.py / notify_slack.py  →  webhooks
```

### Key module behaviors

**fetch_feeds.py** — Parses `feeds/feeds.yaml`, fetches up to `ENTRIES_PER_FEED` entries per feed via `feedparser` with 30 concurrent workers, skips URLs in `state/processed_urls.json`, then calls `defuddle parse --json --md <url>` as a subprocess. All candidate URLs are marked seen immediately (including capped/failed). Content capped at 8000 chars; articles <200 chars discarded but still marked seen. Appends URLs to `logs/urls.txt` with `[YYYY-MM-DD]` prefix.

**classify.py** — Classifies articles using one of two backends selected by `CLASSIFIER_BACKEND` env var:
- **`copilot`** (default) — Calls `@github/copilot` npm CLI as a subprocess per article (120s timeout). Auth via `COPILOT_GITHUB_TOKEN` env var — must be a **fine-grained PAT with Copilot Requests: Read** (classic `ghp_` tokens are rejected).
- **`claude`** — Calls the Anthropic API via the `anthropic` Python SDK. Auth via `ANTHROPIC_API_KEY` env var. Model configurable via `CLAUDE_MODEL` (default: `claude-sonnet-4-20250514`). Requires `uv sync --extra claude`.

Prompt is loaded from `prompts/classify.md`. JSON is extracted by finding the first `{` and last `}` in the response. Returns `category`, `tags`, `priority` (1–3), `summary`. When `SKIP_CLASSIFY=true`, all articles are published as research/priority-2 with no tags or summary.

**write_vault.py** — Writes Jekyll markdown posts with frontmatter. Uses `source_url` (not `url`) to avoid collision with Jekyll's reserved `page.url` permalink variable.

**persist_state.py** — Single commit by `github-actions[bot]` covering `state/processed_urls.json`, `logs/urls.txt`, and `site/_posts/`.

### Priority levels (classifier output)

- **1** — High-signal: cloud/CI-CD attacks, supply chain, PortSwigger/Project Zero, novel techniques
- **2** — Mid-signal: CVE write-ups, parser bugs, less-known researcher posts
- **3** — Low urgency: useful but less immediately actionable research

### Security considerations

Feed content is treated as **untrusted input**. Malicious RSS items can contain indirect prompt-injection aimed at coercing the AI into leaking tokens. `prompts/classify.md` includes explicit defenses — do not weaken them. `COPILOT_TOKEN` must be scoped to `Copilot Requests: Read` only.

## Configuration

| What | Where |
|------|-------|
| Feed list | `feeds/feeds.yaml` |
| Articles per run cap | `MAX_ARTICLES` env var (code default: 100; workflow default: 30) |
| Entries per source | `ENTRIES_PER_FEED` env var (code default: 5) |
| Max article age | `MAX_ARTICLE_AGE_DAYS` env var (code default: 7) — skip feed entries older than this |
| Seen URLs | `state/processed_urls.json` — delete entries to reprocess |
| Classifier prompt | `prompts/classify.md` |
| Classifier backend | `CLASSIFIER_BACKEND` repo variable: `copilot` (default) or `claude` |
| Copilot auth | `COPILOT_TOKEN` secret (fine-grained PAT, Copilot Requests: Read) |
| Claude auth | `ANTHROPIC_API_KEY` secret (required when backend is `claude`) |
| Claude model | `CLAUDE_MODEL` repo variable (default: `claude-sonnet-4-20250514`) |
| Skip AI classification | `SKIP_CLASSIFY=true` repo variable |
| Notification channels | `NOTIFY_CHANNELS`: `both`, `discord`, or `slack` |
| Pages URL | `PAGES_URL` repo variable (used in Discord/Slack "Browse feed" links) |
| Creator branding | Hardcoded in `pipeline/config.py` (`zAbuQasem`, `zeyad-abulaban`) |

## Workflows

| Workflow | Schedule | Purpose |
|----------|----------|---------|
| `security_feed.yml` | 05:00 Amman daily | Full pipeline |
| `pages.yml` | On push to main or after `security_feed.yml` | Jekyll build + GitHub Pages deploy |
| `secret_rotation_reminder.yml` | 1st of month, 09:00 Amman | Discord/Slack reminder to rotate `COPILOT_TOKEN` |

---
> Source: [zAbuQasem/daily-security-feed](https://github.com/zAbuQasem/daily-security-feed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
