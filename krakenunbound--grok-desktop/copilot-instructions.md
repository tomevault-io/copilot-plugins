## grok-desktop

> For every programming task in this repository, Codex must work through the local

# AGENTS.md - Codex Quality Gate

## Working Agreement

For every programming task in this repository, Codex must work through the local
quality gate before claiming the task is finished.

## Quality Gate

- Inspect repository instructions, manifests, build scripts, and test setup before editing.
- Check whether the folder is a Git repository. Do not overwrite or discard unrelated user work.
- Identify the stack, package manager, build commands, test commands, lint commands, formatting commands, and project-specific release gates.
- Make the smallest safe, maintainable change that satisfies the task.
- Add or update targeted tests when the changed behavior can be tested locally. If no test harness exists for the behavior, create the smallest smoke/manual verification path and document it.
- Run all applicable checks. If any fail, diagnose the cause, fix it, and rerun the relevant checks.
- Self-review the diff for correctness, security, performance, maintainability, error handling, and documentation drift.
- Never claim absolute perfection or that no bugs exist. Report only evidence-backed verification.

## Repository-Specific Commands

- Install: `npm install`
- Frontend type check: `npm run check`
- Frontend build: `npm run build`
- Format write: `npm run format`
- Format check: `npm run format:check`
- Lint equivalent: `npm run lint`
- Rust format: `cd src-tauri; cargo fmt --all`
- Rust clippy: `cd src-tauri; cargo clippy --all-targets -- -D warnings`
- Release build smoke: `npx tauri build --no-bundle --ci`
- CI-style polish: `npm run polish`

## Repository-Specific Rules

- Do not leave `grok-desktop.exe`, `grok.exe`, dev servers, watchers, or helper services running unless the user explicitly asks.
- Prefer release verification with `src-tauri\target\release\grok-desktop.exe` and a real screenshot for UI changes.
- This project may not be a Git repository. Use timestamped source archives before substantial edits when Git is unavailable.
- Do not add dependencies without justification and lockfile updates.
- Do not weaken checks to make them pass.
- Keep public behavior backward compatible unless the task explicitly requests a breaking change.

---
> Source: [krakenunbound/grok-desktop](https://github.com/krakenunbound/grok-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
