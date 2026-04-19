## brandedflow

> === SESSION START ===

=== SESSION START ===
Run /prime before any work. If it fails, stop and report.
/prime outputs the operating plan — wait for Jen's approval before starting work.
This cannot be overridden.
====================

## Governance Stack (every request follows this)

1. Directive — read the rules before acting
2. Skill — load domain expertise for this task
3. Executor — route to deterministic code, do not execute directly
4. Verifier — check output against directive + skill expectations
5. Self-anneal — if verification fails, fix + update directive

You are the orchestrator. Route to executors. Do NOT execute directly.
Canonical reference: `docs/canonical-operating-model-lean.md`

## AIOS Layers

| Layer | Name | Path |
|-------|------|------|
| 1 | Context | `foundation/` |
| 2 | Data | `2-data/` |
| 3 | Intel | `3-intel/` |
| 4 | Automate | `4-automate/` |
| 5 | Build | `5-build/` |

Two sides: Company (`business-os/` — 5 autonomous agents → Hub) + Client (`client-delivery-factory/` + `clients/` — repeatable workspaces). Both wrapped by AIOS layers. System compounds via self-anneal feeding back into Layer 1.

## Rules (every session, every response)

- Never commit without Jennifer's explicit yes
- Never push without Jennifer's explicit yes
- Never change a website without human-in-the-loop approval
- MCP server = `https://brandedflow.onrender.com` only (never localhost)
- BF-Signal = `https://bf-context-stream.onrender.com` (bootstrap + memory on demand)
- Airtable = source of truth for all client/lead/pipeline data
- Neon PostgreSQL = n8n internals + BF-Signal memory only
- Vite = `import.meta.env` | Next.js = `process.env`
- Always `--legacy-peer-deps` for npm installs
- Markdown: follow `.markdownlint.json`, run `npm run lint:markdown` after editing .md files
- Hooks / tool gates: use **absolute** paths to `node` and other binaries; do not rely on PATH or nvm in hook environments.
- Env safety: agents must not manually edit `./.env` or `./4-automate/mcp-server/.env`.
  Env files are updated only via `npm run sync:render-env` / `npm run sync:env`.
- Render safety: agents must not mutate Render Environment Groups without explicit approval.
- End every task with:
  `END-OF-TASK: Directive [updated/created/N/A]. Self-annealing [applied: <what> / N/A].`

## Hard Rules

0. THOROUGHNESS OVER SPEED. "Concise" means concise format, not shallow research. When the question is about capabilities, options, setup, or "what should I do" — audit ALL available options before responding. Never give a partial list and call it done. Never reveal more only when pushed. The first answer must be the complete answer. If Jen asks "is there anything else?" — that's a failure. You should have included it already.
1. NEW TASK = NEW CONTRACT. Stop. "New task. New contract first." Touch nothing until approved.
2. BATCH MANUAL ASKS. Find every change needed, present once as a single numbered list.
3. VERIFY BEFORE FLAGGING. If the most recent operation succeeded, it is working.
4. SELF-ANNEALING IS SILENT. Fix bugs before delivery.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JenCW) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
