## mnemo

> A single-page flashcard study tool (Mnemo) with a Python Flask backend.

# Mnemo Flashcards — Project Guide

## Overview
A single-page flashcard study tool (Mnemo) with a Python Flask backend.
The frontend is `index.html`; the server is `server.py`.

## How to Run

**Windows** — double-click `start.bat`
**macOS** — double-click `start.command` (first run: `chmod +x start.command` in Terminal)

Or manually from the project folder:
```bash
ollama serve        # terminal 1
python server.py    # terminal 2  (use python3 on macOS)
# then open http://localhost:5000
```

`start.bat` uses PowerShell so that `ollama` and `python` are found in PATH (CMD lacks them).
`start.command` detects `python3`/`python`, skips Ollama if already running, and kills both on window close.

## File Structure
```
Printable Flashcards/
├── index.html     # Frontend — all HTML/CSS/JS in one file
├── server.py           # Flask backend (APIs + file serving)
├── start.bat           # One-click launcher — Windows
├── start.command       # One-click launcher — macOS (chmod +x first)
├── requirements.txt    # pip install -r requirements.txt
├── backups/            # Auto-created — JSON backups saved here
├── pdfs/               # Auto-created — PDFs per set: pdfs/<setId>/
├── tts/                # Auto-created — cached TTS WAV files: tts/<setId>/<pdfStem>/<n>.wav
├── manifest.json       # PWA manifest
└── study_tools_sw.js   # Service worker (offline support)
```

## Architecture

### Frontend (`index.html`)
- Pure vanilla JS, no build step, no frameworks
- Data stored in `localStorage` key `mnemo_data_v2`
- Served by Flask at `/` when running with `server.py`
- Can also be opened directly as a file (PWA mode — no server features)
- `activeView` controls top-level view: `'set'` (default) or `'calendar'`

### Backend (`server.py`)
Flask app with these API routes:

| Route | Method | Purpose |
|-------|--------|---------|
| `/` | GET | Serve index.html |
| `/api/backup/save` | POST | Save JSON to `backups/` folder |
| `/api/backup/list` | GET | List all backups (newest first) |
| `/api/backup/load/<filename>` | GET | Return backup JSON |
| `/api/ai/models` | GET | List available Ollama models |
| `/api/ai/chat` | POST | Stream Ollama chat response (SSE) |
| `/api/pdfs/<setId>/upload` | POST | Upload PDF for a set |
| `/api/pdfs/<setId>/list` | GET | List PDFs for a set |
| `/api/pdfs/<setId>/<file>` | GET | Serve a PDF file |
| `/api/pdfs/<setId>/<file>` | DELETE | Delete a PDF |
| `/api/pdfs/<setId>/extract/<file>` | GET | Extract text from PDF (pdfplumber) |
| `/api/tts` | POST | One-shot TTS: text → WAV (Kokoro) |
| `/api/tts/prepare` | POST | Split PDF into sentences, cache `sentences.json` |
| `/api/tts/audio/<setId>/<pdfStem>/<n>` | GET | Serve (and generate if missing) nth sentence WAV |

### Data Flow
- **Export**: JS POSTs `db` JSON to `/api/backup/save` → saved to `backups/`
- **Load Backup**: JS fetches `/api/backup/list`, shows modal, loads selected file
- **Auto-restore**: On page load with empty localStorage, auto-loads the latest backup
- **AI Chat**: Streams from Ollama via SSE; system prompt = "I am a high school student studying for {set name}…"; `@Term` mentions in user messages are expanded to `Term is Definition.` before sending
- **TTS pipeline**: `/api/tts/prepare` extracts + splits sentences → cached in `tts/<setId>/<pdfStem>/sentences.json`; each sentence WAV is generated on first request and cached as `1.wav`, `2.wav`, … so repeat plays are instant

