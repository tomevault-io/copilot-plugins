## foxline

> Use `trx` for all issue tracking. Do NOT use markdown TODOs or external trackers.

# foxline

## Issue Tracking (trx)

Use `trx` for all issue tracking. Do NOT use markdown TODOs or external trackers.

```bash
trx ready --json                              # Find unblocked work
trx create "Title" -t task -p 2 --json        # Create issue (types: bug/feature/task/epic/chore)
trx update <id> --status in_progress --json   # Claim task
trx close <id> --reason "Done" --json         # Complete work
```

Priorities: 0=critical, 1=high, 2=medium (default), 3=low, 4=backlog

## Repo

This is the `foxline` layout. It contains source code, runtime code, install tooling, and metadata only.

- `src/` - browser app source.
- `server/` - Foxline runtime server and bridge orchestration.
- `services/` - runtime-adjacent worker adapters, currently Qwen3-TTS and Parakeet helpers.
- `scripts/` - user-facing Bash entrypoints.
- `tools/` - install-time and extraction utilities only.
- `agents/` - agent definitions and registry only; runtime assets are generated locally.
- `assets/` - non-copyright metadata and mapping files only.
- `docs/` - selected technical notes for extraction decisions.
- `session-history/` - past coding agent sessions

Do not commit generated copyrighted/runtime outputs:

- `sources/**`
- `assets/generated/**`
- `agents/*/assets/avatar/**`
- `agents/*/assets/reference_audio/*.wav`
- `agents/*/assets/filler/*.wav`

---
> Source: [WismutHansen/foxline](https://github.com/WismutHansen/foxline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
