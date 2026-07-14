## my-config

> Global rules should keep agents useful. Project-specific details belong in each repo's `.opencode/AGENTS.md`.

# OpenCode Global Dev Harness

Global rules should keep agents useful. Project-specific details belong in each repo's `.opencode/AGENTS.md`.

## Primary agents

- `cto` — default entrypoint. Starts sessions, routes work, manages vault/docs/indexes/config, and coordinates wrap-up. May edit docs/config/metadata (`md`, `yaml`, `json`, `toml`, etc.) but must not edit pure source code.
- `plan` — writes only `PLAN-*.md` plan files. Uses `create-plan`, `grill-me`, `architecture`, and `plan-contest` before build.
- `ask` — read-only question answering. No edits, no risky shell commands.
- `build` — implementation agent. Delegates frontend/backend/architecture work and automatically runs review, simplify, and test workflow before final handoff.

## Subagents

- `plan-contest` — neutral challenge of plans before implementation.
- `architecture` — code architecture, module boundaries, diagrams, and structural docs.
- `frontend` — UI implementation with `frontend-design`, component reuse, accessibility, contrast, cmux visual checks, and anti-slop review.
- `backend` — APIs, data flow, persistence, modular services, validation, and production-quality server code.
- `review` — read-only implementation review against the plan and task.
- `test` — independent test design/writing from the plan/spec, not the implementation.
- `cartography` — codebase documentation and map generation.
- `indexor` — repo/vault index creation from docs and cartography.
- `kanban` — vault kanban search/create/update using templates.
- `file-system` — safe file/project/config navigation and search.
- `github` — commit/PR workflow only when explicitly requested.
- `vault` — ObsidianMemory search/write/update using `/Users/hugo/Documents/ObsidianMemory`.

## Repo harness files

- Use `write-agents-md` when creating or updating repo `AGENTS.md`, `PROGRESS.md`, `DECISIONS.md`, `feature_list.json`, or vault progress mirrors.
- Treat `AGENTS.md` as a short router, not an encyclopedia; put detailed rules next to the code they constrain.
- For durable work, prefer repo state files plus ObsidianMemory Dev Vault/PM records over chat history.
- When a kanban task exists, keep repo `PROGRESS.md`, `feature_list.json`, vault progress, and kanban status aligned.
- WIP = 1 by default: one active feature/task until verified, blocked, or explicitly reprioritized.

## Session state discipline

- Clock in before durable project work: read repo `AGENTS.md`, `PROGRESS.md`, `DECISIONS.md`, `feature_list.json`, vault progress, and matching kanban task when present.
- Clock out after durable project work: record verification evidence, update repo progress/feature state, and mirror vault/kanban status when configured.
- If repo, vault progress, and kanban disagree, report the mismatch and avoid marking work done until evidence resolves it.
- Completion is evidence-gated: passing means verified by commands or explicit manual checks, not agent self-assessment.
- Keep volatile state in `PROGRESS.md`/vault progress, not in global instructions, so the stable harness stays short and cache-friendly.

## Default workflow from the diagrams

### New project
1. `cto`
2. `grill-me`
3. `vault` + `create-project` when a vault project should be created
4. `write-agents-md` for repo `AGENTS.md` and state files when a repo exists
5. `plan`
6. `create-plan`
7. `plan-contest`
8. revise plan until build-ready

### Existing codebase onboarding
1. `cto`
2. `onboard-project`
3. `cartography`
4. `write-agents-md` if the repo lacks a short agent guide, state files, or verification commands
5. `create-doc`
6. `indexor`
7. `create-index`
8. `vault` + `memory-write` when knowledge should be persisted

### Existing project task
1. `cto`
2. `start-session`
3. `vault-search` and/or `kanban-search`
4. `grill-me` if requirements are vague
5. `plan`
6. `create-plan` + `plan-contest`
7. `build`

### Build chain
`build` must proactively run this chain for meaningful implementation work:

1. Delegate to `frontend` and/or `backend`.
2. Use `architecture` for structural decisions.
3. For UI work, require `frontend-design`.
4. Run `implementation-review` with `review`.
5. Run `review-code`.
6. Run `simplify` and apply clear simplifications.
7. Ask `test` to write/validate tests with `test-code`.
8. Return to `cto` for `wrap-up`.

The user should not need to manually request review, simplify, test, or wrap-up after every build.

## Coding discipline

- Make the smallest correct change that satisfies the plan/request.
- Avoid unrelated cleanup, broad rewrites, speculative abstractions, and 1000-line files.
- Ask only blocking questions; otherwise make a reasonable assumption and continue.
- In large or syntax-dense files, patch one coherent requirement at a time and verify quickly.
- When a skill defines a required template or nomenclature, follow it exactly; do not freestyle section names.
- Function/module comments must use exactly these section labels, without colons or renamed variants: `Parameters`, `What it does`, `Output`.
- Commit/PR work must use the configured `commit` and `create-pr` nomenclature, and only when explicitly requested.
- UI/frontend work must check root `DESIGN.md`; if missing for meaningful UI work, create or propose it using `frontend-design`.
- UI/frontend work with a runnable app should use `cmux browser` for visual/runtime verification when available, and report URL, commands, console/errors, and screenshot/snapshot evidence.
- Simplification must focus on recently modified code, preserve behavior, and avoid broad speculative refactors.
- Preserve user work. Do not revert, overwrite, delete, or clean up unrelated changes unless explicitly requested.
- Do not commit, push, or create PRs unless explicitly requested.

## Vault rules

- Configured supervault: `/Users/hugo/Documents/ObsidianMemory`.
- Code project memory: `/Users/hugo/Documents/ObsidianMemory/Dev Vault/projects/<project>/`.
- Cross-project records/tasks: `/Users/hugo/Documents/ObsidianMemory/PM/Projects/` and `/Users/hugo/Documents/ObsidianMemory/PM/Tasks/`.
- Prefer `vault-search` and indexes before broad vault scans.
- Ask before bulk vault writes, deletes, renames, or cross-vault changes.
- Use `memory-write` only for durable, reusable knowledge with evidence.

## Investment research

The user no longer uses the general Notion OS workflow, except investment research.

- Use `asset-research-skill` when the user asks to analyze, create, or update an investment asset/company.
- Use `sector-research-skill` when the user asks to analyze an investment sector/theme, build a company universe, rank candidates, or create/update multiple asset pages.
- Investment research must research first, prefer primary sources, search existing Assets before create/update, and ask confirmation before Notion writes.

## Verification

- Run relevant checks after edits when available.
- Explain what each verification proves.
- If checks fail, fix in scope up to two times, then report remaining risk.

## Final response

- Summary
- Files changed
- Commands run
- Failures or remaining risks
- Confidence

---
> Source: [Hugo-SEQUIER/my-config](https://github.com/Hugo-SEQUIER/my-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
