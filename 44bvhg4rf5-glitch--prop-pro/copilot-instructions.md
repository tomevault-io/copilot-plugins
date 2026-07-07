## prop-pro

> PropMail Pro is an AI-powered estate agent intelligence platform for the Harrow (HA) postcode area.

# PropMail Pro — Claude Code Project

## What this is
PropMail Pro is an AI-powered estate agent intelligence platform for the Harrow (HA) postcode area.
It finds live property listings on Rightmove, extracts real addresses, and generates instruction letters.

## Running the project
```bash
node server.js
# Then open http://localhost:3000
```

## Project structure
```
propmail-pro/
├── server.js              # Node.js HTTP server (no dependencies)
├── package.json
├── CLAUDE.md              # This file
└── public/
    ├── index.html         # App shell (HTML panels only)
    ├── styles.css         # All CSS (~86KB)
    └── app.js             # All JavaScript (~220KB, 156 functions)
```

## Key features (11 panels)
1. **HA District Search** — Live property finder with one-click search
2. **Print Queue** — Letter queue with real addresses and Rightmove links
3. **Auto Flow** — 4-step automated pipeline
4. **Live Bot** — Continuous 24/7 monitoring
5. **AI Intel** — Owner research via 6 public sources
6. **Success Letters** — Postcode address lookup
7. **Templates** — Letter template editor
8. **Printers** — Network printer management
9. **Investor Board** — Revenue KPIs and scenarios
10. **AI Advisor** — Campaign health analysis
11. **Director Vision** — 15 strategic ideas

## Architecture
- Pure vanilla JS — no build step, no npm packages needed
- Calls `api.anthropic.com/v1/messages` directly from the browser
- Uses Claude web_search tool to find live Rightmove listings
- `app.js` is self-contained with all 156 functions

## Extending
To add a new panel: add HTML in `index.html`, add JS functions in `app.js`,
add a nav button, add the panel to `showPanel()` in app.js.

## API key
The frontend never holds the key. Each AI panel posts to the local
`/api/anthropic` endpoint, and `server.js` proxies it to
`api.anthropic.com/v1/messages`, injecting `ANTHROPIC_API_KEY` from the
environment server-side. Run with:
```bash
ANTHROPIC_API_KEY=sk-ant-... node server.js
```
If the key is missing the UI still loads; AI calls return a clear
"AI disabled" error instead of failing silently. Status is exposed at
`/api/config`.

---
> Source: [44bvhg4rf5-glitch/Prop-pro](https://github.com/44bvhg4rf5-glitch/Prop-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
