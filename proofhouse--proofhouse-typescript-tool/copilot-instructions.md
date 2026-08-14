## proofhouse-typescript-tool

> Guidance for AI coding agents working in this repository. Read it alongside the per-tool documentation and any memory files the harness loads.

# Agent instructions

Guidance for AI coding agents working in this repository. Read it alongside the per-tool documentation and any memory files the harness loads.

## Commit messages

Draft every commit message in `COMMIT_AGENTMSG` at the repo root before you run `git commit`. A gitignore entry keeps that file out of history, so it serves purely as a scratchpad for iterating on the message. The workflow goes like this.

1. Write the full message (subject, body, and trailers) to `COMMIT_AGENTMSG`.
2. Run `just lint-commit-msg` and resolve whatever it reports.
3. Commit the validated draft with `git commit -F COMMIT_AGENTMSG`.

`just lint-commit-msg` mirrors the commit-msg hook. Vale reads the message under the commit scope (which catches AI commit tells via `ai-tells-commits`). cspell reads it against the commit dictionary. commitlint checks the Conventional Commits shape, and commit-trailers checks trailer order. Running it while drafting surfaces problems early, rather than at the commit-msg hook where a late failure interrupts the commit.

The prek commit-msg hook on `.git/COMMIT_EDITMSG` stays the real gate. `COMMIT_AGENTMSG` and its recipe only preview that gate, so a clean recipe run predicts a clean commit but never replaces the hook.

## Coverage hints

Every line, branch, function, and statement of `src` sits under the floor, weighed per file. When the report calls a branch uncovered, a test is missing, and writing that test is the answer. Reach for a `v8 ignore` hint only where a line can't run under a test at all. Its shape never varies: `/* v8 ignore <kind> -- @preserve: <reason> */`. Dropping the `@preserve` marker lets the transform strip the comment before the provider ever sees it, and whoever reviews the change reads the reason that follows the colon.

Watch the syntax on two points. `v8 ignore next 3` looks like it names a count and doesn't: the provider skips one line and leaves the other two scored, so the gate stays green over a hole. A span of lines wants `v8 ignore start` and `v8 ignore stop` around it instead. Taking one arm of a conditional out of the denominator also wants the hint inside that arm, since a hint sitting over the `if` speaks to the statement instead of the branch it opens.

Reviewers reject a bare hint with no reason, the same way they reject an undocumented lint suppression. `src/cli.ts` carries the one file-level hint here and states its reason inline.

## Regression examples

The property suites under `tests/property` keep no record of what has already broken. fast-check draws its cases afresh on every run and forgets them when the process ends, so a counterexample survives only if a change carries it. When a property fails, the report names the shrunk value it settled on along with a seed and a path through the shrinking tree. Copy that value into an ordinary test beside the property, watch the test fail for the same reason, then fix the source.

Leave the property itself unseeded. Pinning a seed to hold a past failure trades away the search that found it, and the example test already holds that ground. The seed in a failure report is for reproducing the run while you work, not for committing.

## Prose lint output

The toolchain already defaults to the agent template. Both `just lint-prose` and the prek vale hook pass `--output=proofhouse-agent.tmpl`, so add the flag yourself only when invoking vale directly on specific paths. The template, synced from the proofhouse style package, prints one self-contained line per finding (location, severity, rule, the exact matched text, and the replacement parameter when the rule defines one) plus a totals line, so you can apply fixes without re-reading context through separate commands. Empty output means a clean run, and the exit code carries the result.

---
> Source: [proofhouse/proofhouse-typescript-tool](https://github.com/proofhouse/proofhouse-typescript-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
