## polyflow

> >-


# Workflow Creator

Turn a goal into a **runnable workflow file** — a JavaScript orchestrator for
Claude Code's `Workflow` tool.

A workflow fans work out to fresh-context subagents under **deterministic
JavaScript** control flow: the loops, the conditionals, the fan-out are plain
code, and only the leaf `agent()` calls spend model tokens. The Workflow tool is
new and undocumented, so this skill carries the format, the judgment calls, and a
tested authoring procedure. Use it to write a new workflow, convert a multi-step
job into one, fix a broken script, or explain the format — the `meta` block,
`agent()`/`parallel()`/`pipeline()`/`phase()`, schemas, the determinism rules —
when the user is confused about workflows or a workflow errors.

The deep material lives in two reference files — read them when the step says so:

- `references/api-reference.md` — the complete manual: every global, every option,
  every cap and constant, what happens at each limit.
- `references/patterns.md` — copy-paste orchestration patterns (fan-out, pipeline,
  loop-until-budget, adversarial verify, judge panel, nested workflow).

Starter files are in `assets/templates/`. Six complete, runnable example
workflows are in `assets/examples/` — `assets/examples/README.md` maps each one
to a topology and to the model / structured-output techniques it shows. A linter
is in `scripts/`.

---

## Stability

This skill is pinned to **Claude Code 2.1.149** — the binary against which the
Workflow tool's internals (globals, caps, journal format) were last verified.
Two gates control runtime availability:

- **`CLAUDE_CODE_WORKFLOWS=1`** (env var, user-controlled). See Step 0.
- **`tengu_workflows_enabled`** (Statsig flag, account-controlled). Even with
  the env var set, the tool stays hidden if the flag is off for the user's
  account — surface that possibility if `/workflows` does nothing after the
  export.

**Break-glass.** If a global, cap, or option in `references/api-reference.md`
does not match a runtime error, re-read the manual section and verify via a
one-line `Workflow({ scriptPath })` smoke test against a known-good script — do
not invent behaviour. After a Claude Code upgrade, expect the version pin to
trail the binary until this section is updated.

---

## Step 0 — Confirm the Workflow tool is available

A workflow can only **run** if the Workflow tool is enabled. It is **off by
default**, gated behind an environment variable. The file is always worth
*writing* — check this so the user hears the truth about *running* it:

```bash
echo "${CLAUDE_CODE_WORKFLOWS:-<not set>}"
```

If it is not set, the workflow file is still worth writing — but tell the user
they must enable the tool before it will run, either of:

```bash
# per session
export CLAUDE_CODE_WORKFLOWS=1 && claude
```

```jsonc
// or persistently, in .claude/settings.local.json
{ "env": { "CLAUDE_CODE_WORKFLOWS": "1" } }
```

Workflow files live in `.claude/workflows/<name>.js` (project-local) or
`~/.claude/workflows/<name>.js` (global). The filename is not the workflow name —
the `name` inside the `meta` block is.

---

## Step 1 — Decide whether a workflow is even the right tool

Do not reach for a workflow by default — it is the heaviest option and it is
gated for a reason (it can spend a lot of tokens). Pick deliberately:

| The job | Right tool |
|---|---|
| One subagent, one task | The plain **`Agent`** tool — no workflow |
| A reusable procedure where **Claude** picks the steps each run | A **Skill** |
| Open-ended debugging, novel problem solving, or dynamic exploration | A **conversational agent** or **ReAct loop** |
| Many subagents in a **fixed** shape (fan-out / pipeline / loop), same every run, worth resuming | A **Workflow** ✅ |

**Optimal Use Cases:** Workflows excel at repeatable, scalable processes like "Review PR across 5 dimensions", "Extract themes from 1,000 feedback tickets", or "Sweep codebase for dead code".
**Anti-Patterns:** Do not use workflows for dynamic, open-ended tasks where the steps cannot be predicted upfront, like "Fix this vague build error" or "Set up a new database schema based on a loose PRD".

A workflow earns its cost when **all** of these are true: the work is parallel or
multi-stage; the orchestration must be deterministic and resumable; and
isolating each step in its own fresh context window is an advantage. When
unsure, say so and offer the lighter option instead.

---

## Step 2 — Find the shape of the job

Before writing a line of code, answer these — the answers pick the topology.

1. **What is the unit of work?** The thing one subagent does once — review one
   file, research one question, draft one platform. Name it concretely.
