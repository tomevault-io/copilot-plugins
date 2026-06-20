## gitpulse

> This repository is now a Tauri desktop application built with React and Rust.

# AGENTS.md

This repository is now a Tauri desktop application built with React and Rust.

## Project Overview

GitPulse is a local-first Git report generator. It scans local Git workspaces, extracts commits by author and date range, and generates Markdown work reports, including monthly performance reports grouped by project.

The legacy Python/Tkinter version is preserved on:

```bash
codex/legacy-python-desktop
```

## Key Commands

```bash
npm install
npm run dev
npm run build
npm run tauri dev
npm run tauri build
```

Rust verification:

```bash
cd src-tauri
cargo check
cargo test
```

## Architecture

- `src/`: React frontend
- `src/App.tsx`: main workbench UI
- `src/styles/`: application styling
- `src-tauri/src/lib.rs`: Tauri command registration and orchestration
- `src-tauri/src/git_ops.rs`: local Git scanning and commit extraction
- `src-tauri/src/report.rs`: summary and monthly report generation
- `src-tauri/src/ai.rs`: optional OpenAI-compatible and Anthropic Native report polishing
- `src-tauri/src/models.rs`: shared Rust command models

## Notes

- Do not reintroduce Python runtime dependencies on `main`.
- Keep local filesystem and Git operations in Rust commands.
- Keep API keys out of plain persisted config; use OS-backed secure storage or environment variables.
- Prefer product-grade local desktop UX over a browser-only web app.

---
> Source: [GoldenZqqq/GitPulse](https://github.com/GoldenZqqq/GitPulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
