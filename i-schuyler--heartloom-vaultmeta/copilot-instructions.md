## heartloom-vaultmeta

> Repo-local Codex guidance for `heartloom-vaultmeta`.

# AGENTS.md

## Purpose

Repo-local Codex guidance for `heartloom-vaultmeta`.

Before docs or scope changes, consult `docs/IDENTITY_INHERITANCE.md`.

## Authority Ladder

1) [`heartloom-source`](https://github.com/i-schuyler/heartloom-source) owns upstream meaning, law, identity, and deeper architecture.
2) [`heartloom-identity`](https://github.com/i-schuyler/heartloom-identity) owns downstream governance, translation, inheritance posture, and anti-drift.
3) `heartloom-vaultmeta` owns repo-local tool behavior, output contracts, and install/docs specifics.

## Workflow Defaults

- Codex-prompt-first: wait for the prompt contract, then execute exactly that slice.
- PR-first workflow: branch before edits, keep slices small, and prepare push/PR commands after commit.

### PR Preflight (run at slice start)

- `git fetch origin`
- `git status --porcelain`
- `git branch --show-current`
- `git rev-list --left-right --count origin/main...HEAD`

### STOP Rules (before editing)

- If current branch is `main`, stop and create a slice branch.
- If divergence shows both ahead and behind as non-zero, stop and report.
- If working tree is dirty before the slice begins, stop and report.

### Safe Stale-Branch Cleanup (before new slice)

- Inspect open PRs and current branch state first.
- Clean up stale local/remote slice branches only when already merged and safe.
- Never delete the current branch, unmerged branches, or ambiguous branches.

## Required Prompt Sections

Every slice prompt should define:

- Scope
- NOT in this slice
- Evidence Block
- Hard Stop Conditions
- Verification Plan
- Output Contract

## Evidence + Hard Stops

- Collect exact repo `file:line` evidence before editing.
- If required evidence cannot be found, stop and report what is missing.
- If a requested change conflicts with `docs/IDENTITY_INHERITANCE.md`, stop and report the conflict.
- Do not redefine source-layer meaning or import heavy governance into this repo.

## Linking Rule

When referencing Heartloom repos in touched docs, use hyperlinks for [`heartloom-identity`](https://github.com/i-schuyler/heartloom-identity) and [`heartloom-source`](https://github.com/i-schuyler/heartloom-source) when practical.

## Output Contract + Clipboard

- Match the prompt-defined output contract exactly.
- Output contract must include:
  1) touched files
  2) behavior changes
  3) what was NOT implemented
  4) verification status
  5) stop conditions encountered
  6) re-entry hint

- copy the final summary to the. clipboard using the Termux API via `termux-clipboard-set`
- the clipboard payload must match the prompt-defined output contract exactly
- if Termux API is unavailable, report that explicitly instead of claiming clipboard success

---
> Source: [i-schuyler/heartloom-vaultmeta](https://github.com/i-schuyler/heartloom-vaultmeta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
