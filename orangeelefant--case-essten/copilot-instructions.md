## case-essten

> Mandatory instructions for automation agents (Claude Code, Codex, etc.) working in this repository. Read before making changes.

# AGENTS.md

Mandatory instructions for automation agents (Claude Code, Codex, etc.) working in this repository. Read before making changes.

## Repository

- **Name:** `Webraketen/case-essten`
- **Purpose:** Webraketen case study — essten
- **Production URL:** https://esstenochanlaggning.se

## Before You Change Anything

1. Read `CLAUDE.md` for project context, stack, and conventions.
2. Read `TODO.md` to check for in-flight priorities and avoid duplicate work.
3. If the change is non-trivial (new feature, refactor, dependency bump), plan first — surface tradeoffs before coding.

## House Rules

- **Branching:** Push directly to `main`. No PRs unless explicitly requested.
- **Commits:** One logical change per commit. Conventional, present tense.
- **Build before push:** Run the project's build/test (`npm run build`, `pytest`, etc.) locally before pushing.
- **Secrets:** Never commit `.env`, API keys, or credentials. Use `~/.secrets` and reference via `${VAR_NAME}`.
- **Style:** Match existing code style. Don't refactor adjacent code that isn't part of the task.
- **Files:** Don't add files to repo root unless they belong there (configs, READMEs, manifests). Source under `src/`, tests under `tests/`, docs under `docs/`.

## After Every Change

Update these files in the same commit as the code change:

- `CHANGELOG.md` — add a bullet under `## [Unreleased]` describing the user-visible change.
- `TODO.md` — tick off completed items; add any follow-ups you discovered.

This is not optional. A commit that touches behavior but skips CHANGELOG/TODO updates is incomplete.

## Domain-Specific Rules

- Copy defaults to **Swedish** for client-facing surfaces (per Webraketen house rule), unless this repo's CLAUDE.md says otherwise.
- For UI work, follow `DESIGN.md` if present; create one via the `awesome-design` skill before designing if missing.

---
> Source: [orangeelefant/case-essten](https://github.com/orangeelefant/case-essten) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
