## whodecided

> Guidance for AI agents and contributors working on **whodecided** itself.

# AGENTS.md

Guidance for AI agents and contributors working on **whodecided** itself.

## What this is

A thin TypeScript CLI that distills the decisions an AI coding agent (and you) made inside Claude Code / Codex sessions into an append-only, ratifiable ledger, then feeds ratified precedent back to agents. "Common law for agentic coding."

The product is: a schema + a discardable CLI + a `.wdd/` directory of plain text inside the user's repo. `git`, `claude -p`/`codex exec`, and the browser are shelled out. The tool holds no state of its own.

## Build / test / run

```bash
npm run build      # tsc → dist/
npm test           # vitest (unit tests live next to sources as *.test.ts)
node dist/cli.js --help
```

- Node ≥ 20, ESM (`"type": "module"` → import paths use `.js`).
- Single runtime dependency: `yaml`. Do not add more; argv parsing and everything else is stdlib.
- `dist/` and `planning/` are gitignored. Design docs (SPEC.md, REVIEW-SPEC.md, CROSS-REVIEW.md) live in `planning/` — read them for the full rationale; they are not published.

## Architecture

Read-only pipeline with exactly two write points (distill → ledger, ratify → ledger):

```
cli.ts → commands/*            command routing + orchestration
  transcript/  locate + parse  CC (~/.claude/projects) and Codex (~/.codex/sessions rollouts) → SessionEvent[]
  extract/     candidates      deterministic decision candidates + aware heuristics (NO LLM)
  distill/     detect + distiller   auto-detect claude/codex (config wins, sessions vote) → subprocess → LedgerEntry[] (schema-validated, retry, degrade)
  ledger/      schema + io      append-only jsonl read/write; verdictOf/isPinned/noteOf
  patch/       apply            THE single write path: validate → append ratify lines → (opt) commit
  ledger/      evidence         persisted (redacted) transcript excerpt per decision, keyed by ref
  extract/     redact           mask secrets before a slice reaches the distiller or evidence
  checks/      gate diff-signals risk supersede coverage   pure triage signals over ledger/diff/traces
  commands/    signals          assembles branch diff + per-decision signals for review/board/report
  render/      term/md/html     terminal docket / PR markdown / single-file workbench (Timeline + Threads tabs, keyboard review, change map, per-decision diff, triage)
  config.ts git.ts jsonl.ts validate.ts   infra (git.ts: branchDiff/churn/productFiles)
```

`render/*` and `checks/*` are pure functions (data in, string/struct out) — unit-test them directly.

## Invariants (do not break)

- **Single write path.** Every ledger mutation goes through `patch/apply.ts applyOps`. The live server's `POST /apply` and terminal ratify both call it. Never write `ledger.jsonl` elsewhere except the distill append in `commands/distill.ts`.
- **Precedent is human-ruled only.** `recall` / the CLAUDE.md digest never surface unratified (pending) decisions — that would let an agent follow its own unreviewed guess. Enforced by `recallHits` and `sync.precedentLines`.
- **Never auto-commit by default.** The tool writes files; it commits only when `config.commit === "on"`. Do not touch the user's git history uninvited.
- **Hooks never disturb the session.** `hook trace` / `hook autodistill` swallow all errors and exit 0.
- **Latest ruling wins.** `verdictOf`/`isPinned`/`noteOf` scan ratify lines from the end. The ledger is append-only; changing a mind appends, never rewrites.
- **Self-contained provenance.** Each decision's `ref` is backed by a persisted, redacted excerpt in `.wdd/evidence.jsonl`, so the "why" survives transcript rotation and travels with the audit. Secrets are masked (`extract/redact.ts`) before any slice reaches the distiller or disk.
- **Show the code, not just the claim.** The workbench and `report --md` surface the branch's real diff (a change map, plus per-decision hunks) and flag weakened tests (`checks/diff-signals.ts`) - a silent decision a plain diff scan misses. Diff capture is read-only (`git diff`, `.wdd/` excluded); no LLM.
- **Help the reviewer triage.** Three deterministic signals rank and connect decisions, all from data already on hand (no LLM): `checks/risk.ts` scores the silent × hard-to-reverse × hedged quadrant plus diff-derived churn, scatter (change entropy, Hassan) and fan-out to seed a "review these first" list; `checks/supersede.ts` links a later decision that walks back an earlier one (reversal language + most shared files, one best match); `checks/coverage.ts` says whether a test run followed a behavior change (branch-level, time-based - labeled honestly, not per-line coverage).
- **Data sovereignty.** Truth is plain text in `.wdd/`. `rm -rf` the tool and everything stays readable with cat/jq/grep.

## Conventions

- TypeScript strict, `noUncheckedIndexedAccess`. Prefer `interface` over `type`.
- Comments only for constraints the code can't show; code should be self-documenting.
- Source is English-only. Ledger prose follows the user's language; machine-matching uses the distiller's language-neutral `tags`, never prose keywords (English regexes are a pre-tag fallback).
- Plain dash `-`, never the em dash.
- Keep tests tiny and behavior-focused; they sit beside the code as `*.test.ts` (excluded from the build).
- Task recipes live as skills in `.claude/skills/` (`add-command`, `add-signal`, `workbench-ui`) - follow them instead of re-deriving the wiring.

---
> Source: [zhu-kai/whodecided](https://github.com/zhu-kai/whodecided) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
