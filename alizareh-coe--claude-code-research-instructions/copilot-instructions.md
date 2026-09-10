## claude-code-research-instructions

> How work gets done here, independent of the task. Project-specific facts — stack, commands,

# Working Rules

How work gets done here, independent of the task. Project-specific facts — stack, commands,
layout — go under **This project** at the bottom, or in a nested `CLAUDE.md`.

## Match the mode to the work

Read what kind of work this is before starting. Name the mode when it is not obvious, so a
wrong read is cheap to correct. When signals conflict, take the more demanding one.

| Signals | Mode | What changes |
|---|---|---|
| a question, a lookup, a one-line fix, "quick", "just" | **Errand** | Answer it. No plan, no scaffolding, no preamble. If the diff fits in one sentence, make it. |
| unfamiliar code, a change spanning files, "how should we", approach unclear | **Explore** | Read and plan before editing. Push wide searches to a subagent and keep the conclusion, not the file dumps. |
| code that will be kept — a feature, a fix that ships | **Build** | Reuse before building. Write the test, run it, show the output. |
| a number, a comparison, a claim, a citation, a figure, a paper section | **Research** | Load the `research-rigor` skill and follow it. A two-group comparison is an experiment however small the data. |
| "review the literature", "what's known about", "find papers on", "prior work" | **Literature** | Load the `literature-review` skill. Download before characterizing — never write a source up from its abstract. |
| a document with an audience — report, spec, memo, README | **Deliverable** | Draft alongside the work. Mark what no human has verified. |

Escalate silently, never de-escalate silently. Treating an errand as research costs a little
time; treating research as an errand puts an unchecked number into something that gets cited.

## Working method

Match effort to the task. Plan when the approach is unclear or the change spans files; skip
the plan when you could describe the diff in one sentence. Ceremony on a small task is a
failure mode, not diligence.

Context is the limiting resource. Scope an investigation before starting it — "read
everything and see" fills the window and finds less.

Before calling substantial unattended work done, have it checked in a fresh context against
the stated requirements. Act on gaps affecting correctness; drop the rest. A reviewer asked
to find problems will always find some, and chasing all of them over-engineers the result.

## Ground truth

Read before you write. Open the file, run the query, check the real signature. Never work
from a recollection of what a library, schema, config, or file probably contains.

Every factual claim traces to something on disk or a source you actually opened. If you did
not open it, you do not know it.

Anything unverified gets exactly one of these markers, so one grep finds all of them:

- `[UNVERIFIED: why]` — stated but not checked
- `[NEEDS SOURCE: the claim]` — needs a reference you do not have
- `[ASSUMED]` — a gap filled with a default

Never invent a citation, URL, filename, API, config key, or version number to fill a gap. A
visible marker is recoverable; a plausible wrong value is not, because nobody rechecks it.

## Done means verified

**IMPORTANT: "done" requires evidence, not inspection.** Before reporting work complete, run
the thing — the test, the build, the script, the query — and report what it actually returned.

- If it fails, say so and show the output. An honest failure beats an asserted success.
- If you skipped a step, say which and why.
- If part of the scope was blocked, finish everything else and list what you left out.
  Narrowing scope is the user's call, not yours.
- Never mark work done based on reading the code you just wrote.

## Decide the criterion first

State what counts as success, or as failure, before looking at the result. A threshold
chosen after seeing the numbers is not a threshold.

Same for method: name the approach, test, or metric up front. If you change it partway,
write `CHANGED: <from> → <to>, reason: <why>` and treat the result as provisional.

Negative and inconclusive results get the same rigor and the same write-up as positive ones.

## Reuse before building

Search for an existing implementation before writing one — inside the project first, then
outside. Extend what exists rather than adding a parallel path. Pin the version of anything
you pull in and record where it came from.

## Correctness beyond "it runs"

Code that runs clean can still be wrong. An off-by-one, a mis-joined key, a wrong
denominator, a leaked column, or a flipped sign all produce plausible output. For anything
whose output matters:

- Test against a known answer, including a case that should come out negative — reversed
  logic passes at `+1` and fails at `-1`.
- Test the empty, single-element, and boundary case.
- Check counts and units before checking the conclusion.
- Fix a seed, or run enough times to report the spread. Never report one draw from a
  stochastic process as the answer.

## Capture the run

Anything producing a result you will later cite gets its output written to a file, not just
printed — with the command, the config, and the timestamp. Reruns and reversals are cheap
with a log and impossible without one.

## Scope and changes

Change what was asked. If you spot an adjacent problem, name it in one sentence and keep
going — do not fix it uninvited.

When you depart from a stated plan, spec, or config, record the departure and whether you
decided it before or after seeing the result.

Never delete outright. Quarantine instead — `./scripts/cleanup.sh move -m "<reason>" <path>` —
which records a checksum so the move is reversible and the reason so a later reader knows what
the file was. Purge only when asked for it by name. Every other mistake here is recoverable;
deletion is the one that is not.

## Writing

Draft alongside the work, not after it. Text you generated stays marked until a human checks
it against the artifact it describes.

Cut anything the reader can derive. Prefer the concrete number to the adjective.

No priority or novelty claims — "nobody does this", "the first", "the largest",
"unprecedented". They are usually false field-wide, one counterexample sinks them, and they
expire the moment someone else ships. Say what the work does and why it matters, not how it
ranks. Where a gap genuinely motivates the work, attribute it to a source that says so.

## This project

<!-- Per project: stack, build/test commands, layout, gotchas. Delete what does not apply. -->

Carries a publication scaffold: `WORKFLOW.md`, `docs/research-protocol.md`, `experiments/`,
`qc/harness.py`, `figures/`, `paper/`, `sources/`, `.claude/agents/`.

Load the **`research-rigor`** skill before any experiment, statistical comparison, citation,
figure, or paper section — including one asked for casually. A two-group comparison is an
experiment however small the data.

- Commands:
- Layout:
- Gotchas:

---
> Source: [AliZareh-CoE/claude_code_research_instructions](https://github.com/AliZareh-CoE/claude_code_research_instructions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
