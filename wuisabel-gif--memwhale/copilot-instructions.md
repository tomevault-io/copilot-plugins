## memwhale

> Guidance for AI coding agents (Codex, Claude, etc.) working in this repo.

# AGENTS.md

Guidance for AI coding agents (Codex, Claude, etc.) working in this repo.

## Use MemoryWhale as durable terminal memory

This project *is* a terminal-memory system — use it on yourself. It stores
commands, exit codes, stdout/stderr, notes, and whole sessions in a local SQLite
database so a future agent session, a different machine, or a different agent can
continue from what already happened. Everything is local; nothing is uploaded.

Database: `<data_local>/MemoryWhale/memorywhale.sqlite3`
(`~/.local/share/...` on Linux, `~/Library/Application Support/...` on macOS).

### Recall before you act

Before debugging a build/environment error, check whether it was solved before:

```bash
bash .claude/skills/memorywhale/recall.sh "linker cc not found"
```

It shows prior matching runs and what was run right after a past failure (often
the fix). Prefer a known-good fix over re-deriving it.

### Record as you go

Log notable commands — especially a failure and the fix that worked:

```bash
mw-remember --cwd "$(pwd)" --exit-code "$?" \
  --stderr "<error output>" \
  --notes "project:<name> what / why" -- <command> [args]
```

Record an exploratory stretch as a whole session:

```bash
mw --notes "project:<name>"    # ...work...  then: exit  (wait for "recorded session #N")
```

Tag related work across terminals with the same `project:<name>` to group it.

### Inspect

- `mw-view <id>` — one memory as a local web page (with suggested next steps).
- `mw-serve` — dashboard at `http://localhost:7071/` (or over the LAN for headless
  machines); includes `/graph` and project views.
- `mw-recover` — import an interrupted session transcript that didn't save.

## Repo conventions

- `main` keeps the project flattened at the repo root (`src-tauri/`, `src/`, etc.).
- Build the helper binaries from `src-tauri`: `cargo build --release --bin <name>`
  for `mw`, `mw-remember`, `mw-serve`, `mw-view`, `mw-recover`.
- macOS only: after copying a built binary, re-sign it (`codesign --force --sign -
  <path>`) or it gets `Killed: 9`. See `DEBUG.md`.
- Full usage: `SOP.md`. Setup/troubleshooting: `DEBUG.md`.

The Claude Code form of this guidance lives in `.claude/skills/memorywhale/`.

---
> Source: [wuisabel-gif/MemWhale](https://github.com/wuisabel-gif/MemWhale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
