## orange-garden

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is a personal MCP (Model Context Protocol) context server. Its goal is not to make AI smarter, but to provide stable **assumptions and principles** ("ground")
so that AI misunderstands the owner less. It exposes `AI_GUIDE.md` as a system-level context payload for LLM/tool clients.
Repo-wide AI entrypoint is [`AGENTS.md`](./AGENTS.md); this file is Claude Code-specific routing and constraints only.

## AI workflow: read in this order

When starting any task, read files in this order before writing or changing anything:

1. `AGENTS.md` — repo-wide entrypoint, precedence, and source-of-truth map
2. `AI_GUIDE.md` — the owner's values and behavioral expectations for AI
3. `docs/architecture/ai-development-system.md` — canonical parent for the AI development system
4. focused detail docs when needed
   - `docs/AI_ROLE_POLICY.md` — side-effect boundary detail
   - `docs/AI_WORKFLOW.md` — worktree / branch / VSCode detail
   - `docs/PLAYBOOK.md` — issue-to-handoff lifecycle detail
   - `docs/WORKER_POLICY.md` — dispatch detail
5. `CLAUDE.md` (this file) — Claude-specific routing and constraints
6. `docs/architecture.md` — system structure and extension points (if it exists)
7. Only then: the specific files relevant to the task

## AI role split

Claude remains the no-side-effect implementation side, and Codex remains the side-effecting executor/verifier.
Canonical parent: [`docs/architecture/ai-development-system.md`](./docs/architecture/ai-development-system.md).
Focused detail for side-effect boundaries: [`docs/AI_ROLE_POLICY.md`](./docs/AI_ROLE_POLICY.md).

If this file, `AI_GUIDE.md`, old templates, or historical comments conflict with the parent doc or focused detail docs:

1. prioritize `docs/architecture/ai-development-system.md`
2. use `docs/AI_ROLE_POLICY.md` for boundary detail
3. pause side-effecting work
4. escalate to the human Maintainer with the conflicting citations

To avoid false stops caused by stale wording, never treat non-authoritative role notes as a blocker by themselves.

`CLAUDE.md` is a routing document for Claude Code after `AGENTS.md` — it provides Claude-specific constraints and routes to authoritative sources. It does not replicate role boundary definitions, runtime runbooks, or notification operations.

When orchestration-specific decisions are needed:

* use [`docs/architecture/ai-development-system.md`](./docs/architecture/ai-development-system.md) for the development-system topology
* use [`docs/PLAYBOOK.md`](./docs/PLAYBOOK.md) for phase progression, handoff, pause, and resume rules
* use [`docs/WORKER_POLICY.md`](./docs/WORKER_POLICY.md) for dispatch policy and implementer/reviewer separation
* do not redefine those rules in this file

## Practical guardrails for Claude Code

When working in this repo, treat the following as hard constraints:

* Do not create or switch branches
* Do not run commands or report command results as if they were executed
* Do not delete, move, or rewrite files via shell operations; express those changes as unified diff only
* Do not create PRs or prepare GitHub state changes; Codex handles review-ready verification and PR creation

If a deletion is needed, provide:

1. the deleted file path
2. remaining references checked
3. replacement path or rationale for removal
4. verification commands for Codex to run

## Commands

