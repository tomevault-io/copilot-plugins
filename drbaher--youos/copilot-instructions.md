## youos

> >


# YouOS — Personal Email Copilot

YouOS is a full local Python app (not an instruction-only snippet). It drafts email replies in your style, grounded in your real past replies.

**Naming:** *YouOS* is the shared app/package. During setup it personalizes its name to *<First>OS* (e.g. **BaherOS**) for **your local instance** at `YOUOS_DATA_DIR=~/YouOS-Instances/<you>/` — that's a local config/data directory, not a fork of the project.

## Safety & impact

Read these before installing:

- **Sensitive data ingested locally.** YouOS reads your Gmail (sent + threads), Google Docs, and optional WhatsApp exports into a local SQLite DB. This stays on your machine, but it *is* sensitive content — protect the data directory (`YOUOS_DATA_DIR/var/youos.db`) accordingly.
- **Installation runs local package code.** `./scripts/install.sh` builds a `.venv` and installs this repository — review the source (especially `scripts/install.sh`, `pyproject.toml`, `app/`) before running.
- **Background persistence is opt-in.** YouOS only runs as a launchd LaunchAgent if you explicitly run `youos service install`. Foreground `youos serve` does not persist.
- **Scheduled/nightly runs are opt-in.** Ingestion, fine-tuning, and autoresearch only run automatically if you've enabled the nightly pipeline; `youos improve` is the manual equivalent.
- **External model fallback is optional.** Drafting is on-device by default. Cloud fallback only fires if `review.draft_model` or `model.fallback` is set to a cloud model — set `model.fallback: none` and `review.draft_model: local` for strict local-only operation.

## Install and runtime model

- Install is **manual**: run `./scripts/install.sh` (Python 3.11+) — it creates a `.venv`, installs YouOS, sets up the on-device model (**MLX**) on Apple Silicon, and runs the doctor
- Note: this executes local package install code from this repository; review source before installing
- Requires `python3` (3.11+) and a Google ingestion backend — `gog` (default), `gws`, or native (see Credentials)
- Drafts run on your **fine-tuned local model by default** (Qwen + your LoRA, served warm); the cloud is only a cold-start before your model is trained, or a fallback
- Optional runtime path override: `YOUOS_DATA_DIR`

## Credentials and configuration

- Required: a Google ingestion backend for Gmail/Docs — `gog` (default), `gws`, or native (`youos[google]` + OAuth); set `ingestion.google_backend`
- Optional: Claude CLI/API credentials only for the cold-start/fallback (`review.draft_model` / `model.fallback`)
- For strict local-only: set `review.draft_model: local` and `model.fallback: none`

## Trigger phrases

- "draft a reply to this email"
- "write this email for me"
- "how would I respond to this"
- "what would I say to"
- "help me reply"
- "draft in my style"
- "youos"
- "my email copilot"
- "email copilot"
- "my copilot"
- "generate a draft"
- "reply draft" / "email draft" / "draft reply"
- "compose reply"
- "write a response"
- "email response"
- "how do I usually reply to"
- "reply to this"
- "help me write"
- "write an email"
- "compose a response"
- "email assistant"
- "my writing style"
- "train on my emails"
- "anything important in my inbox"
- "triage my email"
- "what did the agent do"
- "push to gmail drafts"
- "dismiss as noise"
- "save as a training pair"

## Requirements

