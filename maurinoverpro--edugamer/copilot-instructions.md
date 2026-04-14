## edugamer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**EduGamer OS** is an educational web platform for middle school students with learning disabilities (DSA - Disturbi Specifici dell'Apprendimento). It is a fully client-side static web application written in Italian with no build step.

## Development

**No build system.** Edit HTML/JS files directly and open in a browser. All dependencies are loaded from CDNs at runtime.

To test: open `index.html` in a browser. A Google Gemini API key must be entered in the settings modal for AI features to work. The key is stored in `localStorage` under `gemini_api_key`.

## Architecture

### Hub-and-Spoke Model

`index.html` is the home/launcher page. Each module is a self-contained React SPA in its own HTML file:

- `lavagna.html` — AI-powered mind mapping from image uploads (canvas-based, 3000×3000px, jsPDF export)
- `matematica.html` — Visual mathematics with place-value blocks for dyscalculia support
- `italiano.html` — Grammar/writing correction with AI error categorization
- `tutor.html` — AI chat tutor with speech recognition/TTS and quiz generation
- `profilo.html` — User dashboard: XP, level, achievements, streak
- `discover.html` — DISCOVER v4.0 guided problem-solving method
- `risolvitore.html` — Multi-type problem solver

### Gamification Engine (`game-system.js`)

Shared singleton loaded in every module via `<script src="game-system.js">`. Exposes `window.EduGamer` with:

- `addXP(amount, source, action)` — awards XP, checks achievements, updates streak
- 10 level tiers (Principiante → Immortale), XP thresholds: 0, 100, 300, 600, 1000, 1500, 2500, 4000, 6000, 10000
- 22 achievements across 4 rarity tiers (Comune, Raro, Epico, Leggendario)
- Fires custom events: `edugamer-xp-added`, `xpChanged`, `edugamer-stats-updated`

### localStorage Keys

| Key | Purpose |
|-----|---------|
| `edugamer_stats` | XP, streak, per-module action counts |
| `edugamer_achievements` | Unlocked achievement IDs |
| `gemini_api_key` | Gemini API key entered by user |
| `edu_xp` | Legacy XP key (kept for backwards compatibility) |

### AI Integration

All modules call the Google Gemini API directly from the browser:
- Model: `gemini-3-flash-preview` (gemini-2.5-flash was deprecated in March 2026)
- Endpoint: `https://generativelanguage.googleapis.com/v1beta/models/${MODEL}:generateContent?key=${key}`
- The key is retrieved from localStorage before each call

### Tech Stack

| Layer | Technology |
|-------|-----------|
| UI Framework | React 18 (UMD via CDN `unpkg.com`) |
| JSX Transpilation | Babel Standalone (CDN, in-browser) |
| Styling | Tailwind CSS 3 (CDN) |
| Font | OpenDyslexic (accessibility) |
| PDF Export | jsPDF 2.5.1 |
| Markdown | marked.js |
| AI | Google Gemini 3 Flash Preview API |

### Pitfalls to avoid

- **`game-system.js` must be loaded at the bottom of `<body>`**, after the `<script type="text/babel">` block. Loading it in `<head>` causes it to run before React initializes.
- **Never use direct localStorage XP writes** (e.g. `localStorage.setItem('edu_xp', ...)`). Always call `window.EduGamer.addXP(amount, source, action)` so achievements and streaks are triggered correctly.

### Accessibility Conventions

All modules use OpenDyslexic font, `letter-spacing: 0.02–0.08em`, `line-height: 1.6–1.8`, dark theme (`#0a0e17` background), and minimum 56px touch targets. Maintain these when editing UI.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MaurinOverpro)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MaurinOverpro)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
