## morning-intel

> handles auth.

# morning-intel skill pack — installer instructions for Claude Code

This repo is an INSTALLABLE SKILL PACK, not an app. When a user opens this repo
in Claude Code and asks to "install", "set up", "set me up", or similar — or on
first interaction if nothing is installed yet — run the guided installation
below. Be conversational: one step at a time, confirm before writing files.

## What gets installed

Three skills copied into `~/.claude/skills/`:

| Skill | Role |
|---|---|
| `morning-intel` | The brief itself: 5 fetchers + synthesis instructions |
| `github-trending` | GitHub 7d/30d/velocity fetcher (morning-intel delegates to it) |
| `yt-search` | Optional fallback YouTube search via yt-dlp (no API key needed) |

## Installation steps

### 1. Prerequisites check

- `python --version` — needs Python 3.9+ (uses zoneinfo, pathlib). If missing, point to python.org/downloads and stop.
- Ask which OS they're on if not obvious from the environment (scripts are cross-platform; only `run_all.ps1` is Windows-specific).

### 2. Copy skills

Copy `skills/morning-intel`, `skills/github-trending`, `skills/yt-search` from
this repo into `~/.claude/skills/`. If a directory already exists there, ASK
before overwriting (they may have a customized copy).

### 3. Configure ~/.claude/.env

Create or append to `~/.claude/.env` (never commit this file anywhere; see
`.env.example` for the template). Walk through each key:

**AGENTIC_OS_VAULT** (optional, default `~/the-vault`)
The folder where briefs land: `<vault>/inbox/research/morning-intel/`. Any
folder works — an Obsidian vault is ideal. Create the folder if it doesn't
exist.

**YOUTUBE_API_KEY** (recommended — the YouTube Radar section needs it)
Free, 5 minutes:
1. console.cloud.google.com → create a project (any name)
2. "APIs & Services" → "Library" → enable **YouTube Data API v3**
3. "APIs & Services" → "Credentials" → "Create credentials" → "API key"
4. Paste the key when I ask — I'll write it to `~/.claude/.env` myself; don't put it anywhere else.
Free quota is 10,000 units/day; one brief run uses ~410. Without it, the
fetcher writes `status=mock` and the brief skips YouTube (or uses yt-search).

**GITHUB_TOKEN** (optional — only for heavy re-running)
github.com/settings/tokens → "Generate new token (classic)" → NO scopes needed
(public read only). Lifts rate limits from 60/hr to 5000/hr.

### 4. Gmail connector (optional)

The inbox-triage section uses Anthropic's Gmail connector (claude.ai →
Settings → Connectors → Google Mail). It's OAuth through Google — Claude never
sees the password. If the user skips this, the brief simply omits the Inbox
section. Never ask the user for their Google password; the connector UI
handles auth.

### 5. Personalize profile.md

Ask the user 3 short questions, then write
`~/.claude/skills/morning-intel/profile.md`:
1. Who are you / what do you do? (e.g. "YouTube creator covering Claude Code", "engineering lead evaluating AI tooling")
2. What should the "So What" section produce? (content ideas / adoption decisions / competitive intel / all)
3. Anything you already ship or maintain that angles should extend? (channels, products, franchises, repos)

profile.md format: freeform markdown, ~10 lines. The synthesis step reads it to
tailor the payoff section.

### 6. Smoke test

Run the five fetchers in parallel (see morning-intel SKILL.md Step 1). Show the
user the per-source statuses. `mock`/`error` on youtube = key missing/wrong;
everything else should be `ok` on a normal network. Then offer to run the full
brief end-to-end.

### 7. Optional: schedule it

- **Windows:** Task Scheduler daily → `powershell -File ~/.claude/skills/morning-intel/scripts/run_all.ps1` for the fetch layer, then run `/morning-intel` in Claude Code when they sit down (fetch layer pre-warmed), OR a fully headless `claude -p "/morning-intel"` if they use headless runs.
- **macOS/Linux:** cron/launchd on the five python scripts + `claude -p`.
- If Gmail is enabled, keep scheduling LOCAL (not cloud-scheduled agents) — it's personal inbox data.

## Rules for the installer

- Never commit, echo, or log API keys. Keys go in `~/.claude/.env` only.
- Never overwrite an existing skill or .env entry without asking.
- If a step fails, fix or skip gracefully — a partial install (no YouTube, no Gmail) still produces a useful brief.

---
> Source: [cth9191/morning-intel](https://github.com/cth9191/morning-intel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
