## runoq

> This repository implements the deterministic runtime layer for GitHub-backed agentic development orchestration.

# runoq

This repository implements the deterministic runtime layer for GitHub-backed agentic development orchestration.

## Core principles

- Keep durable behavior in code and machine-readable contracts, not in prompts.
- Prefer structured outputs over prose at script and tool boundaries.
- Treat GitHub as the audit and control surface.
- Treat `.runoq/state/*.json` as local recovery breadcrumbs, not the system of record.
- Preserve the target repository's main checkout; do execution work in sibling worktrees or explicit sandboxes.

## Development guidelines

- ALWAYS USE TDD SKILL WHEN WRITING CODE
- Keep the CLI thin and push stable logic into reusable runtime components.
- Prefer small, composable commands with explicit inputs, outputs, and side effects.
- Preserve existing contracts unless there is an intentional, documented versioned change.
- Favor deterministic behavior over cleverness.
- Reuse existing helpers, fixtures, and workflows before adding new ones.
- Keep prompts and agents thin; they should call stable repository logic rather than reimplement it.
- Focus on modularity and testability. Avoid monolithic scripts or agents that do everything.
- If current code architecture manifests as an implementation concern, prefer refactoring into a cleaner architecture over hacking solution in place; follow Kent Beck's principle of "First make the change easy, then make the easy change"

## Testing and validation

- Add or update deterministic tests whenever behavior changes.
- Use fake `gh` fixtures for integration-style coverage where possible.
- Use live smoke lanes for real GitHub and real LLM validation, but keep them opt-in and credential-gated.
- Validate both happy paths and recovery or failure paths.
- Smoke test environment variables are in `.env.smoke-sandbox`. Source it before running live smoke tests. See `docs/live-smoke.md` for full setup.

## Go code

When editing or creating Go code, load and follow these skills from `.agents/skills/`:

- `use-modern-go`
- `golang-linter`
- `golang-data-structures`

Rules:
- Always load `use-modern-go` and `golang-linter` before making Go changes.
- Load `golang-data-structures` whenever changing data structures, collection types, pointer usage, or performance-sensitive paths.

# runoq runtime notes

- Agents and skills in this repo are loaded into target projects via `--add-dir`.
- Utility scripts are the source of truth for queue state, PR lifecycle, state management, and reporting.
- Do not duplicate deterministic rules from the scripts into prompts.

## Documentation

- Update docs when changing operator-visible behavior, contracts, or architecture.
- Read `docs/development-guidelines.md` for the more detailed contributor guidance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DropBearLabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
