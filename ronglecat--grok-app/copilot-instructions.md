## grok-app

> 1. **`docs/llm-wiki/`** — product rules for agents (i18n, Grok Build catalog).

# Agent notes — Grok App

## Read first

1. **`docs/llm-wiki/`** — product rules for agents (i18n, Grok Build catalog).  
   - [i18n.md](docs/llm-wiki/i18n.md) — all UI strings via `src/i18n/`  
   - [dialogs.md](docs/llm-wiki/dialogs.md) — **no `window.confirm` / `prompt` / `alert`**; use in-app dialogs  
   - [catalog.md](docs/llm-wiki/catalog.md) — models / effort / YOLO  
   - [automations.md](docs/llm-wiki/automations.md) — automation design (Build `/loop` / scheduler; non-blocking)  
   - [account.md](docs/llm-wiki/account.md) — official login, membership, quota, heatmap  
   - [providers.md](docs/llm-wiki/providers.md) — custom relays, agent `GROK_HOME`, editors  
   - [setup.md](docs/llm-wiki/setup.md) — first-run gate (CLI required, account optional)  
   - [icons.md](docs/llm-wiki/icons.md) — app dock icons vs tray/status-bar icons (never mix)

1b. **Release (AI handoff)** — **[docs/llm-wiki/release.md](docs/llm-wiki/release.md)** is the single source for ship steps. Platforms / local build: [docs/BUILD.md](docs/BUILD.md). Window chrome: `tauri.macos.conf.json` (Overlay) vs `tauri.windows.conf.json` (frameless).  
   - Never tag without `## [X.Y.Z]` in `CHANGELOG.md`.  
   - GitHub Release body = `scripts/changelog-for-release.py` (must include update list + macOS “damaged app” / `xattr`).  
   - Do not hand-edit Release notes only on GitHub; change the script + CHANGELOG.

1c. **Open-source surface** — public docs: `README.md` / `README_EN.md`, `LICENSE`, `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, `CHANGELOG.md`. Do not commit secrets, `auth.json`, or local agent homes.

2. Do **not** hardcode user-facing English/Chinese. Use `createT(locale)` / `t()`.

2b. **Dialogs** — never use `window.confirm` / `window.prompt` / `window.alert` in Tauri UI. Use App `setAppDialog`, `GlassModal`, or the same in-app portal + glass CSS. All floating panels share frosted glass tokens. Details: [docs/llm-wiki/dialogs.md](docs/llm-wiki/dialogs.md).

3. When adding models or permission modes, update `src/lib/grokCatalog.ts` **and** `docs/llm-wiki/catalog.md`.

3b. Custom providers write `~/.grok-app/agent-home/config.toml` and spawn agent with `GROK_HOME` (independent mode). Do not leave relay keys only in App secrets.

4. Prefer real Grok Build CLI behavior (`grok models`, `--always-approve`, `--effort`).

5. Assistant messages: render markdown (`MarkdownBody`); user messages: gray bubble, no role labels.

---
> Source: [RongleCat/grok-app](https://github.com/RongleCat/grok-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
