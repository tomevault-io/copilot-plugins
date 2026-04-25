## mythos-router

> Defines architecture, constraints, and execution guarantees for mythos-router.

# AGENTS.md — mythos-router Project Standards

Defines architecture, constraints, and execution guarantees for mythos-router.

## Project Identity
- **Name**: mythos-router
- **Type**: CLI power tool (local-first, zero-slop)
- **Stack**: TypeScript on Node.js 20+ (ESM, `tsx` for dev)

## Architecture
- `src/cli.ts` — Commander.js entry point
- `src/config.ts` — Constants, system prompt, validation, budget defaults
- `src/client.ts` — Anthropic SDK wrapper (adaptive thinking)
- `src/budget.ts` — Session budget limiter (token cap, turn cap, progress bar)
- `src/swd.ts` — Strict Write Discipline engine + dry-run preview
- `src/memory.ts` — Self-healing MEMORY.md manager (dry-run aware)
- `src/utils.ts` — Terminal colors, spinner, formatting, badges, confirm prompt
- `src/commands/chat.ts` — Interactive REPL (budget + dry-run + verbose)
- `src/commands/verify.ts` — Codebase ↔ Memory drift scanner (dry-run aware)
- `src/commands/dream.ts` — Memory compression (dry-run aware)

## Conventions
1. **Zero external runtime deps** beyond `@anthropic-ai/sdk` and `commander`
2. **No `chalk`, no `ink`** — all terminal output uses native ANSI for zero overhead
3. **ESM only** — `"type": "module"` in package.json
4. All file operations use `node:fs` (sync) to ensure deterministic SWD verification
5. **SWD is non-negotiable** — every model output must match real filesystem state
6. **MEMORY.md is sacred** — never delete it, only append or compress via Dream
7. The system prompt lives in `config.ts` — do NOT scatter prompt fragments
8. **Budget defaults live in `config.ts`** — 500K tokens, 25 turns, 80% warning
9. **Pricing constants live in `config.ts`** — update `COST_PER_INPUT_TOKEN` / `COST_PER_OUTPUT_TOKEN` when Anthropic changes rates
10. **Dry-run mode** — all filesystem writes must check `dryRun` flag before mutating

## File Operation Protocol (Core Safety Layer)

This protocol ensures that model outputs are not trusted blindly and must match real filesystem state.

- All file operations must be explicitly declared by the model
- Wrapped in `[FILE_ACTION: path]...[/FILE_ACTION]` blocks
- SWD parses and verifies against actual filesystem state
- Max 2 correction retries before yielding to human
- In `--dry-run`, actions are previewed instead of executed

## Budget Limiter Protocol
- `SessionBudget` tracks tokens + turns + estimated cost per session (not persisted across runs)
- Pre-check before every API call — triggers graceful save at limit (progress → MEMORY.md)
- Warning at 80% consumption
- `--no-budget` disables for expert users
- Correction turns count toward the budget

## Running
```bash
# Dev mode (no build required)
npx tsx src/cli.ts chat
npx tsx src/cli.ts chat --dry-run --verbose
npx tsx src/cli.ts chat --max-tokens 100000 --max-turns 10
npx tsx src/cli.ts chat --no-budget
npx tsx src/cli.ts verify
npx tsx src/cli.ts verify --dry-run
npx tsx src/cli.ts dream
npx tsx src/cli.ts dream --dry-run

# Or via npm scripts
npm run chat
npm run verify
npm run dream
```

## Invariants

- The filesystem is the single source of truth
- Model output is never trusted without verification
- No file mutation occurs without successful SWD validation

---
> Source: [thewaltero/mythos-router](https://github.com/thewaltero/mythos-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
