## cc-web-powerpack

> Userscripts (Tampermonkey) that improve **Claude Code Web** (`claude.ai/code`).

# CLAUDE.md — cc-web-powerpack

Userscripts (Tampermonkey) that improve **Claude Code Web** (`claude.ai/code`).
Public repo: `brunompicinini/cc-web-powerpack`, default branch `main`.

## Workflow (publishing an update)
- **Edit the canonical local clone**, not stray copies: `~/Documents/GitHub/cc-web-powerpack` (the old "Web Empire" folder is retired).
- Edit the `.user.js` → **bump `@version`** → `git commit` + `git push` on `main`.
- Tampermonkey pulls from `raw.../main/...` on auto-update (checks once a day). To test right away: reinstall from the raw URL with a cache-buster `?cb=N`.

## Structure
- `scripts/*.user.js` — one userscript per file (filename ends in `.user.js`).
- `README.md` — installation (Tampermonkey + the Chrome 138+ "Allow User Scripts" toggle) and auto-update.
- `CLAUDE.md` — this file.

## Conventions
- **Comentários curtos, estilo Caveman:** direto, simples, **máximo 2 linhas** por comentário. O conhecimento (DOM facts, gotchas) mora em comentário curto **perto do código**, onde é olhado - **não** centralizado aqui no CLAUDE.md. Narrativa de versão e deduções longas não se anota (git guarda).
- Every script has a `// ==UserScript== ... ==/UserScript==` header.
- `@match https://claude.ai/code*`, `@run-at document-start`. `@grant`: favicon and shortcuts use `none`; **the notepad uses `GM_openInTab`** (to reliably open a link in a background tab — see below). With `@grant` != none the script runs in the Tampermonkey sandbox (DOM/`history`/`location`/`localStorage` are still real; it just can't see the page's JS vars).
- **Auto-update:** `@downloadURL` and `@updateURL` point at the raw URL on `main`:
  `https://raw.githubusercontent.com/brunompicinini/cc-web-powerpack/main/scripts/<file>.user.js`
- **Every published change requires bumping `@version`** — that's Tampermonkey's auto-update trigger.
- **Do NOT change `@name` between versions.** Tampermonkey uses `@name` (+`@namespace`) as the script's **identity**. If `@name` changes, a manual reinstall from the raw URL shows **"Install"** (a new/duplicate script) instead of **"Update"** — and the old one stays installed and keeps running alongside. Background auto-update updates in place even with a new name, but the documented "test right away" flow (reinstall from raw) breaks. So: new features go only in `@description` (free to change, TM doesn't match on it), **never** in `@name`. If you must change `@name`, accept that it becomes a reinstall + deleting the old one by hand.
- Filename and the tail of the raw URLs must match (otherwise auto-update breaks).
- Watch out for invisible characters used on purpose in regexes (zero-width `​-‍﻿` in the favicon; the notepad normalizes nbsp via ` `). Don't "clean them up" by accident.

## The scripts (DOM facts + gotchas live inline in each file)

Every DOM fact (selectors, aria-labels, roles) and gotcha is a short comment **inside the `.user.js`**, next to the code it explains - read the file. Here just the high-level purpose + the cross-file contract:

- **session-status-favicon.user.js** - recolors Claude's real favicon by the open session's live status (running/awaiting/ready) or, when there's none, its PR badge (merged=purple / open=teal), and swaps the tab title for the session name.
- **session-notepad.user.js** - per-session notes side panel: a floating card styled like the app's native panels, resizable, auto-open, links open in a background tab. `@grant GM_openInTab`. Injects the `[data-cc-notes-btn]` action-bar button.
- **session-shortcuts.user.js** - every keyboard shortcut in one capture-phase `keydown`. Modifier `Mod` = Ctrl on Mac / Alt on Win (plus plain `Ctrl+\` for the sidebar). Full binding list: the `@description` + the switch statement.
- **Cross-file contract:** the shortcuts script toggles the notepad by clicking `[data-cc-notes-btn]` (the button the notepad injects). Pure DOM, no cross-sandbox event.

## Testing/linting
- Not a Node project; no build. Manual editing of the `.user.js`.
- Quick syntax check: `node --check scripts/<file>.user.js`.
- Optional lint with ESLint (flat config), formatting rules + `no-undef` with browser globals. Globals used: `window, document, location, navigator, localStorage, history, setTimeout, setInterval, clearTimeout, MutationObserver, Image, Intl, addEventListener, Promise`. The notepad also uses `GM_openInTab` and `GM_info` (Tampermonkey).

---
> Source: [brunompicinini/cc-web-powerpack](https://github.com/brunompicinini/cc-web-powerpack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
