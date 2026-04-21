## factoriomod

> `AGENTS.md` in this directory is a symlink to this file so Codex and Claude Code stay in sync. Edit this `CLAUDE.md` target, not the symlink.

# second_engineer — Shared Agent Instructions

`AGENTS.md` in this directory is a symlink to this file so Codex and Claude Code stay in sync. Edit this `CLAUDE.md` target, not the symlink.

## Repo Purpose

This repository contains the `second_engineer` Factorio 2.0 mod plus supporting tooling for worldgen authoring, blueprint conversion, and automated testing.

The mod theme is scarcity, salvage, and ruined infrastructure. Prefer changes that reinforce circular-economy gameplay and ruined-world presentation over vanilla-style infinite extraction.

## Repo Layout

- `second_engineer/` is the actual mod root loaded by Factorio.
- `docs/` contains stable design and behavior references.
- `docs/features/` documents implemented systems.
- `docs/hooks.md` is the runtime event map.
- `tools/` contains blueprint extraction, normalization, ruin-template, wear-profile, and sector-compilation tooling.
- `scripts/` contains shell wrappers for Factorio tests and world inspection.
- `planning/` and brainstorming docs are design input, not the operational source of truth.

## Working Rules

- Keep Codex and Claude instructions aligned by editing only `CLAUDE.md` targets.
- Treat `second_engineer/` as the source of gameplay code. Most implementation work belongs there.
- Prefer repo evidence over stale prose. When behavior docs and code disagree, update the docs or note the mismatch.
- Keep Space Age content behind `if mods["space-age"] then`.
- Favor small, targeted edits over broad rewrites unless the task explicitly calls for restructuring.

## Canonical References

- Mod identity: [docs/identity.md](docs/identity.md)
- Runtime hooks: [docs/hooks.md](docs/hooks.md)
- Feature behavior: `docs/features/*.md`
- Roadmap: [docs/planned.md](docs/planned.md)
- Worldgen pipeline and authored assets: `tools/`, `tools/BlueprintToLuaPipeline.md`
- Git workflow rules: [.claude/rules/git.md](.claude/rules/git.md)

## Additional Rule Files

Codex does not automatically discover arbitrary files under `.claude/rules/`. Surface them from this shared instruction chain instead. Claude-native rule loading remains the primary mechanism for Claude Code.

- Before doing git work, read and follow [.claude/rules/git.md](.claude/rules/git.md).
- When working in `second_engineer/**/*.lua`, also read and follow [.claude/rules/second_engineer/lua-guidelines.md](.claude/rules/second_engineer/lua-guidelines.md).

## Verification

Run the smallest relevant verification for the change:

- Lua/mod integration: `npm test`
- Node tooling only: `npm run test:node`
- Surface inspection: `npm run inspect:surface`
- Perimeter/worldgen inspection: `npm run inspect:perimeter`
- Central district regeneration: `npm run blueprint:build:central-district`

If a change affects prototype loading, runtime hooks, worldgen, or generated blueprint outputs, do more than a pure static review.

## Test Environment Notes

- Factorio integration tests run through [scripts/test-factorio.sh](scripts/test-factorio.sh).
- The repo supports WSL workflows that call a Windows `factorio.exe` when `FACTORIO_PATH` points to an `.exe`.
- `FACTORIO_PLAYER_DATA` may be linked into `~/.factorio/player-data.json` by the test wrapper.
- When the WSL + Windows path is active, keep Windows path semantics in mind for test execution and failures.

## Running Tests

FactorioTest is installed locally as a dev dependency (not in `info.json`). Enable it in the Factorio mod list, then:

```bash
# one-shot run
factorio-test run -p ./second_engineer

# watch mode — reruns on file changes
factorio-test run -p ./second_engineer -w
```

To add new test files: create `second_engineer/tests/<name>.lua` and add `require("tests.<name>")` to `scripts/tests.lua`.

## Stable Factorio Rules

- Use `data:extend(...)` only in data stage files.
- Use `storage.*` for persistent runtime state.
- Internal helper entities should stay hidden from normal player interaction unless a task explicitly calls for debug visibility.
- Keep generated assets and generated Lua outputs reproducible from source tooling rather than hand-maintained.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YuliRox) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
