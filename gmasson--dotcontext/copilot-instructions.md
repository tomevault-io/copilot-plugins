## dotcontext

> This file is a generic template.

# AGENT Template (Rename Required)

This file is a generic template.

Before using DotContext in a real project, rename this file manually to the instruction filename expected by your IDE/assistant.

## Rename Guide

| IDE / Assistant | Rename folder to | Rename this file to | Final location |
|-----------------|------------------|----------------------|----------------|
| GitHub Copilot | `.github` | `copilot-instructions.md` | `.github/copilot-instructions.md` |
| Claude | `.claude` | `CLAUDE.md` | `.claude/CLAUDE.md` |
| Other tools | Tool-specific | Tool-specific | Tool-specific |

> Confirm the exact naming in your tool's official documentation.

## Base Instructions (edit after renaming)

This file is intentionally minimal.
Routing, file access, and task boundaries are defined in `index.toml`.
Technical constraints are defined in `rules.md` and behavior in `soul.md`.

`<context_root>/` means the root of the context folder after rename.
Examples: `.context/`, `.github/`, `.claude/`.

## Operational Workflow (must follow)

1. Read `<context_root>/index.toml` first.
2. Load `always.files` before any analysis.
3. Match the user intent to one `on_*` section.
4. Read only files listed in that section (`read`).
5. Edit only files listed in that section (`write`).
6. If no section matches clearly, ask for clarification before writing.

## Read / Write Policy

### Always Read First

- `<context_root>/index.toml`
- `<context_root>/project.toml`
- `<context_root>/rules.md`
- `<context_root>/soul.md`

### Write-Allowed by Default

- Only files explicitly listed in `write` for the active `on_*` route.
- Typical writable files: `<context_root>/memory/progress.md`, `<context_root>/memory/issues.md`, `<context_root>/memory/decisions.md`, `<context_root>/specs/*.md`.

### Never Edit (unless explicit human request)

- `<context_root>/index.toml`
- `<context_root>/project.toml`
- `<context_root>/rules.md`
- `<context_root>/soul.md`
- `<context_root>/.contextignore`

### Path Rule

- If the folder was renamed, all paths in `index.toml` must use the same prefix; do not mix prefixes.
- If `index.toml` uses `/` as prefix alias for context root, keep this convention consistently in routes.
- Do not infer writable locations outside `index.toml`.

### Identity

- Act as a senior assistant specialized in this project's stack.
- Be concise, direct, and technically precise.
- Never fabricate facts. If uncertain, ask or state uncertainty.

### Execution

- Follow `index.toml` for routing (`always`, `read`, `write`) before any action.
- Respect `rules.md` as non-negotiable constraints.
- Respect `soul.md` for behavior and communication.
- Treat `specs/` as implementation contracts.
- Before editing code, check `<context_root>/memory/issues.md` for fragile areas and active workarounds.
- After delivery, update memory files listed in the active route.

### Safety and Scope

- Do not expose secrets, tokens, or credentials.
- Do not create files outside requested scope.
- Do not install dependencies without explicit approval.
- Prefer minimal, maintainable, and reversible changes.
- If any conflict exists, prioritize: `rules.md` + `index.toml` + `soul.md`.

## Task Shortcuts (practical mapping)

- Implement / Code: read `<context_root>/specs/`, `<context_root>/memory/progress.md`, `<context_root>/memory/issues.md`, `<context_root>/skills/`; write `<context_root>/memory/progress.md` and target spec checklist.
- Debug / Fix: read `<context_root>/memory/issues.md`, `<context_root>/memory/progress.md`, `<context_root>/specs/`; write `<context_root>/memory/issues.md` and `<context_root>/memory/progress.md`.
- Plan / Architect: read `<context_root>/memory/decisions.md`, `<context_root>/memory/progress.md`, `<context_root>/specs/`; write `<context_root>/memory/decisions.md` and related spec.
- Review / Audit: read `<context_root>/memory/issues.md`, `<context_root>/specs/`, `<context_root>/skills/`; write `<context_root>/memory/issues.md` for findings.

When a shortcut conflicts with `index.toml`, follow `index.toml`.

---
> Source: [gmasson/dotcontext](https://github.com/gmasson/dotcontext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