## Key JS State Variables
- `db` — global database object (mirrors localStorage)
- `sd()` — saves `db` to localStorage
- `activeSetId` — currently selected set id (null = none)
- `activeTab` — current tab: `'terms' | 'flashcards' | 'progress' | 'ai' | 'sources'`
- `activeView` — top-level view: `'set'` or `'calendar'`
- `aiChats[setId]` — in-memory chat history per set (not persisted)
- `aiModel` — currently selected Ollama model string
- `ttsState` — full TTS player state (see below)
- `_mentionAnchor`, `_mentionTerms`, `_mentionSelIdx` — @mention autocomplete state

## Key JS Functions
- `renderSetView()` — rebuilds the whole set panel (tabs + content)
- `renderTabContent()` — renders the active tab's HTML into `#tabContent`
- `switchTab(t)` — changes `activeTab`, stops TTS if leaving sources, re-renders
- `showGlobalCalendar()` — switches to the global calendar view
- `refreshGlobalCalendar()` — re-renders calendar in place (called after add/delete events)
- `sendAiMessage(setId, set)` — sends message to Ollama with @mention expansion
- `expandMentions(text, set)` — replaces `@Term` with `Term is Definition.`
- `startTts() / stopTts() / pauseResumeTts() / ttsSeek(delta)` — TTS pipeline controls

## Tabs (per set)
`Terms | Flashcards | Progress | 🤖 AI | 📚 Sources`

Calendar was removed from per-set tabs — it is now a **global view** accessible from the sidebar.

## AI Tab
- System prompt: `"I am a student in high school studying for {set.name}, you are a master study assistant…"`
- No term dump in system prompt — terms are injected inline via `@mentions`
- Typing `@` in the input shows **all** terms in a scrollable dropdown immediately
- Typing more characters narrows down the list (starts-with ranked above contains, cap 12)
- Arrow keys navigate, Enter/Tab selects, Escape closes
- Selected mention appears as `@Term` (gold pill) in the chat bubble
- What gets sent to Ollama: `@Armistice` → `Armistice is An agreement to stop fighting; ended WWI on November 11, 1918.`

## Global Calendar
- Accessed via "📅 Calendar" in the sidebar under "Views"
- Shows ALL sets' goals, study sessions, and events on one calendar
- Study sessions display as "Study Session — {Set Name}"
- Goals display as "Exam Goal — {Set Name}"
- User-created events are global (no set association)
- `db.calEvents` entries no longer require a `setId` field

## TTS Sentence Pipeline
- "Read" button on a PDF calls `/api/tts/prepare` → server splits text into sentences
- Sentences cached in `tts/<setId>/<pdfStem>/sentences.json`
- Each sentence WAV cached as `1.wav`, `2.wav`, … generated on first request
- While sentence N plays, sentence N+1 is pre-loaded (`ttsState.preAudio`)
- Player controls (◀ ⏸ ▶ ■) appear inside the PDF viewer bar
- Auto-pauses when the PDF viewer is closed or the user switches tabs
- `ttsState` object: `{ active, setId, filename, stem, sentences, current, paused, audio, preAudio, preN }`

## CSS Variables (theme)
```css
--bg: #0f0e0c          /* near-black background */
--surface: #1a1916     /* card surfaces */
--surface2: #222119    /* slightly lighter surface */
--accent: #d4a843      /* gold — primary accent */
--accent-dim: #8a6c27  /* darker gold */
--text: #f0ece2        /* off-white body text */
--text-dim: #7a7669    /* muted text */
--red / --green / --yellow / --blue  /* status colors */
```

## Adding a New Tab
1. Add a `.tab` div in `renderSetView()` (search "Sources" tab for placement reference)
2. Add `else if(activeTab==='newtab'){el.innerHTML=renderNewtab(set);bindNewtab(set);}` in `renderTabContent()`
3. Write `renderNewtab(set)` returning an HTML string
4. Write `bindNewtab(set)` for any event listeners needed after render
5. Call `stopTts()` in `switchTab` if the new tab should pause audio

## Dependencies
```bash
pip install -r requirements.txt
```
Also requires:
- [Ollama](https://ollama.ai) installed — run `ollama pull llama3` once
- [espeak-ng](https://github.com/espeak-ng/espeak-ng/releases) — required by Kokoro TTS

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NotBraker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
