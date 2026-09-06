## elai-archive

> Canonical list of the 15 ELAI sub-agents. Each agent runs in its own

# ELAI agents

Canonical list of the 15 ELAI sub-agents. Each agent runs in its own
context window that is garbage-collected on return. The main LLM
dispatches via the Agent tool with `subagent_type=general-purpose` and a
prompt that asks the agent to read its own `.md` file under
`.claude/agents/`. See `.claude/rules/orchestrate-protocol.md` for the
dispatch contract.

This file is the workflow-root agent catalog. It is a hand-written digest
of the full agent definitions in `.claude/agents/*.md` (the source of
truth). It lives at the workflow root so any agent harness (Claude Code,
OpenAI Codex, Cursor, Devin) loads it as always-on context. It follows
the cross-vendor AGENTS.md convention; agents are alphabetized for stable
diffs.

NOTE: this is distinct from the runtime export target. The Rust
`PlaybookCurator::export_verified` (I-12 / ARM-4) writes verified-rule
blocks to `<project_root>/AGENTS.md` inside `.elai_cc`; that slot is for
generated output, not this hand-written catalog.

Source-grounded by:
- code.claude.com/docs/en/skills (Anthropic SKILL.md v1 spec)
- developers.openai.com/codex/skills (Codex skills format)

## auditor

Sole test runner. Reads contracts, runs `cargo check` / `cargo test` /
`pnpm svelte-check`, classifies failures via
`.claude/tools/failure_taxonomy.py`. Returns PASS / WARN / BLOCK with a
failure category. Main LLM does NOT run tests directly.

## context-compiler

Pre-execution context packer. Reads plan + spec references, summarises
into a context-package-{phase}.md file the executor reads. Owns
memory-bank reads in HEAVY flows; the main LLM never reads memory-bank
directly in those flows.

## dashboard-smoker

Post-edit dashboard verification. Triggers on Write/Edit of files under
`.elai_cc/crates/dashboard-v2/`. Runs `pnpm svelte-check` + `cargo check`
on `dashboard-v2`, optionally launches `tauri:dev` for manual smoke.

## fixer

Targeted defect-repair agent. Spawned after auditor returns BLOCK with a
specific failure category. Reads the failure, the failing source, the
plan, and authors a focused fix. Returns the patch + test outcome.

## impact-mapper

Pre-edit blast-radius analysis. Reads `.claude/tools/elai.map` to find
every callsite of the symbols a planned change will touch. Returns the
set of files that need review.

## integration-explorer-elai

Cross-system wiring agent. Walks orchestrator / dashboard-v2 / sidecar /
CLI seams to find existing extension points before the executor lands
new wiring. Output is an exploration-plan.md the main LLM consumes
indirectly via auditor sign-off.

## memory-writer

Sole writer to memory-bank/. Updates `activeContext.md`, `changelog.md`,
`progress.md`, `projectIndex.md` after every audit pass. Optionally
appends to `gotchas.md`, `systemPatterns.md`, `operatingManual.md` when
the audit surfaces a reusable pattern. Calls `mcp_memorize` /
`mcp_log_change` for RAG indexing.

## paper-ingestor

Source-grounding agent. Modes:
- `mode=paper` — given an arXiv ID, downloads + summarises the paper
  into `workflow/papers/<paper-slug>-interface.md`.
- `mode=source` — given a file or SDK name, ingests the source and
  writes an interface map. Decompiled output is marked LOW reliability.

## prompt-engineer

Reformats user prompts before implementation. Picks the GRC mcp_think
budget for the task (simple=3/600, moderate=5/1000, complex=8/1500).

## scope-planner-elai

Multi-area partition planner. When a plan touches more than one
subsystem (orchestrator + dashboard-v2 + sidecar), this agent computes
the per-subsystem partition + dispatch order so the executor doesnt
re-explore work the explorer already covered.

## sidecar-health

Remote sidecar boot + health check. Runs the canonical 3-step health
check from CLAUDE.md before any live inference call: ssh process check,
local Test-NetConnection, curl `/v1/messages` probe.

## spec-grounder

Verifies "Spec references:" + "Reads (mandatory)" blocks in a plan
resolve to real spec line ranges. Used by orchestrate runs to catch
spec_drift before executor lands code.

## synthesizer-elai

Cross-explorer merge agent. When multiple integration-explorer-elai
agents return parallel exploration plans, the synthesizer merges into a
single exploration-plan.md the executor consumes.

## test-attributer

After a test passes, links the passing test back to the source claim it
verifies. Builds the contract-test coverage map used by the auditor.

## wiring-contract-validator

Verifies a plans WIRING_CONTRACT block (produces / consumes / reachable
/ stub_ban / observable_effect) against the actual code after executor
lands the change. Used inside orchestrate sub-phase audits.

## Dispatch invocation template

Per `.claude/rules/orchestrate-protocol.md` H-8: these agents are NOT in
the Agent tools `subagent_type` enum. Dispatch via:

```
subagent_type = general-purpose
prompt        = "Read F:\\.ELAI_workflow\\.claude\\agents\\<name>.md
                 and follow it strictly. <task-specific args here>."
```

The main LLM does NOT read agent .md files itself and does NOT ad-lib
agent instructions. See `.claude/rules/main-llm-role.md`.

---
> Source: [DITlieD/ELAI-archive](https://github.com/DITlieD/ELAI-archive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