2. **How many units, and is the count known up front?** A known list → map over
   it. An unknown count (discovery, "find all the bugs") → a loop.
3. **What is the topology?**
   - Independent units, one pass each → **fan-out**.
   - Units flow through ordered stages (review → verify) → **pipeline**.
   - Keep going until a target count or a budget runs low → **loop**.
4. **Does any later step need *all* the earlier results at once** — to dedup,
   merge, count, or early-exit on a zero total? If yes, that needs a **barrier**.
   If no, it does not — prefer `pipeline`.
5. **Does a step need structured data back** (not free text)? Then that
   `agent()` call needs a `schema`.
6. **How will you verify the results automatically?** (Nyquist Validation Principle). Every workflow should incorporate a verification stage (e.g., executing code, running tests, or a skepticism loop) rather than trusting subagent output blindly.

Write these six answers down for the user before coding. They are the design.

Before writing JavaScript, copy `assets/templates/workflow-spec.template.md`
into a working `WORKFLOW-SPEC.md` (or equivalent scratch artifact), fill it in,
and get human sign-off on the topology and barrier decision. Run
`node scripts/validate-workflow-spec.mjs WORKFLOW-SPEC.md` and fix every error
before Step 4. Do not begin Step 4 until that spec exists and has been reviewed.

---

## Step 3 — The decision that matters most: `pipeline` vs `parallel`

This is the call people get wrong, so make it explicitly.

- **`pipeline(items, stage1, stage2, …)` is the default for multi-stage work.**
  Each item flows through every stage on its own — **there is no barrier between
  stages**. Item A can be in stage 3 while item B is still in stage 1.
  Wall-clock = the slowest single item's whole chain, not the sum of the slowest
  stage at each step.

- **`parallel(thunks)` is a barrier.** It waits for every task before returning.
  Reach for it **only** when a stage genuinely needs the *entire* previous
  result set in hand — dedup across all findings, merge, a count-based
  early-exit. "It is cleaner code" or "the stages feel separate" are **not**
  reasons — a pipeline models separate stages fine.

Smell test: writing `const a = await parallel(...)`, then a plain
transform (`flat`/`map`/`filter`) with no cross-item dependency, then another
`parallel(...)` — that middle transform does not need the barrier. Make it a
pipeline stage instead. **When in doubt, `pipeline`.**

Before moving on, record the Step 2 answers and this Step 3 decision in the
workflow spec. The spec must explicitly say whether there is a barrier and why.

---

## Step 4 — Write the file

A workflow file has exactly two parts, in this order. The parser is strict.

### Part 1 — the `meta` block (must be the very first statement)

```js
export const meta = {
  name: 'review-changes',                         // required, non-empty
  description: 'Review changed files, verify each finding', // required — shown in the permission dialog
  whenToUse: 'Before shipping a branch',          // optional — shown in the workflow list
  phases: [                                       // optional — one entry per phase() call
    { title: 'Review' },
    { title: 'Verify', model: 'haiku' },
  ],
}
```

`meta` **must be a pure literal** — no variables, function calls, spreads, or
template strings inside it. Build dynamic values in the body, never in `meta`.
Use the same phase `title` strings in `meta.phases` as in the `phase()` calls.

### Part 2 — the body (async JavaScript)

Everything after `meta` is the body. It runs inside an `async` function — `await`
at the top level. A fixed set of globals is injected; **import nothing**:
`agent`, `pipeline`, `parallel`, `phase`, `log`, `console`, `budget`, `args`,
`workflow` (plus an injected `setTimeout`/`clearTimeout` pair). The body's
`return` value becomes the tool result handed back to Claude.

**Read `references/api-reference.md` §5 now** for every global's full signature
and the `args` normalizer. The one trap to remember inline: `args` arrives
**exactly as passed** — an object stays an object, a string stays a string,
nothing passed is `undefined` — so never call `JSON.parse(args)` unconditionally;
parse only when `typeof args === 'string'`.

### Setting a model, and getting structured data back — the two `agent()` opts to tune most

**Model — `agent(prompt, { model })`.** Each agent call runs on its own model.
Accepts `'haiku'`, `'sonnet'`, `'opus'`, `'inherit'`, or a full model ID; omit it
and the agent inherits the session's model. Drop **cheap, high-volume,
mechanical** leaf work (per-item summaries, refute-this checks, classification)
to `'haiku'`; leave judgement-heavy work on the inherited model. Two cautions:

- **There is no validation.** A typo (`'hauku'`) is not rejected — it is passed
  through and the agent fails later. Spell the alias exactly.
- **`model` on a `meta.phases[]` entry does nothing at runtime** — it is a label
  for the permission dialog only. The model is set *solely* by the `model` opt on
  the `agent()` call. For a Haiku phase, set `model` in *both* places: the phase
  entry (honest dialog) and every `agent()` call in it (actual effect).

**Structured output — `agent(prompt, { schema })`.** Without `schema`, the call
returns the agent's final text as a **string**. Pass a JSON Schema and the agent is
*forced* to return a **validated object** matching it — the runtime builds a hidden
`StructuredOutput` tool from the schema, AJV-validates the result, and makes the
agent retry on a mismatch. `agent()` returns the parsed object directly — no
`JSON.parse`. Use `schema` for any result a later line of JavaScript reads a
field off of; keep schemas small and `required`-tight. To pass data *between*
stages, stringify it into the next prompt (`JSON.stringify`) — the orchestrator
shares no memory with the subagent, only the prompt text.

### Keeping inter-stage context lean

Every string you embed in an `agent()` prompt burns input tokens — the
orchestrator itself spends zero. Token cost lives entirely in `agent()` calls
and their prompts. Six habits that cut that cost:

**1. Project before you stringify.** Strip fields the next stage does not read.
`results.map(({ title, file }) => ({ title, file }))` before the stringify — not
`results` itself.

**2. No indent for inter-stage payloads.** `JSON.stringify(x, null, 2)` adds
whitespace the subagent never reads. Use the compact form for data passed between
agents; reserve pretty-printing for the final human-facing return value.

**3. Sliding window in discovery loops.** A growing `seen` list sends O(n)
context per round. Cap it: `[...seen].slice(-30).join('\n')` is enough to avoid
re-finds without sending the full history.

**4. Tight schemas.** Add `additionalProperties: false` at every schema level so
the model emits only declared fields; add `maxItems: N` where the count is
bounded. See the schema discipline section in `references/patterns.md`.

**5. Use `pipeline`'s `originalItem` argument.** Stage callbacks receive
`(prevResult, originalItem, index)` — use `originalItem` in later stages so
stage 1 returns a lean schema instead of copying source data into its return.
Example: `(findings, file) => agent('Verify ' + file + '…')` re-uses the
original `file` directly; `findings` stays small.

**6. Two-tier model routing.** Route mechanical fan-out (extraction,
   classification, binary refutation) to `model: 'haiku'`; leave synthesis and
   judgement on the inherited model. Fan-out stages are the highest-volume and
   usually the least demanding. See pattern 12 in `references/patterns.md`.

### Externalizing State & Artifacts

The orchestrator cannot access the filesystem directly (`fs` and Node APIs are banned). Because of this sandbox constraint, you cannot persist intermediate data or state directly from the JavaScript body. 

To externalize workflow state, intermediate progress, or final outputs (e.g., a `STATE.json` or `.planning/` directory structure), design your `agent()` calls to explicitly run file writing or command execution tools (e.g., using `write_file`, `git commit`) to persist state onto the host filesystem. This prevents session context loss, mitigates context-passing bloat on long discoveries, and allows external tools or agents to audit progress.

If a workflow needs memory across runs, use a scoped memory contract instead of
a shared live memory layer: perform a read-only recall stage before the main
work, optionally persist a compact summary after the work finishes, and keep the
first implementation file-based so the state is inspectable and replay-safe. If
a semantic index is needed later, put it behind the same recall/persist contract
instead of changing the workflow shape. The default artifact pair is
`assets/templates/memory-entry.schema.json` plus `.planning/memory/index.jsonl`.

For full signatures, every option, and every cap, **read
`references/api-reference.md` now.** For ready-made orchestration shapes, **read
`references/patterns.md`** and copy the one that fits Step 2's answers. Or start
from a file in `assets/templates/`, or adapt a full worked example from
`assets/examples/` (its `README.md` says which one fits).

---

## Step 5 — Validate before running

Catch the parser's hard rules before wasting a run. Use the bundled linter:

```bash
node ${CLAUDE_SKILL_DIR}/scripts/validate-workflow.mjs <path-to-file.js>
```

It flags: missing or non-first `meta`, a non-literal `meta`, a missing
`name`/`description`, banned non-deterministic calls, and an oversized script.
Fix every error it reports before invoking the workflow.

---