```bash
# Install in editable mode (development)
pip install -e ".[dev]"
# or via Makefile shorthand:
make setup

# Lint and format (Makefile targets wrap ruff)
make lint          # ruff check .
make fmt           # ruff format .

# Run tests
make test          # pytest

# Run the CLI — subcommand is required
python -m personal_mcp.server --help
python -m personal_mcp.server event-today
python -m personal_mcp.server event-add "note text" --domain worklog
python -m personal_mcp.server web-serve --port 8080

# Makefile shortcuts for common CLI operations
make log TEXT="note text" DOMAIN=worklog   # event-add
make today                                 # event-today
make summary DATE=2026-03-11               # summary-generate
make run                                   # web-serve (port 8080)
make smoke                                 # log + today + summary smoke check

# Verify Ruff using pyproject.toml as the source of truth
python -m ruff check .
python -m ruff format --check .

# Verify AI_GUIDE.md copies are in sync
diff AI_GUIDE.md src/personal_mcp/AI_GUIDE.md
make guide-check
````

Notes:

* Ruff, pytest, and make targets are all configured in this repo.
* Ruff `line-length` and `E501` policy are defined in `pyproject.toml`; do not override them with CLI flags in docs or CI.
* `python -m personal_mcp.server` without a subcommand exits with a usage error — always supply a subcommand.

## Architecture

AI_GUIDE.md                          ← Authoritative guide (edit root, then sync copy)
docs/                                ← Design docs referenced from this file
src/personal_mcp/
AI_GUIDE.md                        ← Packaged copy (must match root AI_GUIDE.md)
core/guide.py                      ← load_ai_guide(): loads the guide with two-stage fallback
adapters/mcp_server.py             ← get_system_context(): thin adapter returning the guide text
adapters/http_server.py            ← web-serve HTTP adapter (mobile log form)
tools/                             ← one file per domain tool (event, daily_summary, github_*, …)
storage/                           ← storage boundary: events_store, jsonl, sqlite, path
server.py                          ← subcommand CLI (event-add, event-today, web-serve, …)

Data flow: server.py → tools/* / adapters/* → storage/* / core/guide.py → AI_GUIDE.md / data/

Event model principle:
* All domains converge to a common event format (timestamp, domain, payload)
* Future auto-ingestion sources follow the same event model — do not diverge
* Design should allow history reconstruction from stored events alone

## Where to put things

| What                                       | Where                                                                        |
| ------------------------------------------ | ---------------------------------------------------------------------------- |
| AI behavior rules (attitude, prohibitions) | AI_GUIDE.md (root), then sync to src/personal_mcp/AI_GUIDE.md                |
| New MCP adapter                            | src/personal_mcp/adapters/<name>.py + entry in docs/adapters.md              |
| New MCP tool                               | src/personal_mcp/tools/<name>.py + entry in docs/adapters.md or docs/tools/<name>.md |
| Architecture decisions                     | docs/architecture.md                                                         |
| Bug fix / refactor                         | Relevant .py file only; update docs only if public behavior changes          |
| Claude Code skill definitions              | .claude/skills/<name>/SKILL.md                                               |

## Skills
- Project skills live under `.claude/skills/`.
- For unclear requirements before issue drafting, use: `/clarify-request <request-fragment-or-issue-url>`
- For drafting a GitHub Issue from clarify results, use: `/issue-draft <clarify-request-output-b>`
- For implementation tasks tied to a GitHub Issue, use: `/minimal-safe-impl <issue-url-or-number>`
- For research/proposal tasks (no code changes), use: `/research-propose-structured <topic-or-issue-url>`

## AI_GUIDE.md sync rule

AI_GUIDE.md exists in two places:

* AI_GUIDE.md (repo root) — human-editable source of truth
* src/personal_mcp/AI_GUIDE.md — packaged copy accessed via importlib.resources

Rule:

* After editing the root file, copy it to the package path and commit both together.
* Verify with: `diff AI_GUIDE.md src/personal_mcp/AI_GUIDE.md`

## Branch / commit conventions (minimal)

* Branch names: feat/<topic>, fix/<topic>, docs/<topic>
* Commit messages: imperative, lowercase, no period (e.g., "add poe2 adapter skeleton")
* Keep changes single-concern and reversible

Claude Code note:

* These conventions are for humans or Codex when they perform Git operations
* Claude itself must not create branches, commit, or claim those actions completed

## Key design principle

This repo prioritizes 納得感・安全性・可逆性 (conviction, safety, reversibility) over efficiency and completeness.
Changes should be minimal, reversible, and slow by design. Do not over-engineer or add abstractions ahead of need.

* Do not break the non-evaluative design — avoid scoring, comparative expressions, or implicit improvement suggestions
* Do not add notifications or prompts that encourage user action
* Logs are append-only; treat past records as immutable
* When proposing deletion or overwrite of data, state the irreversibility explicitly and handle with extra care

## Change policy

* Do not make UX more stimulating (no urgency cues, streaks, countdowns, or progress bars)
* Do not introduce gamification elements of any kind
* Do not implement numeric optimization logic ahead of explicit user request

## Compatibility policy

MVP 期間中の互換性方針は以下を参照（重複コピペしない）：

* [README — 互換性ポリシー（MVP期間中）](./README.md#互換性ポリシーmvp期間中)
* [AI_GUIDE.md — 互換性に関するガードレール](./AI_GUIDE.md#互換性に関するガードレール)
* 背景: [Issue #19](https://github.com/wakadorimk2/personal-mcp-core/issues/19)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wakadorimk2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
