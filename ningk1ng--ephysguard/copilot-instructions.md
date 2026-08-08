## ephysguard

> EphysGuard is a coding-agent safety harness for in-vivo electrophysiology data

# EphysGuard Repository Guidelines

EphysGuard is a coding-agent safety harness for in-vivo electrophysiology data
analysis. It is not an electrophysiology analysis pipeline and must not absorb
domain algorithms, private datasets, or the user's reference analysis package.

The Pi desktop shell is the interaction surface. Safety decisions belong to a
host-owned core that treats the model as an untrusted proposer:

- Raw inputs are registered by identity and remain read-only.
- Agents may propose code, configuration, and analysis intents but may not
  approve their own configuration, network access, physical mapping, event
  clocks, manual curation, or scientific claims.
- Unknown scientific facts stay unknown. Missing evidence blocks the affected
  action instead of being filled with a plausible value.
- A successful process exit is not a scientific completion signal. Completion
  requires input, configuration, run, output, and human-review evidence to
  refer to the same snapshot.
- Keep adapters narrow. The first external adapter may invoke tungsten_ephys,
  but no tungsten_ephys source code or example data belongs in this repository.

These rules apply for the full session.

## Workflow
- Define success criteria before coding; if unclear, stop and clarify.
- For non-trivial work, plan verification up front with the `self-test` skill.
- Do not create or switch to new branches to start work unless the user explicitly asks; respect the current branch or worktree as intentional.
- Commit in small focused checkpoints; don’t batch unrelated changes.
- Run `simplify` before closing non-trivial implementation work.

## Product
- This repo is building an EphysGuard desktop app on the `pi` runtime; preserve the conversation-first product direction.
- Desktop work is not done until it is verified on the real Electron surface, not only by unit tests.
- Transcript/timeline behavior, session correctness, and Codex-style UX are product features, not polish.
- Prefer clean reimplementation over patching around local complexity.

## Safety
- Never delete user session history, cached transcripts, screenshots, or temp artifacts without approval.
- Treat files you didn’t edit as read-only when multiple agents may be working.
- Ask before destructive commands or history rewrites.

## Structure
- Prefer path-scoped guidance in nested `AGENTS.md` files over growing this file.
- Keep the desktop renderer/main/preload boundary tight; avoid broad Node exposure to the renderer.
- Keep `pi-sdk-driver` thin over `pi-mono`; don’t fork or reimplement `pi` runtime behavior unless necessary.

## Source Of Truth
- Root `AGENTS.md` is the repo instruction source of truth.
- Root `CLAUDE.md` should remain a symlink to `AGENTS.md`.

---
> Source: [NingK1ng/ephysguard](https://github.com/NingK1ng/ephysguard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
