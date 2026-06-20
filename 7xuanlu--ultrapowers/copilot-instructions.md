## ultrapowers

> Shared, tool-agnostic guidance for any coding agent (Claude Code, Codex, Gemini CLI, …) that

# AGENT.md, operating manual for agents working on ultrapowers

Shared, tool-agnostic guidance for any coding agent (Claude Code, Codex, Gemini CLI, …) that
contributes to **this repo**. Claude Code imports it via `CLAUDE.md`. It is about *building
ultrapowers itself*, not about running the harness on a downstream project.

## What this repo is

ultrapowers is a Claude Code **Workflow**, a deterministic JavaScript coordinator
(`workflow/ultrapowers-development.js`) that hosts Superpowers' SDD/TDD discipline on disposable
subagents: plan → per-task (implement under strict TDD → deterministic gate → re-witness-RED →
two-stage fail-closed review → fix-loop) → dry-until-clean critic → final integration review.
The coordinator makes **zero LLM calls**; all work tokens are paid by subagents. The
`/workflows-driven-development` command owns the two human gates and *dispatches* the engine, it
must never re-implement the loop.

## The discipline bar (eat our own dog food)

This is a tool that gates *other people's* code on TDD and adversarial review. Contributions are
held to the same bar.

- **TDD where it applies**, code/logic changes follow red → green → refactor. Write the failing
  test first, watch it fail, make it pass. Docs, JSON manifests, and CI YAML have no unit test;
  verify them by validity (parses / lints) and by reading, not by skipping verification.
- **re-witness-RED is the headline mechanism**, strip a task's production files, re-run the
  suite, confirm it fails without them. Never weaken it. Its proof lives in `tests/re-witness-red/`.
- **Surgical changes**, touch only what the task needs; match existing style; no "while I'm here"
  refactors. No speculative surface (no flags/abstractions/error-handling beyond what's asked).
- **Verify before claiming done**, run `npm run check`; for engine behavior, run the re-witness
  self-test. Evidence before assertions.

## Verification commands

| command | what it does | cost |
|---------|--------------|------|
| `npm run check` | parses the engine the way the runtime loads it (strips `export`, compiles as async IIFE) | free, no deps |
| `npm run test:rewitness` | replays the re-witness-RED catch path through Claude Code | **spends tokens / needs auth** |

CI runs `npm run check` + JSON-manifest validation only. The re-witness self-test is a
runtime/manual gate (it cannot run in stock CI).

## Things that are load-bearing, do not casually change

- **`SP_VERSION = '5.1.0'`** (`workflow/ultrapowers-development.js:88`) pins the Superpowers
  version the embedded prompts (TDD + the two reviewer briefs) are synced to; the engine logs
  `SP DRIFT` if the installed Superpowers differs (`:682`). If you edit an embedded prompt, keep
  it synced to this pin and update the pin deliberately.
- **The two-stage review is fail-CLOSED**, spec-compliance THEN code-quality, blocking derived
  from the findings, not the model's `approved` boolean. Don't make it fail-open.
- **Default implementer**: the product (`/workflows-driven-development`) passes `implementer:"claude"`,
  which needs no external CLI. The engine's raw fallback is `codex` (`:94`), only reached when the
  engine is dispatched directly with no `implementer` arg.

## Editing the engine, footguns

The engine (`workflow/ultrapowers-development.js`) is a Workflow script, not a normal module.
Things that bite:

- **`args` arrives as a JSON string, not an object.** The engine parses it once into `_args`
  (`:80-81`); read new args from `_args`, never assume an object.
- **`tasks` must be `[{id,spec}]` objects.** Bare strings are dropped; the engine validates
  fail-loud (incl. empty/whitespace `id`/`spec`). Format + footgun: `reference/task-list.md`.
- **`meta.phases` declares only `Plan` + `Preflight` on purpose** (`:13-16`). Per-task `Build`
  and final `Integrate` are *dynamic* progress groups (`task:<id>`, then `Integrate`) so
  /workflows renders in true execution order; declaring them statically breaks that. Don't.
- **Workflow-script constraints:** plain JS (no TypeScript syntax); `Date.now()` /
  `Math.random()` / argless `new Date()` are unavailable (they break resume). Pass timestamps via
  args; vary by index for randomness.

## Security, unattended code execution

The harness writes files, runs `verifyCmd`, and commits, across subagents, with no human in the
loop between gates. External implementers (`codex`/`gemini`) run **unsandboxed** and require
`Bash(codex *)` allow + `sandbox.excludedCommands:['codex']` (`:316`). Any change touching
execution, the sandbox carve-out, or worktree/branch isolation must keep `SECURITY.md` accurate,
that doc is the threat model and is the most dangerous thing to let drift.

## Attribution, non-negotiable

ultrapowers **complements** [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent
([@obra](https://github.com/obra)); it does not replace it. The SDD/TDD discipline and the
embedded prompts are obra's, adopted verbatim. Keep `NOTICE` + `LICENSE-superpowers` accurate when
you touch any embedded file, keep obra credited in the plugin manifest, and never let positioning
drift into "ultrapowers > superpowers."

## Honesty in claims (esp. benchmarks/README)

Tag factual claims (`[V src]` verified, `[I]` inferred, `[ESTIMATE: calc]`, `[U]` unknown). Never
claim a per-bill cost discount (the measured N=5 head-to-head was a tie); the flat coordinator is a
**scaling/capability** property, not a saving. Projections are marked PROJECTED, not measured. The
honest comparison is the moat, marketing that erodes a qualifier is a regression, not a feature.

---
> Source: [7xuanlu/ultrapowers](https://github.com/7xuanlu/ultrapowers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
