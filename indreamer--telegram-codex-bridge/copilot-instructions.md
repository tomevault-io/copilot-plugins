## telegram-codex-bridge

> summary: root routing contract for coding agents working in this repository

<!-- docmeta
role: agent
layer: 1
parent: null
children:
  - docs/AGENTS.md
  - src/AGENTS.md
  - scripts/AGENTS.md
  - skills/AGENTS.md
summary: root routing contract for coding agents working in this repository
read_when:
  - first agent entry into the repository
  - need to choose the docs, code, scripts, or skills domain
skip_when:
  - already inside the correct domain router
source_of_truth:
  - AGENTS.md
  - docs/AGENTS.md
  - src/AGENTS.md
  - scripts/AGENTS.md
  - skills/AGENTS.md
-->

# AGENTS.md

Agent router for `telegram-codex-bridge`.

Current shipped truth in this repo is still **Telegram-first**.
Future repository direction may describe a broader **Codex Bridge Core** with additional platform packs.
Keep those two truths separated.

Goal: let a coding agent build a useful mental index of the repo **without preloading the repo**.
The default path is:

1. read this file
2. read **one** domain agent
3. read **one** leaf doc or **one** narrow code file

That is the whole default retrieval path.

## Retrieval Contract

Maximum default depth: **3 layers**.

- Layer 1: `AGENTS.md` (choose the domain)
- Layer 2: one domain agent such as `docs/AGENTS.md` or `src/AGENTS.md`
- Layer 3: one leaf doc or one narrow source file

Do **not** read sibling docs or sibling code files "just in case".
Expand only when the active task is cross-cutting, conflicting, or explicitly architectural.

## Source Priority

Use sources in this order:

1. active user instruction
2. current docs in `docs/product/`, `docs/architecture/`, `docs/operations/`, and `docs/generated/current-snapshot.md`
3. current implementation in `src/`
4. protocol evidence in `docs/research/`
5. planning and history in `docs/roadmap/`, `docs/future/`, `docs/plans/`, and `docs/archive/`

Do not treat planning/history as shipped behavior.
Do not treat protocol capability as proof that Telegram UX already exposes it.

## Domain Router

| If the task is mainly about | Read next |
|---|---|
| current docs, intended behavior, product/runtime/ops questions | `docs/AGENTS.md` |
| current implementation, code ownership, refactors, bug fixes | `src/AGENTS.md` |
| GitHub install entry scripts | `scripts/AGENTS.md` |
| bundled Codex skills | `skills/AGENTS.md` |

## Fast Task Router

| Task | Domain agent | Typical leaf |
|---|---|---|
| v1 boundary, trust model, scope | `docs/AGENTS.md` | `docs/product/v1-scope.md` |
| unsure which Telegram UX doc to use | `docs/AGENTS.md` | `docs/product/chat-and-project-flow.md` |
| auth, project picker, browse, sessions, archive, rename, pin | `docs/AGENTS.md` | `docs/product/auth-and-project-flow.md` |
| Codex-backed Telegram commands, rich inputs, `/review`, `/rollback`, `/compact`, `/model`, `/skills`, `/plugins`, `/apps`, `/mcp`, `/account` | `docs/AGENTS.md` | `docs/product/codex-command-reference.md` |
| runtime cards, `/where`, `/inspect`, `/interrupt`, `/status`, final-answer delivery | `docs/AGENTS.md` | `docs/product/runtime-and-delivery.md` |
| callback payload families and stale/duplicate callback handling | `docs/AGENTS.md` | `docs/product/callback-contract.md` |
| current bridge-versus-platform decoupling progress, what has landed, or what is still Telegram-shaped | `docs/AGENTS.md` | `docs/architecture/platform-decoupling-status.md` |
| runtime lifecycle, SQLite state, recovery, degraded behavior | `docs/AGENTS.md` | `docs/architecture/runtime-and-state.md` |
| current Codex app-server adoption, supported request families, approvals, notification reduction, or rejected server requests | `docs/AGENTS.md` | `docs/architecture/codex-app-server-adoption.md` |
| current pack boundary, active pack selection, platform capabilities, or Telegram/Feishu pack ownership | `docs/AGENTS.md` | `docs/architecture/platform-pack-boundary.md` |
| current Telegram/Feishu capability matrix or future Web/App target capability rows | `docs/AGENTS.md` | `docs/architecture/platform-capability-matrix.md` |
| where ownership lives in `src/` today | `docs/AGENTS.md` or `src/AGENTS.md` | `docs/architecture/current-code-organization.md` |
| current internal Core seam, semantic view contracts, or workflow adapters | `src/AGENTS.md` | one narrow file under `src/core/` |
| new Codex Console continuation task or context-budget decision | `docs/AGENTS.md` | `docs/roadmap/codex-console-continuation-brief.md` |
| future multi-platform Core direction, platform packs, Web/App control surface goals | `docs/AGENTS.md` | `docs/future/multi-platform-core-prd.md` |
| install, config, service, update, diagnostics | `docs/AGENTS.md` | `docs/operations/install-and-admin.md` |
| volatile versions, counts, current size snapshot | `docs/AGENTS.md` | `docs/generated/current-snapshot.md` |
| command registry or Telegram UI implementation | `src/AGENTS.md` | `src/telegram/commands.ts` or one narrow `src/telegram/ui-*.ts` file |
| project/session orchestration implementation | `src/AGENTS.md` | `src/service/session-project-coordinator.ts` |
| Codex command orchestration implementation | `src/AGENTS.md` | `src/service/codex-command-coordinator.ts` |
| runtime-surface implementation | `src/AGENTS.md` | `src/service/runtime-surface-controller.ts` |
| app-server boundary or protocol adoption | `src/AGENTS.md` | `src/codex/app-server.ts` |
| SQLite persistence details | `src/AGENTS.md` | `src/state/store.ts` then one narrow `src/state/store-*.ts` file |
| GitHub shell installer behavior | `scripts/AGENTS.md` | one installer script |
| bundled setup skill behavior | `skills/AGENTS.md` | `skills/telegram-codex-linker/SKILL.md` |

