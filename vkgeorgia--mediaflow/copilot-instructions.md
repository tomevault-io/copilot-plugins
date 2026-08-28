## mediaflow

> Instructions for an AI coding agent working in this repository — Claude Code, Cursor, Codex, Copilot, or any other. Human contributors: read [CONTRIBUTING.md](CONTRIBUTING.md) instead. Read this file before your first commit.

# AGENTS.md

Instructions for an AI coding agent working in this repository — Claude Code, Cursor, Codex, Copilot, or any other. Human contributors: read [CONTRIBUTING.md](CONTRIBUTING.md) instead. Read this file before your first commit.

## Commit identity — set it in this clone, first thing

A token decides whether a push is *accepted*. `user.email` decides whose name is on the *work*. They are unrelated: pointing a tool at a different account changes nothing about authorship, so the identity has to be set explicitly, in this clone, before anything is committed.

```bash
git config user.name  "automation"
git config user.email "automation@korobeinikov.consulting"
```

- **An agent commits as `automation <automation@korobeinikov.consulting>`.** Never under the maintainer's name: the history has to keep saying which commits a person wrote.
- **The maintainer commits as `Valerii Korobeinikov <valerii@korobeinikov.consulting>`.**
- **No third identity belongs in this history.** Before you push, run `git log -1 --pretty='%an <%ae>'`. If it shows anything other than the two above, fix the clone's config and amend — do not push and sort it out later. Authorship is only cheap to fix before it is published.
- Set it **per clone**. A global identity is whatever the machine happened to be configured for, which is exactly how a wrong one gets inherited silently.

## What this repository is

MediaFlow — a desktop app that copies, sorts and deduplicates photos and videos and edits EXIF and file dates. FastAPI backend (`app.py`), single-page UI (`static/index.html`), Tauri 2 desktop shell (`src-tauri/`). Default branch is `main`.

## House rules

- **Everything committed is in English** — code, comments, docs, commit messages, PR titles and bodies. The UI is the one exception: it ships `ru` and `en` strings in `static/index.html`, and both must be updated together.
- **No build output in git.** Installers, portable packs and any `.zip` are published as GitHub Release assets; the repository carries source only.
- **`CLAUDE.md` is local and gitignored** — never commit it. Repository-wide agent instructions belong in this file.
- **Never commit** `settings.json`, `.env`, `.venv/`, `node_modules/`, or anything under `src-tauri/target/`.
- Keep platform-specific code behind explicit OS checks. Windows is the packaged platform today; macOS is planned, so do not write Windows-only assumptions into shared paths.
- Keep changes small and focused, and preserve existing copy / sort / dedup behaviour unless the change to it is the point.

## Before you finish

- State how you tested — browser, Tauri shell, or both — and on which OS.
- Update `README.md` when you change how the app is downloaded, run or built.
- Bump the version in `package.json`, `src-tauri/Cargo.toml` and `src-tauri/tauri.conf.json` together; they must not drift apart.

---
> Source: [vkgeorgia/mediaflow](https://github.com/vkgeorgia/mediaflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
