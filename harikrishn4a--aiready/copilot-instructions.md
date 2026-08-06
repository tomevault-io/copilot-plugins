## aiready

> > Note: aiready audit recognises AGENTS.md, CLAUDE.md, AGENT.md,

> Note: aiready audit recognises AGENTS.md, CLAUDE.md, AGENT.md,
> .cursorrules, and .windsurfrules as valid entry points.
> AGENTS.md is recommended for broadest agent compatibility.

# AGENTS.md — AIReady

## What this is
CLI tool that audits repositories for AI agent readiness and generates
harness artifacts so coding agents can work effectively across sessions.
Distributed via npx — zero install required.

## Product stages

| Stage | Command | What it does | Status |
|---|---|---|---|
| 1 | `npx aiready audit` | LLM-powered audit — scores 5 harness subsystems, writes `plan/plan.md` | **complete** |
| 2 | `npx aiready init` | Reads plan.md + source context, generates missing harness artifacts | **current** |
| 3 | `npx aiready analyze` | Reads code + Graphify graph, finds undocumented intent, writes `.aiready/gaps.md` | **complete** |
| 4 | `npx aiready drift` | Reads harness + git history, finds stale docs, writes `.aiready/drift.md` | not started |
| 5 | `npx aiready fix` | Reads plan/gaps/drift, patches exactly what's wrong, shows diff before write | not started |

See `docs/ARCHITECTURE.md` for full design detail on each stage.

## Current stage
Stage 3 complete — `npx aiready analyze` — next: Stage 4 `npx aiready drift`

Stage 3 walks all source files, runs a structural string-match pass (no LLM)
to find modules not mentioned in any harness doc, then runs a semantic LLM
pass (one call per relevant file) to find inadequately documented behavior.
Writes `.aiready/gaps.md` with proposed language-appropriate doc blocks per gap.

## Stack
- Node.js 20+, TypeScript (strict)
- Commander.js — CLI framework
- Vitest — testing
- tsup — build/bundle to dist/
- ESLint — linting
- `@anthropic-ai/sdk`, `openai` — LLM providers (Anthropic, OpenAI, Ollama)
- `@inquirer/prompts` — interactive provider/model selection
- `dotenv` — `.env` file loading

## Repo structure
```
aiready/
  src/
    audit/           ← Stage 1: LLM audit pipeline
    init/            ← Stage 2: artifact generation
    analyze/         ← Stage 3: semantic gap analysis (complete)
    drift/           ← Stage 4: drift detection (stub)
    fix/             ← Stage 5: auto-remediation (stub)
    cli.ts           ← Commander entrypoint
    utils/
  examples/          ← harness templates (reference for Stage 2 generation)
  tests/
  AGENTS.md          ← this file (repo root entry point)
  feature_list.json  ← feature tracker (repo root)
  docs/
    ARCHITECTURE.md
    PROGRESS.md
    SESSION-HANDOFF.md
    DECISIONS.md
    TASK.md
    features.md
    structure.md
```

See `docs/structure.md` for artifact ownership and layout rules.

---

## Session start
1. Run `pwd` — confirm you are in the repo root
2. Read this file completely
3. Read `docs/PROGRESS.md` — understand current state
4. Read `docs/SESSION-HANDOFF.md` — see what the last session left
5. Run `git log --oneline -5` — see recent changes
6. Run `npm run build && npm test` — confirm baseline is not broken
7. Read `feature_list.json` — identify the current active feature
8. Pick exactly one unfinished feature. Work only that until verified or blocked.

If baseline verification is failing, repair that first before adding new scope.

## Session end
1. Run full verification (see Verification Commands below)
2. Update `docs/PROGRESS.md` if a feature completed, was added, or got blocked
3. Update `feature_list.json` — set new status and record evidence
4. Overwrite `docs/SESSION-HANDOFF.md` with this session's state
5. If a key decision was made, append it to `docs/DECISIONS.md`
6. **Update `docs/ARCHITECTURE.md`** if any of the following changed:
   - A new module was added or renamed
   - A stage pipeline changed
   - A layer boundary rule changed
   - The product stage table changed (status, command, description)
7. Commit with a descriptive message — leave a clean restart path

## Working rules
- One active feature at a time — never work on two features in parallel
- Before starting a feature, generate a sprint contract and save it to `docs/TASK.md`
- Do not claim completion without runnable verification evidence
- Do not rewrite `docs/PROGRESS.md` to hide unfinished work
- Do not remove or weaken tests to make a task appear complete
- Stay in scope — do not modify files unrelated to the current feature

## Completion gate
A feature moves to `passing` only when ALL of the following are true:
- [ ] Target behavior is implemented
- [ ] All verification commands pass (see below)
- [ ] Evidence recorded in `feature_list.json`
- [ ] Repository is restartable from `npm run build && npm test`

## Verification commands
```bash
npm run build     # tsc + tsup, must emit zero errors
npm run typecheck # tsc --noEmit --strict
npm run lint      # eslint src/, must be clean
npm test          # vitest, must pass
```

## Escalation
- **Architecture decisions**: Check `docs/DECISIONS.md`, then ask the user
- **Unclear requirements**: Check `docs/SESSION-HANDOFF.md` and `docs/TASK.md`, then ask the user
- **Repeated failures**: Mark feature as blocked in `feature_list.json`, flag for human review
- **Scope ambiguity**: Re-read `docs/TASK.md` sprint contract before expanding scope

## Constraints — never do these
- MUST keep `AGENTS.md` at the repo root — MUST NOT move it to `docs/`
- MUST store all other harness markdown under `docs/` (e.g. `docs/ARCHITECTURE.md`, `docs/PROGRESS.md`)
- MUST NOT write to the target repo without `--force` or explicit user confirmation
- MUST NOT run verification commands in the target repo automatically
- MUST NOT overwrite existing harness files in the target repo unless `--force` is passed
- MUST NOT add new dependencies without recording the decision in `docs/DECISIONS.md`
- MUST NOT claim completion without runnable evidence
- MUST NOT implement a future stage while the current stage is in progress
- MUST NOT let `docs/ARCHITECTURE.md` drift — update it at session end when anything structural changes

## Error message format
All errors emitted by the CLI must follow this structure:

```
ERROR: <what was found and where>
WHY:   <why this is a problem>
FIX:   <exact action to resolve it>
```

Example:
```
ERROR: Verification command `npm run typecheck` in AGENTS.md not found in package.json scripts
WHY:   Agent will try to run a command that does not exist, wasting context on a fixable error
FIX:   Add "typecheck": "tsc --noEmit" to package.json scripts, or update AGENTS.md to match
```

---
> Source: [harikrishn4a/aiready](https://github.com/harikrishn4a/aiready) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