## Step 6 — Run, watch, iterate

Run a named workflow with `Workflow({ name: 'review-changes' })`, or a file with
`Workflow({ scriptPath: '…' })`. It runs in the **background** — the call returns
a run ID immediately and a `<task-notification>` arrives on completion. Watch it
live with the `/workflows` command, which can also skip or retry a single
agent mid-run.

To iterate: **edit the saved file**, then re-invoke with
`Workflow({ scriptPath, resumeFromRunId })`. Every `agent()` call before the
first edit replays instantly from cache; only the changed call and everything
after it re-runs. Same script + same args = a 100% cache hit. Never re-paste the
whole script after the first run — edit the file.

---

## Gotchas — check every one before handing over the file

These are the mistakes that actually break workflows:

- **No coding before the workflow spec.** Fill out the workflow design spec,
  get human sign-off, then write the JS. If the topology or barrier choice is
  still fuzzy, you are not ready for Step 4.
- **Determinism bans.** `Date.now()`, `Math.random()`, and argless `new Date()`
  **throw** inside a workflow — they would break resume. Pass timestamps in via
  `args` and stamp results *after* the workflow returns; vary "randomness" by
  loop index instead. `new Date(specificValue)` is fine.
- **No filesystem, no Node APIs** in the orchestrator. No `require`, `fs`,
  `process`. Any file read/write/Bash work belongs **inside an `agent()`** — the
  subagent has the normal tools; the orchestrator does not. However, subagents *should*
  write files (e.g., logs, state documents) to externalize intermediate state to the host.
- **`parallel()` takes thunks, not promises.** It must be
  `[() => agent(...), () => agent(...)]`, never `[agent(...), agent(...)]`. Bare
  calls start immediately and defeat the concurrency limiter.
- **Always `.filter(Boolean)`.** `parallel()` and `pipeline()` put `null` in the
  slot of any item that threw, was skipped, or was dropped by the budget. The
  result arrays have holes by design; filter them out before doing downstream work.
- **`meta` is a pure literal and the first statement.** No dynamic values, no
  code before it.
- **Open-ended loops need a hard stop** — both a hard iteration counter (`while (rounds < 10)`) AND a token/budget guard (`while (budget.total && budget.remaining() > 50_000)`). If either is missing, the loop is unsafe and can run to the 1000-agent lifetime cap.
- **`isolation: 'worktree'` is expensive** (~200–500 ms + disk per agent). Use it
  only when parallel agents mutate files and would otherwise collide.
- **Inter-stage context bloat.** Passing a stage's full structured output to the
  next agent sends every schema field, including fields the agent ignores. Project
  first: `items.map(({ id, title }) => ({ id, title }))`. And drop the indent:
  `JSON.stringify(data)` not `JSON.stringify(data, null, 2)` — the spacing inflates
  large arrays by 20–40% with no benefit to the subagent.
- **Growing `seen` lists in discovery loops.** Re-sending the full history each
  round means round N pays for N−1 items of input context. Use a sliding window:
  `[...seen].slice(-30).join('\n')`.

### Scope discipline

- **YAGNI.** Add only what the current workflow needs. Do not add schema fields,
  phases, or branches "just in case".
- **Surgical changes.** When iterating on a workflow, edit the failing stage or
  the narrowest supporting context. Do not restructure unrelated stages.
- **Token cost is a design constraint.** Every `agent()` call has a cost. Name
  the model deliberately, keep prompts lean, and project structured output
  before passing it to another stage.

---

## Worked example — review a branch, verify each finding

The canonical worked example is `assets/examples/review-branch.js`: fan out one
reviewer per dimension, then — the instant a dimension's review returns — fan out
a verifier per finding. Read it next to `assets/examples/README.md`, which maps
it to its topology and the model / structured-output techniques it demonstrates.

It uses `pipeline` rather than `parallel` on purpose: a finding should verify the
moment *its own* review is done, so dimension `bugs` can be verifying while `perf`
is still under review — no waiting for the slowest dimension, each agent reasons
from a clean context, and the orchestrator JavaScript spends zero model tokens.

---

## When the user wants to learn, not just get a file

If the request is "explain how workflows work" rather than "build me one", walk
them through `references/api-reference.md` — it is written to be read top to
bottom as the missing manual. Then offer to scaffold their first workflow from a
template so they have something runnable to poke at.

---
> Source: [hiranp/polyflow](https://github.com/hiranp/polyflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
