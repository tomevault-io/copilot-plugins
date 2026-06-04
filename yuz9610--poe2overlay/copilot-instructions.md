## architecture

> Project architecture, IPC, and dev workflow — read before coding


# Architecture (PoE2 Area Overlay)

**Read [`docs/README.md`](docs/README.md#目標信息架構) for docs ownership** and [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for full technical context. Summary:

Workflow **canonical rules**: [`docs/PROJECT_WORKFLOW.md`](docs/PROJECT_WORKFLOW.md). Gate summary: [`.cursor/rules/workflow.mdc`](workflow.mdc). Prompts: [`docs/AI_AGENT_PROMPTS.md`](docs/AI_AGENT_PROMPTS.md). Human cheat sheet: [`docs/AI_AGENT_SETUP.md`](docs/AI_AGENT_SETUP.md).

Workflow short commands: follow matching step in `AI_AGENT_PROMPTS.md`; **first response = proposal only** unless skip phrase in same message (see `workflow.mdc`).

Docs architecture: every `docs/**/*.md` should begin with `文檔目的` / `不負責`. Keep one source of truth per knowledge type: architecture/IPC → `ARCHITECTURE.md`; user-facing behavior → `USER_GUIDE.md`; build schema → `BUILD_JSON.md`; build generation workflow → `CREATE_BUILD.md`; workflow rules → `PROJECT_WORKFLOW.md`; prompts → `AI_AGENT_PROMPTS.md`; agent maintenance → `AI_AGENT_SETUP.md`; current state → `BRAINSTORM.md` / `SPRINTS.md` / `docs/sprints/`.

## Stack

Tauri 2 + Rust (`src-tauri/`) · React 19 + TS + Vite 6 (`src/`) · JSON data (`data/`). Local-only; no HTTP/shell in Rust.

## Where things live

- IPC: `src/api/tauri.ts` ↔ commands in `src-tauri/src/lib.rs`
- App state: `src/app/useOverlayData.ts` (single source for settings, area, build, profiles, reminders)
- Log pipeline: `watcher.rs` → `log_parser.rs` → emit `area-changed` / `character-changed`
- Hints: `area_hints.rs` + `data/areas.json` (+ optional `data/build/` override)
- Types: `src/types.ts`
- Build JSON: `docs/BUILD_JSON.md`（schema）、`docs/CREATE_BUILD.md`（AI 生成 BD）
- Project workflow/state: rules in `docs/PROJECT_WORKFLOW.md`; prompts in `docs/AI_AGENT_PROMPTS.md`; current state in `docs/SPRINTS.md`, `docs/BRAINSTORM.md`, `docs/sprints/`

## Agent rules

1. Encoding: UTF-8 no BOM, LF — see `docs/ENCODING.md` and `encoding.mdc`
2. New IPC: update Rust command + `tauri.ts` + `types.ts` if needed
3. Avoid duplicate `listen` on same Tauri event; pass static config via props from `useOverlayData`
4. Background locks: use `sync_util::{mutex_lock,rw_read,rw_write}` in Rust worker threads
5. Chinese area names in JSON must match `Client.txt` bytes exactly
6. **Docs sync**: feature/behavior changes must update matching docs — see `docs-sync.mdc` and ARCHITECTURE § 文檔同步
7. **Workflow sync**: task/sprint/commit flow — follow `docs/PROJECT_WORKFLOW.md` and **`.cursor/rules/workflow.mdc`** (proposal before writeback; Step 4 two phases)
8. **Docs ownership**: do not duplicate long rules across docs; update the SOT named in `docs/README.md` and keep this rule / AGENTS as summaries only

## Verify after changes

```pwsh
npm.cmd run check:encoding
npm.cmd run build
# if Rust changed:
cd src-tauri; cargo clippy -- -D warnings
```

Also see [`AGENTS.md`](../../AGENTS.md) (Codex entry point).

---
> Source: [yuz9610/POE2Overlay](https://github.com/yuz9610/POE2Overlay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
