## loopgen

> `/loopgen` is a **compile-time prompt compiler**: a markdown-only skill (no

# loopgen — repo working notes

`/loopgen` is a **compile-time prompt compiler**: a markdown-only skill (no
runtime, no build, no test harness). The skill source is the inner `loopgen/`
dir (`SKILL.md` + `archetypes/` + `primitives/` + `references/` + `templates/`).

## Live-linked — edits here are immediately active

The inner `loopgen/` dir is symlinked into all three agents:

- `~/.claude/skills/loopgen`
- `~/.codex/skills/loopgen`
- `~/.pi/agent/skills/loopgen`

Each points at the working tree, so any edit is live in `/loopgen` the moment it
hits disk — there is **no install/sync step**. (Cold-start on a new machine is a
manual `git clone` + `ln -s`, documented in bootstrap; it is *not* auto-installed.)

## Contract verification

Contract verification lives at `tools/` (`verify_loopgen_contracts.py` +
`classify.py`) and runs via `make check`. `dev/` (gitignored, local-only) stays
maintainer governance — `dev/GOVERNANCE.md` and planning scratch — not the
verifier's home.

## Conventions

- **No plan docs in the tree.** Plan docs (`docs/plans/*`) are session
  scaffolding, not loopgen content — **none should live here**. Delete them once
  the work lands; the durable decision goes to an ADR, never a retained plan doc.
- **Architecture decisions → `docs/adr/`.** Boundary / contract / scope decisions
  get a numbered ADR (see `0001`). Reference the merge commit as the lineage
  anchor; do not keep a plan doc as the record.

---
> Source: [pro-vi/loopgen](https://github.com/pro-vi/loopgen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
