## project-tokyo

> {3-5 line description of the product, the problem it solves, and the target audience.}

# {PROJECT NAME}

## Context

{3-5 line description of the product, the problem it solves, and the target audience.}
{Fill in during /speckit.constitution or /speckit.specify.}

## Project Type

{web app | CLI tool | library | data pipeline | mobile | desktop | service | game | other}
{Set during /speckit.plan.}

## Stack

{Defined during /speckit.plan — do not fill in before that.}

## Implementation Directories

Paths where Spec Kit enforcement applies. Writes to these paths require an active approved spec — the `spec-guard` hook blocks otherwise.

Set during `/speckit.plan` once project structure is decided. Mirror the list to `.claude/context/state.json` `implementation_dirs`.

```yaml
implementation_dirs:
  - src/
  - lib/
  # add more as your project structure requires
```

Default fallback (if state.json is empty): `src/`, `lib/`, `app/`, `pkg/`, `internal/`, `cmd/`, `api/`.

## Language Policy

All artifacts written to disk must be in English — `.md` files, configuration, code comments, commit messages, spec/task files, issue and PR descriptions. Conversations may be in any language; disk is English.

## Sources of truth (read on demand, do not embed)

- Non-negotiable rules: `.specify/memory/constitution.md`
- Architecture decisions: `docs/ARCHITECTURE-DECISIONS.md`
- Harness internals (hooks, enforcement, session lifecycle): `docs/harness-architecture.md`
- Module docs: `docs/modules/{module}.md` — load only the one you are touching
- Current spec state: `.claude/context/state.json`
- Session history (one line per session): `.claude/context/sessions/HISTORY.md`
- MCP contracts (only if MCP is used): `docs/optional/mcp-contracts.md`

## Spec Kit

Spec-Driven Development is **mandatory**. The constitution is the source of truth — consult it before naming artifacts or making architectural decisions.

Commands: `/speckit.constitution`, `/speckit.specify`, `/speckit.clarify`, `/speckit.plan`, `/speckit.tasks`, `/speckit.analyze`, `/speckit.implement`.

Writes to `Implementation Directories` without an active spec are **blocked at hook level** (not honor-system). After `/speckit.specify` and explicit human approval, use `/spec-activate <slug>` to mark the spec as the working one.

## Token economy

- Load only the context relevant to the current task
- Never load the full spec.md or tasks.md — reference specific sections
- Ask before loading another module's context
- Prefer pointers over embeds in any doc you write

## Defaults

- Never create secrets in code — use `.env`; never read `.env*` files
- Never make network calls not documented in the plan
- Ask before architectural decisions not covered in the spec
- Prefer established libraries over custom code
- Before committing: run lint and tests

## MCP policy (optional)

MCP is opt-in for this template. **Skip this section if your project does not expose tools to LLMs.**

When enabled:
- Claude.ai account connectors are disabled by default (`.claude/settings.json` sets `ENABLE_CLAUDEAI_MCP_SERVERS=false`)
- Project-specific MCP servers are declared in `.mcp.json` at repo root (committed, team-shared). The file is not pre-shipped — `claude mcp add --scope project ...` creates it on first registration. See `docs/optional/mcp.json.example` for reference stanzas.
- Contract first: register every tool in `docs/optional/mcp-contracts.md` before implementing — use the `mcp-contract` skill

## Skills (triggered on demand — do not inline their content here)

- `worktree-workflow` — create, merge, and clean up task worktrees
- `security-checklist` — run before marking any task complete
- `spec-flow` — guides through the Spec Kit sequence
- `mcp-contract` — register MCP tools before implementing (only triggers on MCP topics)

## Slash commands

- `/spec-activate <slug>` — mark a spec as the active one (enables writes in implementation dirs)
- `/spec-status` — show active spec, phase, and pending tasks
- `/session-resume` — load the last session log summary
- `/checkpoint` — force commit of pending changes (no push)

## Docs to keep current

- `docs/modules/{module}.md` when modifying a module
- `docs/optional/mcp-contracts.md` when creating or changing MCP tools (if used)
- `CHANGELOG.md` with Conventional Commits

---
> Source: [gabrielfernandesbonfim/project-tokyo](https://github.com/gabrielfernandesbonfim/project-tokyo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