- Apple Silicon Mac (M1/M2/M3/M4) with 8GB+ RAM (16GB recommended) — the local model runs on MLX (`./scripts/install.sh` sets it up)
- Python 3.11+
- A Google ingestion backend for Gmail/Docs: [gog CLI](https://github.com/openclaw/gog) (default), Google's `gws` CLI, or the native Google API (`youos[google]` + OAuth)
- ~5GB free disk space
- Run the UI locally by default (do not expose publicly unless intentionally secured)

## Quick start

```bash
# Install (creates .venv, installs YouOS + MLX on Apple Silicon, runs the doctor)
cd youos
./scripts/install.sh
source .venv/bin/activate

# Check system requirements (Python, Google backend, MLX, disk space, etc.)
youos doctor

# Run setup wizard (identity, ingestion, style analysis) — or open /welcome in the browser
youos setup

# Draft a reply (uses your local fine-tuned model by default)
youos draft "paste inbound email here"
youos draft --sender john@company.com "email text"

# Run the web UI (then open /feedback, /stats, /settings, /about)
youos serve
youos service install     # or run it as a background service (starts at login)

# Compare the backends on YOUR mail, ranked by how closely each sounds like you
youos compare-models --limit 30 --semantic

# Warm local-model server (loaded once for fast drafting)
youos model server status

# Check status / view stats
youos status
youos stats

# Run the nightly pipeline manually (add --verbose for step-by-step output)
youos improve --verbose

# Golden benchmark (10 curated cases)
youos eval --golden

# Full corpus health report (pairs, quality scores, top senders)
youos corpus

# Ingest a WhatsApp chat export (optional — augments your corpus)
youos ingest --whatsapp ~/Downloads/WhatsApp-Chat.txt

# Add a sender note (immediately rebuilds their profile)
youos note john@company.com "integration partner, prefers bullet points"

# Submit a feedback pair directly from the terminal
youos feedback --inbound "email text" --reply "your reply" --rating 4

# Teardown (remove all data, keep code)
youos teardown
```

## Drafting inside Gmail

Install the **YouOS browser extension** (Chrome/Edge/Brave) — it lives in the repo's
`extension/` folder ([homepage](https://github.com/DrBaher/youos/tree/main/extension)),
and the web UI's Gmail page has one-click "Load unpacked" steps. The extension adds a
panel to Gmail:
- Open an email → click the teal ✉ launcher → the panel opens
- Sender + message auto-detected; add an instruction or pick a tone
- Click **Generate** → drafted in your voice; **Insert into Gmail** drops it in the reply box
- Rate 1–5 and **Submit feedback** — YouOS learns from it, same as the Review Queue

A bookmarklet remains as a no-install fallback (it can break when Gmail changes its markup;
the extension doesn't).

## Autonomous triage (opt-in)

YouOS can sweep your unread inbox on a schedule and pre-draft replies for the ones that actually want a reply. **Never auto-sends** — drafts surface at **`/triage`** for you to review, edit, and either push to your Gmail Drafts folder or copy-paste manually.

**Turn it on**:
```bash
youos config set agent.enabled true
youos config set agent.interval_minutes 15
youos config set agent.standing_instructions "today I'm OOO; politely decline meetings"
youos serve
```

Every N minutes (default 15) the loop:
1. Fetches unread inbox via the configured Google backend
2. **Filters**: hard-skips `List-Unsubscribe` newsletters, automation domains (`@github.com`, `@gitlab.com`, `*.atlassian.net`, `fireflies.ai`, `otter.ai`, etc.), `mailer-daemon`/bounces, CI/build subject patterns (`[Org/Repo]`, `PR run failed`); soft-penalises `noreply@` + operational mailboxes (`billing-support@`, `notifications@`, `calendar-notification@`)
3. **Scores** survivors with prior-history boost, question/imperative detection, cold-outreach heuristics, length-based signals
4. **Drafts** what crosses the threshold using the same local-LoRA pipeline `/feedback` uses (standing instructions threaded into the prompt; cold outreach gets an additional decline-nudge)
5. **Persists** to `agent_pending_drafts`; **macOS notification** when new drafts land
6. **Audit log** — every sweep records what was attempted, by what trigger (scheduled / manual / api), with counts, duration, and any per-message errors

`/triage` shows three things: the **pending drafts** with inbound + editable draft side-by-side, the **surface for review** (borderline cases not auto-drafted), and **Recent activity** (last 15 sweeps).

**Send path** (Phase 2.1, gog backend): **Push to Gmail Drafts** creates a real Gmail Draft on the original thread; you finish-and-send from Gmail. **Mark sent manually** records the row as sent without writing to Gmail (for when you sent through another channel).

**Safety**:
- `agent.skip_senders` — comma-separated emails or `@domain` entries to hard-skip
- `agent.daily_draft_cap` — per-UTC-day quota per account; defends against a runaway loop on a noisy inbox
- `agent.strict_local` — refuses cloud fallback during background triage (interactive `/feedback` unaffected)
- Manual one-shot: `youos triage [--account] [--window 3d] [--limit 8] [--dry-run]`

**Self-tuning** (opt-in): Dismiss with a categorical reason (`noise` / `wrong_sender` / `wrong_content` / `already_handled` / `other`) — these aggregate into a dismissal-rate metric and a "promote to skip-list" candidate list on `/triage`'s Agent health card. When `agent.auto_promote_skip_senders` is on, senders dismissed as `noise` 3+ times in 30d are auto-added to `agent.skip_senders` at the end of each sweep — fully self-tuning loop. Default off; the candidates also appear with checkboxes for one-click manual promotion.

**Remote dismissal**: apply the Gmail label `YouOS/skip` to a thread from any Gmail client (phone, web, work laptop) → next sweep marks the matching pending row dismissed-as-noise + removes the label. Full setup in `docs/REMOTE_ACCESS.md`.

## Integrations (orchestrator backend)

YouOS is also a backend that **orchestrators** can drive — Hermes, OpenClaw, a Telegram/WhatsApp/Slack bot, or any HTTP client. The end-user lives in their existing chat app; the orchestrator handles the email category by calling YouOS:

```
You (Telegram) → "Anything important?" → Hermes → GET /api/agent/digest → paraphrases summary
You (Telegram) → "Push #12 to Gmail" → Hermes → POST /api/agent/pending/12/push_to_gmail
```

**Surface area** (zero added config — works out of the box):
- `GET /openapi.json` — auto-generated OpenAPI 3.x for tool-discovery
- `GET /docs` — Swagger UI
- `GET /api/agent/digest?account=&days=1` — `summary` headline + counts + top-5 pending rows with action handles
- `GET /api/agent/pending` — full queue
- `POST /api/agent/pending/{id}/{push_to_gmail,dismiss,amend,mark_sent,save_as_feedback_pair}` — actions
- `GET /api/agent/observability` / `dismissal_stats` / `sweeps` — drill-downs
- `POST /api/agent/triage` — trigger sweep
- `POST /api/agent/skip_senders/promote` — bulk-add to skip list

**Auth**: `X-YouOS-Token: <token>` header. Mint with `youos token-create` (stored hashed; shown once). Per-account isolation via `?account=...` on every endpoint.

**Network**: bind to `0.0.0.0` and put YouOS behind Tailscale; the orchestrator runs on the same Tailnet (`docs/REMOTE_ACCESS.md` covers the setup).

Full recipe + ~30-line Telegram bot example in `docs/INTEGRATIONS.md`. **YouOS already ships an OpenClaw bundle** (this file + `clawhub.json`) and is published on ClawHub; Hermes-style orchestrators discover the surface via `/openapi.json`.

**For LLM-driven agents operating YouOS at runtime**: read `docs/AGENT_OPERATIONS.md` — it covers when to call what, idempotency, HTTP error handling, disambiguation patterns, trust boundaries, paraphrasing guidance, side-effect tables, and a worked end-to-end conversation. The OpenAPI spec is the schema; that doc is the runtime contract.

## How it works

1. Ingests Gmail, Google Docs, WhatsApp exports — plus organic pairs from emails you sent without YouOS
2. Builds a retrieval index — BM25 + query expansion + semantic (LRU-cached) + multi-intent + per-account isolation + same-thread 2× + subject + topic signals + sender-type boosts + quality scores + relative confidence thresholds
3. When you ask for a draft: detects multi-intent, retrieves score-ranked thread-deduplicated exemplars (reply preserved 600 chars, inbound trimmed 400), prompt token budget enforced; generates with per-mode persona + first-name greeting on your **fine-tuned local model by default** (Qwen + your LoRA, served warm via `mlx_lm.server` so it's fast and on-device). The cloud is only a cold-start (before your model is trained) or a fallback; a per-draft model badge + the Stats "Drafting with" row always show which model actually ran
4. Every email you review trains the model further — curriculum-ordered, quality-filtered, training pairs deduplicated by similarity, DPO pairs supported; nightly pipeline skips steps when data insufficient
5. Nightly: ingests + organic pairs, incremental persona re-analysis (90-day weighted, EWMA avg words, p25/p75 confidence intervals), fine-tunes (with golden eval check), runs autoresearch on rotating benchmark sample
6. Autoresearch benchmarks rotate weekly (seeded re-sample) — prevents overfitting to fixed test cases; golden eval composite tracked in pipeline log
7. Style drift detection: Stats dashboard flags when your writing patterns shift significantly
8. Your best-rated, least-edited replies surface higher in future retrievals via quality scoring
9. Sender profiles track reply-time patterns and topics; `youos note` immediately rebuilds that contact's profile
10. Submit feedback from terminal: `youos feedback --inbound "..." --reply "..." --rating 4`
11. Setup wizard asks for internal domains — accurate sender classification from day one
12. Facts store (`/api/facts`) — save context about contacts, projects, and preferences; facts are injected into generation prompts automatically for context-aware drafts
13. Auto fact extraction — sender notes and feedback notes are parsed automatically on save using 15+ rule patterns (preferences, timezone, schedule, sign-offs, roles, relationships, project metadata); negation-aware with confidence scoring; LLM (Claude CLI) fallback for unstructured notes; fact deduplication/merging on upsert
14. Measure it, don't guess — `youos compare-models` drafts your held-out replies under each backend and scores them against what you actually wrote (voice-match), so you can verify the local fine-tuned model beats the cloud on your mail
15. Readiness gate — a soft "preparing your voice model" banner holds you back from relying on drafts until your model is trained **and** benchmarked (with a "Run benchmark now" action); drafting still works meanwhile
16. Becomes *your* OS — during setup the app personalizes its name from your first name (e.g. Baher → BaherOS)

## Security & privacy notes

- Gmail/Docs ingestion uses your configured Google backend's auth (`gog` / `gws` / native OAuth); review connected accounts before ingestion
- Drafting is on-device by default; the cloud is only the cold-start/fallback. If it's used (`review.draft_model: claude` or `model.fallback: claude`), inbound email/context is sent to Claude for that draft
- For strict local-only operation, set `review.draft_model: local` and `model.fallback: none` in `youos_config.yaml`
- Data location defaults to local instance paths under `YOUOS_DATA_DIR` (e.g. `~/YouOS-Instances/<you>/`), or the repo's `var/`
- Review `PRIVACY.md` before first ingestion/deployment

## Provenance

- Source/homepage: https://github.com/DrBaher/youos
- This skill bundles a full local Python app and is intended for explicit local install/review before use.

---
> Source: [DrBaher/youos](https://github.com/DrBaher/youos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
