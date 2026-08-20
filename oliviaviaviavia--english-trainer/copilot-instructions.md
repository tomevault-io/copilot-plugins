## english-trainer

> This file tells an AI coding agent (OpenAI Codex, Claude Code, Cursor, Windsurf, Devin, etc.)

# AGENTS.md — setup & run instructions for AI coding agents

This file tells an AI coding agent (OpenAI Codex, Claude Code, Cursor, Windsurf, Devin, etc.)
how to set up and run this project for the user. Follow it top to bottom.

This is a **web app with a local Python backend** (FastAPI + uvicorn). Frontend is a single
static `index.html` served by the backend. No build step, no database.

## Goal

Get the server running at `http://localhost:8000` so the user can open it in a browser.
Three of the four practice modes (blind listening / listening / connected reading) work with
**zero credentials** — get those working first. Two optional features need user-supplied
secrets (see "Secrets the human must provide" below); set up the environment, then ask the user.

## Prerequisites (check, install if missing)

- **Python 3.10 or 3.11** (newer may work; some pinned deps were built against these).
- **ffmpeg** on PATH (needed for audio extraction & pronunciation assessment).
  - Windows: `winget install Gyan.FFmpeg`
  - macOS: `brew install ffmpeg`
  - Debian/Ubuntu: `sudo apt-get install -y ffmpeg`

## Setup

Run the helper script, or the equivalent manual steps.

- **Windows (PowerShell):** `./setup.ps1`
- **macOS / Linux:** `bash setup.sh`

Manual equivalent:

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate    macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
```

## Run

Always use the venv's Python, not the system/conda Python:

```bash
# Windows:
.venv\Scripts\python -m uvicorn app:app --port 8000
# macOS/Linux:
.venv/bin/python -m uvicorn app:app --port 8000
```

Then tell the user to open **http://localhost:8000** in a browser.
Use `localhost` (not a LAN IP) so the browser grants microphone access for the shadowing mode.

## Verify it works

- `GET http://localhost:8000/api/health` should return JSON like `{"ok":true,"azure":...}`.
- Open the page, paste any English paragraph, click "拆成句子，开始练" — sentences should appear
  and the four mode tabs should switch.

## Secrets the human must provide (do NOT commit these; not in the repo by design)

After the environment runs, prompt the user only if they want these features:

1. **Pronunciation scoring (shadowing mode)** — Azure Speech key. **Guide the user through this in
   the conversation, then write the key for them. There is no in-app settings UI by design.**
   - Tell them it's optional: without a key all four modes still work (shadowing falls back to a
     local Whisper word-match). Only the phoneme-level *precise* scoring needs Azure.
   - If they want it: have them create a free Azure Speech resource — Azure for Students (school
     email, no credit card) → Speech resource, tier **Free F0**, region **Southeast Asia** (East Asia
     is blocked for student subscriptions). They copy KEY 1 from "Keys and Endpoint".
   - Then YOU write it for them: key on line 1, region on line 2 of `azure_key.txt` at the repo root.
     To apply it to an already-running server without restart, POST it:
     `curl -X POST localhost:8000/api/config/azure -H 'Content-Type: application/json' -d '{"key":"<KEY>","region":"southeastasia"}'`
     (otherwise it loads on next start).
   - Confirm: `GET /api/health` returns `"azure":true`. Never print, log, or commit the key;
     `azure_key.txt` is git-ignored.
2. **Bilibili videos** — a logged-in `cookies.txt`. **Guide the user through this in the
   conversation, then place the file for them. Only Bilibili needs it — YouTube / X / most sites
   work without any cookie.**
   - Bilibili returns HTTP 412 for logged-out requests; a logged-in cookie fixes it.
   - Have the user install the "Get cookies.txt LOCALLY" extension **from the official Chrome Web
     Store / Edge Add-ons** (this extension reads all browser cookies — never tell them to download
     a loose `.crx`/`.zip` and sideload it). Then log in to bilibili.com and click Export — it
     downloads a `cookies.txt` (usually to their Downloads folder).
   - Then YOU move/copy that file to the repo root as `cookies.txt`. It's auto-detected and read
     fresh on every request — no restart. (Or point elsewhere with `YT_COOKIES_FILE=<path>`.)
   - If a Bilibili extract still 412s, the cookie expired — have them re-export, and you replace it.
   - Never print, log, or commit it (holds their login session); `cookies.txt` is git-ignored.

Full step-by-step for both is in [docs/SETUP.md](docs/SETUP.md).

## Gotchas / pinned versions (do NOT "upgrade" these blindly)

Some versions in `requirements.txt` are pinned because newer ones crash on some CPUs
(observed on AMD Ryzen 6000-series):

- `ctranslate2==4.4.0` — newer segfaults when loading the Whisper model.
- `numpy==1.26.4` (<2) — otherwise "Failed to initialize NumPy".
- Optional background-music removal needs `torch==2.2.2` + `torchaudio==2.2.2` (CPU wheels);
  newer torch fails to import (c10.dll). Install only if the user wants Demucs vocal separation.

If a pinned version genuinely won't install on the target platform, tell the user before changing it.

## Notes for the agent

- Never commit `azure_key.txt`, `cookies.txt`, `*.key`, `.env`, or the `media/` folder — they are
  git-ignored and may contain secrets or large files. A pre-commit hook blocks obvious key leaks.
- The repo ships no credentials. Do not invent or hardcode any.
- If running in a sandbox without a browser/microphone, you can still install deps and start the
  server; the user opens `localhost` themselves to actually use it.

---
> Source: [Oliviaviaviavia/english-trainer](https://github.com/Oliviaviaviavia/english-trainer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
