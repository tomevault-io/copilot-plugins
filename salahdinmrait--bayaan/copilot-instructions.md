## bayaan

> Guidance for Claude Code when working in this repository.

# CLAUDE.md — Bayaan

Guidance for Claude Code when working in this repository.

## Project Overview

**Bayaan** (بيان) is a browser-based audio transcription and translation app that extends the Al-Bayaan Vertalingen service (`albayaanvertalingen.nl`). It transcribes audio memos with Groq Whisper and translates the result with Groq LLaMA, supporting two directions:

- Arabic → Dutch (`ar-to-nl`)
- Dutch → Arabic (`nl-to-ar`)

The design system (colours, fonts, spacing tokens, dark/light theme) is intentionally identical to the parent Vertaler project so both tools feel like one product family.

## Tech Stack

| Layer | Choice |
|-------|--------|
| Frontend | Vanilla HTML5 + CSS3 + JavaScript (no framework) |
| Transcription | Groq `whisper-large-v3` (same model as ChatGPT, excellent Arabic support) |
| Translation | Groq `llama-3.3-70b-versatile` |
| Backend | Vercel Serverless Functions (`api/`) — Node.js 20 CJS |
| Multipart parsing | `formidable` v3 |
| Fonts | Poppins, Lora, Kufam, Mirza (local TTF, same as Vertaler) |
| Hosting | Vercel (auto-deploy on push to main via GitHub) |

## Local Development

```bash
# Install dependencies
npm install

# Copy .env.example → .env and fill in your Groq key
cp .env.example .env

# Start local dev server
npm run dev
```

Then open `http://localhost:3000`.

> The API routes (`/api/*`) are handled by `dev-server.js`. For a full Vercel-compatible local environment use `npx vercel dev` after logging in with `npx vercel login`.

## Project Structure

```
/
├── index.html            # Single-page app
├── css/
│   ├── fonts.css         # @font-face declarations (Poppins, Lora, Kufam, Mirza)
│   └── style.css         # All styles — design tokens + Bayaan-specific components
├── js/
│   └── main.js           # App logic (theme, recording, upload, process flow, results)
├── api/
│   ├── transcribe.js     # POST — receives audio, calls Groq Whisper, returns transcription
│   └── translate.js      # POST — receives text + direction, calls Groq LLaMA
├── dev-server.js         # Local dev server (not used by Vercel)
├── Poppins/ Lora/ Kufam/ Mirza/  # Local font TTF files
├── vercel.json           # Framework=null static site + function timeout config
├── package.json          # formidable + dotenv dependencies
├── .env.example          # Required env vars
└── CLAUDE.md             # This file
```

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `GROQ_API_KEY` | Used by both `/api/transcribe` and `/api/translate`. Free at console.groq.com |

Set in Vercel dashboard → Project → Settings → Environment Variables for production.

## API Endpoints

### `POST /api/transcribe`
- **Body**: `multipart/form-data` with fields `audio` (file) + `direction` (`ar-to-nl` | `nl-to-ar`)
- **Returns**: `{ text: string }` — the Whisper transcription
- **Model**: `whisper-large-v3` via Groq, language hint set from direction
- **Limit**: 25 MB file, 60 s function timeout
- **Note**: m4a MIME types are normalised to `audio/mp4` before sending to Groq

### `POST /api/translate`
- **Body**: `{ text: string, direction: string }`
- **Returns**: `{ text: string }` — the LLaMA translation
- **Model**: `llama-3.3-70b-versatile` via Groq, temperature 0.2
- **Limit**: 30 s function timeout

## Design System

Inherited from **Vertaler** (`albayaanvertalingen.nl`). CSS variables are in `css/style.css` under `:root` and `[data-theme="dark"]`. Do not change the core tokens — they must stay in sync with the parent project's feel.

Key tokens:
- Accent: `#d4a800` (light) / `#FFD700` (dark)
- Background: `#ffffff` (light) / `#000000` (dark)
- Card: `#ffffff` (light) / `#1a1a1a` (dark)
- Border: `#e5e7eb` (light) / `#2a2a2a` (dark)
- Fonts: Poppins (headings/UI), Lora (body), Kufam (Arabic headings), Mirza (Arabic body)

Theme stored in `localStorage` under key `theme` (`'light'` | `'dark'`).

## Working Conventions

### Pre-Commit Checks
Check that no workflow file triggers an unnecessary full build before committing.

### Parallel Agents
Partition by file boundary — no two agents should edit the same file simultaneously.

### End-to-End Verification
After touching the API routes or the process flow in `main.js`, verify the golden path manually:
1. Upload a short Arabic m4a file
2. Click "Transcribe & Translate"
3. Confirm transcription and translation both appear in the result cards

### Deployment
Push to `main` — Vercel auto-deploys via GitHub integration. Check the Vercel dashboard for function logs if the API returns errors.

## Known Constraints

- **File size limit**: 25 MB. Groq Whisper accepts: `mp3`, `mp4`, `m4a`, `wav`, `ogg`, `webm`, `flac`.
- **Multipart construction**: Uses `Buffer.concat` to build the multipart body manually. Do NOT use string join — binary audio data must not be string-coerced.
- **Arabic text direction**: Result text areas use `dir` attribute set dynamically in `main.js` based on direction.
- **Groq free tier**: Very generous — no credit card needed. Rate limits are high enough for normal personal use.

---
> Source: [salahdinmrait/Bayaan](https://github.com/salahdinmrait/Bayaan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
