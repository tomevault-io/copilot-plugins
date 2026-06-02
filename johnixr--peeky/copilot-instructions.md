## peeky

> Guidance for AI agents (and humans) contributing to **peeky**. Read this before

# CLAUDE.md — working notes for Claude / agents

Guidance for AI agents (and humans) contributing to **peeky**. Read this before
making changes.

## What peeky is

A transparent floating **macOS desktop AI companion** built with **Tauri 2**:
a Rust backend (`src-tauri/`) + a **vanilla TypeScript + Vite** webview frontend
(`src/`). A draggable, decoration-less character that watches the screen via
periodic screenshots and speaks at the right moments, plus on-demand
screenshot-and-ask shortcuts. macOS only (Apple Silicon / Intel).

## Layout

- `src-tauri/src/` — Rust. Entry/wiring + global shortcuts + main loop: `lib.rs`.
  Modules: `capture` (screen capture + crop + JPEG), `trigger` (cheap pHash /
  scroll gate + front-app context), `api` (OpenAI-compatible streaming client),
  `modes` (system-prompt assembly), `agent`/`tools` (copilot tool-calling),
  `restraint`, `memory`, `settings`, `permission` (macOS Screen Recording),
  `commands` (the `#[tauri::command]` surface), `types` (the `Config` + shared
  serde types), `state`.
- `src-tauri/prompts/*.md` — system prompts (roast/nerd/copilot + quick_*).
- `src/` — frontend: `mascot.ts/.css` (the sprite + result bubble + ask box),
  `capture-main.ts` (the fullscreen region-selector overlay), `settings.ts`
  (settings window), `main.ts` (event glue), `markdown.ts`, `i18n/`.
- Three Vite HTML entries: `index.html` (overlay), `settings.html`, `capture.html`.

## Build / run / test

```sh
pnpm install
pnpm tauri dev        # dev (NOTE: debug build — image ops are much slower)
pnpm build            # frontend only: tsc --noEmit (strict) + vite build
cd src-tauri && cargo test     # Rust unit tests
cd src-tauri && cargo check    # keep this at ZERO warnings
```

**Release / signed build** (required for screen capture to actually work — see
below):

```sh
APPLE_SIGNING_IDENTITY="<your Apple Development identity>" pnpm tauri build
```

## Conventions (please keep these green)

- `cargo check`/`cargo test` must stay at **0 warnings, all tests passing**.
- `pnpm build` (strict `tsc`) must pass.
- **i18n parity**: `src/i18n/{en,zh,ja}.json` must have the **identical key set**.
  Add any new UI string to all three. Quick check:
  `node -e 'const e=require("./src/i18n/en.json"),z=require("./src/i18n/zh.json"),j=require("./src/i18n/ja.json");const k=o=>Object.keys(o).sort();const eq=(a,b)=>{a=k(a);b=k(b);return a.length===b.length&&a.every((x,i)=>x===b[i])};console.log(eq(e,z)&&eq(e,j))'`
- Model/system prompts live in `src-tauri/prompts/*.md` (embedded via
  `include_str!`), not inline in Rust.
- Prompt design: keep the **system prompt stable** (cache-friendly) and put
  per-call/variable content (context, user directive, image) at the **end** of
  the user message.

## macOS specifics (important)

- **Screen Recording permission** is required and is **finicky with ad-hoc
  signatures**: on macOS 15 an ad-hoc-signed build can be granted yet still
  capture an all-black frame. Build with a real **Apple Development** signature
  (stable designated requirement) so the grant sticks across rebuilds. Capture
  goes through in-process `xcap` first, falling back to the `screencapture` CLI.
- Accessibility permission is needed for `osascript` front-app context + copilot
  input tools (`enigo`).

## Security red-lines (do NOT cross)

- **Never hardcode API keys.** Keys come from settings or the `PEEKY_API_KEY`
  env; an env-supplied key is never written to disk.
- **Never disable TLS verification** (no `danger_accept_invalid_certs`).
- **Never commit secrets or PII** — no keys, no private endpoints/IPs, no signing
  identity (pass it via the `APPLE_SIGNING_IDENTITY` env at build time, not in
  `tauri.conf.json`). The internal product spec is kept out of this repo.
- Copilot/agent actions honor a hard-forbidden list (delete files, quit/close
  apps, shutdown/restart, system settings, sudo, password managers, install/
  uninstall, browser data). These are blocked, never executed.

`AGENTS.md` is a symlink to this file.

---
> Source: [Johnixr/peeky](https://github.com/Johnixr/peeky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
