## loops

> `loops` is a small, nestable job primitive for driving AI agents in **convergence loops**: do a bit of work with a fresh context, check whether the work is *actually* done against a gate you define, and if not, go again. The user-facing tour is `README.md`; this file is the operating guide for an agent working *on* the library.

# loops — agent guide

`loops` is a small, nestable job primitive for driving AI agents in **convergence loops**: do a bit of work with a fresh context, check whether the work is *actually* done against a gate you define, and if not, go again. The user-facing tour is `README.md`; this file is the operating guide for an agent working *on* the library.

`AGENTS.md` is a symlink to this file.

## Mental model (the one idea everything hangs off)

There is one universal unit of work and two supporting types:

- `Job = (ctx) => Promise<Outcome>` — a unit of work of any size.
- `Condition = (ctx, last) => Promise<{ met, reason, confidence? }>` — a yes/no gate.
- `Engine` — a one-method interface (`run(req, onEvent, signal)`) where an agent turn actually executes.

`loop()` returns a `Job`. `dag()` returns a `Job`. So loops and DAGs nest both ways: a DAG node can be a loop, a loop body can be a DAG. **Nesting is the absence of a special case, not a feature.** Preserve that property in any change; do not add a node type that only works in one position.

Two design tenets that must survive every edit:

1. **Honest convergence.** "Ask the model if it is done" is the trap the library exists to avoid. A gate combines a deterministic signal (`commandSucceeds` — the tests really pass) with a separate judge (`agentCheck`). Hardening lives in `quorum` (k-of-n jury) and the judge's `dimensions` (opens on the geometric mean, so one weak dimension drags the verdict down). `toVerdict` treats a missing confidence as `0` (fail-closed). Never reintroduce a "yes ⇒ 1.0" default.
2. **The workspace is the state.** Progress accumulates on disk (files, git), so each iteration starts clean. The loop carries only thin bookkeeping. This is why the library deliberately does not do durable mid-run replay; that is an orchestration concern (embed a loops job inside Temporal/LangGraph/Mastra if you need it).

## Source map

```
src/api.ts            public surface — the package's only export ("." → ./src/api.ts)
src/index.ts          CLI entry (run via tsx)
src/cli.tsx           commander CLI: flags mode + `run <file>` definition mode
src/core/
  job.ts              the Job type + helpers (defineJob, agentJob, gateJob)
  loop.ts             loop(): start → body → until → review, with retry/delay/caps
  dag.ts              dag()/sequence()/parallel(): toposort, needs/optional/when,
                      bounded cross-stage kickback (re-run a dirty subgraph)
  condition.ts        commandSucceeds, agentCheck, quorum, all/any/not, predicate,
                      forgeChecks, toVerdict, the dimensional-judge geometric mean
  forge.ts            Forge seam (PR host): GhForge (gh CLI) + MockForge, arg-builders
  pr.ts               pushJob / pullRequestJob / mergeJob — keep the squash body a
                      consolidation of the branch so the Ledger survives a squash merge
  budget.ts           Budget + assertBudget (non-retryable BUDGET LoopError)
  context.ts          JobContext (iteration, state, lastReview, signal)
  errors.ts           LoopError + terminal status taxonomy
  types.ts            shared types
src/engines/
  engine.ts           Engine interface + EngineOptions (incl. permissionMode)
  claude-cli.ts       `claude` subprocess via execa; exported buildClaudeArgs()
  agent-sdk.ts        @anthropic-ai/claude-agent-sdk
  anthropic-api.ts    @anthropic-ai/sdk (token streaming; cheapest for judges)
  mock.ts             scripted, offline — for tests/examples
  registry.ts         name → Engine resolution
src/runtime/
  runner.ts           the run() driver
  persist.ts          makeRecorder (JSONL), makeCheckpointer/loadCheckpoint
  hub.ts, signals.ts  event fan-out + abort plumbing
src/tui/              Ink TUI (App.tsx, model.ts, theme.ts)
src/reporters.ts      --no-tui (plain) and --json (NDJSON) reporters
bin/loops.mjs         the bin shim (see "Running" below)
examples/*.loop.ts    runnable definition files (mock engine = offline)
tests/                vitest
```

## Develop

No build step. The package imports TypeScript source directly through [`tsx`](https://github.com/privatenumber/tsx); `exports["."]` points at `./src/api.ts`.

```bash
npm test          # vitest — offline + deterministic via the mock engine
npm run typecheck # tsc --noEmit
npm run example:poll   # offline demo, no key
node bin/loops.mjs --help
```

Requires Node >= 20. `package.json` is `private: true` as a guard against accidental `npm publish` while the API is alpha; an npm release is on the README roadmap (flip `private` when releasing).

## Running .loop.ts files (the tsx loading model)

`loops run <file>` imports and executes a definition file's default-exported `Job`. `bin/loops.mjs` registers tsx's ESM loader **globally** before handing off to the CLI, so a `.loop.ts` is transformed **wherever it lives** — under this package's tree or in a consumer repo that has `loops` installed. `loops validate <file>` is the offline pre-flight: it loads and constructs the loop (catching syntax, import, transform, and bad-export errors) without running it, and reports a fix-oriented error so an authoring agent self-corrects before spending a turn.

One requirement survives: the recipe's folder must be an ES module scope (a `package.json` with `{"type":"module"}`) so this library's ESM-only dependencies (execa → unicorn-magic) resolve as ESM. Repos that consume `loops` as a submodule or dependency already have this. A load that fails with an ES-module error is missing that scope, and `loadJob`'s error says so.

The authoring guide an agent reads to compose a loop is `skills/author-loop/SKILL.md`. The run-from-anywhere contract is covered by `tests/run-anywhere.spec.ts` (it spawns the bin against a consumer-shaped out-of-tree recipe).

## Consumption

Designed to be consumed as a **git submodule** by a parent repo (source import, no published build needed), with an eventual npm release. A parent recipe imports the public surface from `src/api.ts` and registers its own engines/conditions; it never reaches into `src/core` internals. Keep `src/api.ts` the single seam.

## Conventions

- Keep the core small; that smallness is the point. Resist adding configuration or node types that are not pulling their weight.
- Match the surrounding style. Conditions are pure where they can be; side effects live in engines and the runner.
- Evergreen language in all docs and comments (no "currently", "now", "recently"); use versions or explicit references where timing matters.
- Tests run offline against the mock engine. A change to convergence logic needs a deterministic test, not a live model call.

---
> Source: [jonny981/loops](https://github.com/jonny981/loops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
