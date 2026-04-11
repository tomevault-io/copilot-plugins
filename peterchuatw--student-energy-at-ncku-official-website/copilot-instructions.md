## student-energy-at-ncku-official-website

> Repository-level operating rules for Codex and collaborators.

# AGENTS.md

## Purpose
Repository-level operating rules for Codex and collaborators.

## Core Workflow Rules
- Never run `git push` unless the user explicitly asks for it in the current turn.
- After code changes, stop at local verification and report status before any remote action.
- Prefer `npm.cmd` / `npx.cmd` in this Windows + Codex environment.
- Distinguish sandbox issues from project issues before changing env vars or system setup.

## Editing Rules
- Use `apply_patch` as the default file editing method.
- If `apply_patch` fails (especially with no actionable error output), use the PowerShell fallback workflow in `DEBUGGING.md` under `File Edit Fallback (apply_patch -> PowerShell)`.
- In fallback mode, always read before write and verify after write to prevent accidental overwrites.

## Safety Rules
- Do not run destructive commands unless explicitly requested.
- Treat these as destructive/high-risk: `git reset --hard`, `git clean -fdx`, recursive forced delete.
- If a command can affect remote history or shared environments, require explicit user confirmation in the same turn.

## Runtime Validation Rules
- `npm run build` is required but not sufficient for homepage/runtime-heavy changes.
- For Home/Hero/Navbar or visual-runtime changes, run `npm.cmd run smoke:home` when feasible.
- If smoke fails with sandbox-style spawn/permission errors, retry with elevated execution before concluding code is broken.

## Reference Docs
- Runtime/white-screen troubleshooting: `DEBUGGING.md`
- PR validation checklist: `PR_CHECKLIST.md`

## Notes
- Keep this file concise and behavior-focused.
- Put detailed diagnostics and step-by-step playbooks in `DEBUGGING.md`.

## Task Completion Reporting
- At the end of each task, report:
  - files changed
  - verification commands run
  - key outputs (pass/fail or important errors)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PeterChuaTW)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PeterChuaTW)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
