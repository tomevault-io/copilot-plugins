## exodus-wallet-injector

> Guidance for Claude Code working in this repo. Read `HANDOFF.md` too for the

# CLAUDE.md

Guidance for Claude Code working in this repo. Read `HANDOFF.md` too for the
full picture.

## What this is

An injector for the real Exodus desktop wallet (Electron). It overlays fake
balances and transactions on the running app for larp/demo use. No real funds,
no keys, no blockchain. Everything is faked in the renderer.

## Pieces

- `inject.js` — the engine injected into the wallet renderer. Wraps
  `store.getState` with ONE memoized overlay for fake balances, native `txLog`
  Activity, and swap progress. Also hooks the native Send and Swap buttons and
  fires the real notification toasts. Control API is `window.__wallet.*`.
- `controller/controller.py` — Python `rich` console that drives the wallet over
  CDP (port 9223) via `window.__wallet.*`. `run.sh` / `run.bat` set up a venv and
  launch it.
- `patcher/patch.js` — cross-platform patcher. Unpacks `app.asar`, prepends a
  shim to `main/index.js` (opens debug port 9223, injects `inject.js`), exposes
  the webpack require as `window.__wpr`, and renames `app.asar` so the unpacked
  `app/` loads. `install-windows.ps1` / `install-mac.sh` wrap it.

## How to work on it

- Exodus must be fully closed before patching (a running app locks `app.asar`;
  on Windows it also lingers in the tray).
- To test `inject.js` changes without restarting Exodus, hot-inject over CDP into
  the live `exodus-prod` renderer on port 9223: neutralize the old copy
  (`window.__demoInstalled=false; window.__demoV6Installed=false;`) then eval the
  new file. See the testing note in `HANDOFF.md`.
- Syntax check before committing: `node -c inject.js`, `node -c patcher/patch.js`,
  and `python3 -c "import ast; ast.parse(open('controller/controller.py').read())"`.

## Gotchas

- Keep a single `store.getState` overlay. Stacking a second wrapper breaks tx
  details and makes "Start new swap" stick.
- The Send screen fires its own "Sent!" modal on promise resolve; the Send hook
  should just resolve once and not also poke `state.txSend`.
- The swap progress screen crashes on null amounts — pass the real
  `fromAmount`/`toAmount` NumberUnits.

## Conventions

- Commits: simple and short, lowercase, no dashes, russian or english. No AI
  attribution unless asked (this repo does add a `Co-Authored-By: Claude` trailer
  by request).
- Author for commits here: `nakanune <woakxow@gmail.com>`.
- Comments: short, lowercase, only where non-obvious.

---
> Source: [mullvadov/exodus-wallet-injector](https://github.com/mullvadov/exodus-wallet-injector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
