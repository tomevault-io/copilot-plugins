## heterarchy-alexandria

> - For requested repository changes, perform reversible in-scope local edits and relevant non-destructive validation without unnecessary confirmation.

# Repository Guidelines

## Execution and Approval
- For requested repository changes, perform reversible in-scope local edits and relevant non-destructive validation without unnecessary confirmation.
- Preserve unrelated dirty worktree changes. Do not use broad reset, clean, revert, checkout, or stash as routine cleanup.
- Ask before external publication/deployment, credential mutation, destructive data actions not already requested, commit/push, or irreversible scope expansion.

## Applicable Instructions
Precedence: platform/safety → current user task → closest `AGENTS.md` → broader repository instructions → adopted Engineering Harness profile/rules → task-relevant Skills → live code conventions.

Load only the rule families relevant to the touched surface. PRDs and notes are task inputs, not engineering-rule authority unless explicitly designated.

## Engineering Harness
Engineering testing, verification, change discipline, solution minimality, and language invariants follow the repository Harness under `.agents/`.

The `.agents/` bundle is private, locally provisioned, and excluded from Git and
its history. Preserve its local rules, skills, and documents; do not publish them. Agent development
guidance continues to use this bundle; clean checkouts and CI do not require it.
Repository-owned executable Python checks and their configuration live under
`backend/scripts/verification/`; Rust checks live in `native/xtask/`. Canonical
CI runs those tools without checking the private bundle's presence or manifest.

Shared contracts:
- `.agents/shared/contracts/00-harness-authority-and-layering.md`
- `.agents/shared/contracts/01-testing-and-verification.md`
- `.agents/shared/contracts/02-agent-execution.md`
- `.agents/shared/contracts/04-solution-minimality.md`
- `.agents/shared/skills/minimal-engineering/SKILL.md`

Python work:
- `.agents/python_dev_harness/PROJECT_PROFILE.md`
- `.agents/python_dev_harness/HARNESS.toml`
- `.agents/python_dev_harness/rules/README.md`
- load only applicable normal/type/async/Pydantic/framework profile rules and Skills.

Rust work:
- `.agents/rust_dev_harness/PROJECT_PROFILE.md`
- `.agents/rust_dev_harness/HARNESS.toml`
- `.agents/rust_dev_harness/rules/README.md`
- `.agents/rust_dev_harness/skills/rust-engineering/SKILL.md`

Python↔Rust boundary work additionally loads:
- `.agents/python_rust_dev_harness/CROSS_LANGUAGE_PROFILE.md`
- `.agents/python_rust_dev_harness/CROSS_LANGUAGE.toml`
- `.agents/python_rust_dev_harness/rules/README.md`

Do not duplicate Harness doctrine in AGENTS. If a mechanical verifier is stricter than its governing rule/profile, treat it as Harness drift before distorting production design.

## Repository Architecture
- Python owns MCP/FastAPI/Pydantic/DI, lifecycle/policy, PostgreSQL/Redis/filesystem effects and external I/O.
- PostgreSQL is the persistent runtime authority for indexed graph source and memory data.
- Rust is the single production authority for declared deterministic compute, including graph projection/traversal/candidate selection and retrieval kernels.
- PyO3 adapter code stays thin; Rust core has no Python/PyO3/database/effect ownership.
- Permanent Python/Rust dual compute authority and SQLite compatibility fallbacks are forbidden.
- `backend/app/shared/` is for genuinely shared contracts/infrastructure, not a generic utility bucket.

## Build and Verification
Canonical entrypoints:
- backend focused work: `cd backend && uv run ruff check .`, `uv run pyrefly check`, relevant `uv run pytest ...`
- Rust/native: `cargo xtask rules`, `cargo xtask ci`
- repository closure: `make ci`

Only checks actually executed are VERIFIED. Report exact failures/blockers and unexecuted lanes.

## Subagent Routing
Repository-local Codex topology:
- Main/Coordinator: GPT-6 Astra with `high` reasoning.
- All actual subagents: GPT-5.6 Luna with `max` reasoning.
- Canonical roles: `context_steward`, `exploration_steward`, `implementation_steward`, `integration_steward`, `verification_steward`.
- No separate technical-lead model tier is active; Astra Main owns architecture and authority decisions.

Use the fewest agents that materially improve correctness, evidence, or elapsed time; zero-worker routing is valid. Parallelize only independent work with disjoint ownership. Never run overlapping writers.

Astra Main owns user intent, scope interpretation, architecture/authority decisions, root-cause and contract freeze, decomposition, acceptance criteria, conflict resolution, and final semantic acceptance. Luna workers own bounded context gathering, exploration, implementation, integration, and verification evidence.

Luna must escalate architecture/authority ambiguity, concurrency or race semantics, recovery, persistence, security, unsafe/FFI, protocol semantics, or cross-boundary lifecycle questions directly to Astra Main rather than designing a new authority locally.

`context_steward` and `exploration_steward` are read-only. `implementation_steward` and `integration_steward` are bounded writers after contract freeze. `verification_steward` is an independent evidence producer and must not modify tracked production source, tests, fixtures, golden files, configuration, expected outputs, or acceptance criteria.

When Forge is used, Goal/DAG truth, Writer Lease, source/worktree mutation, Approval/Execution, ChangeSet/Diff and evidence provenance remain Forge-owned. Do not duplicate Activity/Goal/DAG state, overlap writers, or retry an unknown mutation outcome before durable read-back.

## Final Report
Report only meaningful changes, affected authority/runtime surfaces, exact validation outcomes, and unresolved risks or unverified lanes.

---
> Source: [heterarchy-systems/heterarchy-alexandria](https://github.com/heterarchy-systems/heterarchy-alexandria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
