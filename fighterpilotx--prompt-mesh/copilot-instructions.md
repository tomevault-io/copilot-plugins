## prompt-mesh

> Pure vanilla HTML/CSS/JS. Zero build steps, zero npm, zero frameworks.

# PromptMesh — Claude Code Guide

## Architecture

Pure vanilla HTML/CSS/JS. Zero build steps, zero npm, zero frameworks.

| File | Product | Notes |
|------|---------|-------|
| `index.html` | Hub (Netlify entry) | Identical to hub; Netlify loads this by default |
| `promptmesh-hub.html` | Hub | Internal navigation target from sub-pages |
| `promptmesh-studio.html` | Studio | 5-stage AI pipeline; large file (~122 KB) |
| `promptmesh-enterprise.html` | Enterprise | Agent routing dashboard |
| `promptmesh-dev.html` | Dev Arena | Bot arena + ELO leaderboard |
| `promptmesh-sysai.html` | SysAI | PC optimizer; local agent integration |
| `agent/promptmesh_agent.py` | Local Agent | Flask on port 7842; real CPU/GPU/disk stats |

## Key Patterns

### API Calls (all pages)
Every `fetch` to `https://api.anthropic.com` **must** include this header or the call will fail with a CORS error:
```js
"anthropic-dangerous-request-when-unauthenticated": "true"
```

### API Key Storage
- Stored in `sessionStorage` only — never `localStorage`, never sent to any server
- Accessed via `sessionStorage.getItem('pm_key')`

### Dark Mode
All pages use `localStorage` key `pm_dark`:
- `'1'` = dark mode on
- `'0'` = light mode on

**Class convention differs by page:**
- Studio / Enterprise: add `body.dark` for dark mode
- Dev Arena / SysAI: add `body.light` for light mode (dark is CSS default)

### AI Model
Use `claude-haiku-4-5-20251001` for all calls. ~$0.004/pipeline run.

### CSP Headers
Each HTML file has its own `<meta http-equiv="Content-Security-Policy">`. When adding new external origins (fonts, APIs, etc.) update **both** the file's `<meta>` CSP and `netlify.toml` headers.

SysAI `connect-src` allows: `https://api.anthropic.com http://localhost:7842`

## Local Agent (SysAI)

The Python agent runs on `localhost:7842` and provides real hardware stats that browsers cannot expose.

```bash
pip install psutil wmi requests flask flask-cors
python agent/promptmesh_agent.py
```

Key endpoints:
- `GET /ping` — health check
- `GET /snapshot` — full system snapshot (used by Quick Scan)
- `GET /cpu` — CPU usage, frequency, temperatures
- `GET /memory` — RAM and swap
- `GET /disk` — partitions and I/O
- `GET /gpu` — GPU name, VRAM, driver version (WMI)
- `GET /processes` — top 15 by CPU
- `GET /startup` — startup programs
- `GET /drivers` — installed drivers
- `GET /eventlog` — recent error/warning events

SysAI `runQuickScan()` calls `/snapshot` with a 3-second timeout. If unreachable, it falls back to browser APIs and shows an "Agent Offline" card with setup instructions.

## Deployment

- Netlify auto-deploys on every push to `main`
- `netlify.toml`: `publish = "."` (serves files from repo root)
- `index.html` is the Netlify entry point

## Common Tasks

### Add a new page
1. Create `promptmesh-YOURPAGE.html` following the existing structure
2. Add a product card to both `index.html` and `promptmesh-hub.html`
3. Add the filename to the `FILES` JS object in hub/index
4. Update the compare table in hub/index
5. Update `README.md`

### Modify the AI system prompt
Each page has a `*_SYSTEM` constant near the top of the `<script>` block.

### Add a new PowerShell command to SysAI Command Library
Find the `COMMANDS` array in `promptmesh-sysai.html`. Each entry:
```js
{name: 'Display Name', risk: 'safe|moderate|advanced', desc: 'What it does', code: 'Get-Something | ...'}
```

### Debug API calls
Open browser DevTools → Network tab → filter by `api.anthropic.com`. Check that the `anthropic-dangerous-request-when-unauthenticated` header is present.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FighterPilotX)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/FighterPilotX)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
