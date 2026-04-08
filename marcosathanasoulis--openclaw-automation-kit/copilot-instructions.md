## openclaw-automation-kit

> cd ~/openclaw-automation-kit && git pull origin main

# OpenClaw Automation Kit — Agent Instructions

## Getting Latest Code
```bash
cd ~/openclaw-automation-kit && git pull origin main
```
Always pull before making changes to avoid conflicts.

## Architecture: Two Browser Runners

The system has two machines that can run browser automation:

### Mac Mini (primary) — `marcoss-mac-mini.local`
- **Chrome CDP**: port 9222 (real Chrome with user profile)
- **Kit path**: `~/openclaw-automation-kit/`
- **Credentials**: macOS automation keychain (`~/Library/Keychains/automation.keychain-db`)
- **SMS 2FA**: Direct access to `~/Library/Messages/chat.db`
- **CDPLock**: `/tmp/browser_cdp.lock` — file-based lock to prevent concurrent browser use
- **Start Chrome**: `/tmp/launch_chrome_cdp.sh`

### Home-mind (secondary) — `home-mind.local` (Ubuntu)
- **Chrome CDP**: port 9226 (real Chrome via Xvfb on display :99)
- **Kit path**: `~/openclaw-automation-kit/`
- **Credentials**: File-based at `~/.credentials.json` (add new sites here)
- **SMS 2FA**: Via SSH to Mac Mini (`/tmp/read_sms_code.py`)
- **CDPLock**: `/tmp/browser_cdp.lock`
- **Start Chrome**: `/tmp/start_chrome_real.sh`

## Runner Selection (automatic)

The AI agent on home-mind (`src/agent/tools.py`) automatically picks the runner:

1. Checks Mac Mini's `/tmp/browser_cdp.lock` via SSH
2. If lock exists and PID is alive → Mac Mini is **busy** → runs locally on home-mind
3. If no lock or stale lock → Mac Mini is **free** → SSHes search to Mac Mini
4. If Mac Mini unreachable → falls back to home-mind

Function: `_pick_browser_runner()` in `src/agent/tools.py`

## CDPLock Protocol

**CRITICAL**: Only one BrowserAgent can use Chrome at a time per machine.

- Lock file: `/tmp/browser_cdp.lock`
- Contains: `{"pid": 12345, "started": "2026-02-16T20:31:25"}`
- The BrowserAgent acquires the lock on start and releases on exit
- If you see a lock from a dead PID, it's safe to delete
- **Never run two browser automations on the same Chrome simultaneously**

## Key Config (.env)

Each machine's `~/openclaw-automation-kit/.env` must have:
```
OPENCLAW_USE_BROWSER_AGENT=true
OPENCLAW_BROWSER_AGENT_PATH=/path/to/src/browser
OPENCLAW_CDP_URL=http://127.0.0.1:<port>
ANTHROPIC_API_KEY=sk-ant-...
```

Mac Mini uses port 9222, home-mind uses port 9226.

## Running a Search

```bash
cd ~/openclaw-automation-kit
set -a && source .env && set +a
.venv/bin/python -m openclaw_automation.cli run-query \
  --query "Search United business SFO to SIN June 30 for 2 people under 250k miles"
```

## Adding Credentials for Home-mind

Home-mind uses `~/.credentials.json` instead of macOS keychain:
```json
{
  "www.united.com": {"user": "ka388724", "password": "..."},
  "www.delta.com": {"user": "9396260433", "password": "..."}
}
```

## Airline Runners

Located in `library/<airline>_award/runner.py`. Each has:
- `_goal()` — generates step-by-step instructions for the BrowserAgent
- `run()` — entry point called by the engine
- `_parse_matches()` or uses `result_extract.py` for parsing

### United (`library/united_award/runner.py`)
- Login: MileagePlus ka388724, SMS 2FA from sender 26266
- Approach: Cash URL first → click "Money + Miles" tab → re-enter date → Update
- "Remember me" checkbox instruction included in login goal

### Singapore Airlines (`library/singapore_award/runner.py`)
- Hybrid: BrowserAgent login + Playwright form fill + scrape
- KrisFlyer 8814147288, Akamai CAPTCHA risk

### Delta, AeroMexico, ANA, JetBlue
- See individual runner.py files for details

## Git Workflow
- **Branch**: each agent must work on its own topic branch (`codex/*`, `claude/*`, `gemini/*`)
- **Commit**: include what changed and test results
- **Push**: push your branch and open/update a PR; do not commit directly to `main`

## CI Gate (Required)
- If CI fails on your branch or on `main` for your changes, fixing CI is blocking work.
- Run lint/tests locally before handoff: `ruff check .` and `pytest`.
- Do not hand off or merge while required CI checks are red.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcosathanasoulis)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/marcosathanasoulis)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