## Retrieval Budget

Default budget for most tasks:

1. read **one** domain agent
2. read **one** leaf file
3. optionally read **one** verification file if there is ambiguity or a code/doc mismatch

Only go beyond that when:

- the task spans multiple domains
- intended behavior and implementation conflict
- the task explicitly asks for architecture, history, or roadmap synthesis

## Local MCP Helper

This repo's runtime environment includes a local helper command for the Feishu developer-docs MCP:

- `feishu-docs-mcp status` - report whether the `feishuDeveloperDocs` MCP block is present in `~/.codex/config.toml`
- `feishu-docs-mcp on` - add the managed `feishuDeveloperDocs` MCP config block to `~/.codex/config.toml`
- `feishu-docs-mcp off` - remove the managed `feishuDeveloperDocs` MCP config block from `~/.codex/config.toml`

Operational notes:

- after `on` or `off`, re-run `codex mcp list` locally if you need immediate CLI confirmation
- after `on` or `off`, send `/mcp reload` in the Telegram or Feishu bridge chat so the running bridge refreshes MCP status
- the helper manages only the Feishu developer-docs MCP, not `feishuOpenAPI`

## Conflict Rules

- If the active user instruction is explicit, follow it for the active task.
- If current docs and code differ, treat that as a real mismatch rather than blending them.
- If current docs are clearly newer intent, update code to match the active spec when the task is implementation.
- If code is the confirmed shipped behavior and docs lag, update docs to match the code when the task is documentation.
- If `docs/research/` shows a protocol capability, still verify bridge adoption before claiming Telegram support.
- If `docs/future/multi-platform-core-prd.md` describes a broader Core direction, treat it as future intent only unless the task is explicitly about that future direction.

## Stop Rules

Stop reading when one of these is true:

- one leaf file is enough to answer correctly
- one leaf file plus one verifier is enough to edit safely
- the remaining uncertainty is a real source conflict that should be reported

Do not widen the search just to feel safer.
Prefer a small, well-supported answer over a repo-wide reading pass.

## Anti-Patterns

Avoid these mistakes:

- reading `README.md`, `docs/README.md`, and multiple product docs before choosing a domain
- reading `src/service.ts` first for every implementation question
- reading protocol research when the question is purely about current Telegram UX
- reading roadmap/future/plans/archive before current docs and code
- mixing intended behavior, observed behavior, and future intent into one unsupported answer
- answering multi-platform architecture questions from current Telegram implementation alone without checking `docs/future/multi-platform-core-prd.md`

## User-Facing Output Contract

This section only constrains the final user-visible wording.
It must not change retrieval, planning, verification, editing, or testing behavior.

Unless the user explicitly asks for sources or exact file locations, do **not** put in the final reply:

- `docs/...` or `src/...` paths
- markdown links to local repo files
- directory trees or file inventories
- AGENTS routing steps or retrieval breadcrumbs
- doc citation lists when plain-language synthesis is enough

Prefer a plain-language answer.
If file paths appear in a draft, remove them unless they are strictly necessary to resolve a real conflict or make an edit.

## Final Reminder

Default traversal for this repo is:

`AGENTS.md` -> one domain agent -> one leaf file

Keep the index shallow.
Keep retrieval incremental.
Load more only when the task proves you need more.

---
> Source: [InDreamer/telegram-codex-bridge](https://github.com/InDreamer/telegram-codex-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
