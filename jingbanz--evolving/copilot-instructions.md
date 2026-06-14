## evolving

> Guide for any Claude session working in this repo. Read this first.

# CLAUDE.md

Guide for any Claude session working in this repo. Read this first.

## What Evolving is

A CLI tool that runs a fixed **Scout → Worker → Gatekeeper** pipeline in a loop to autonomously iterate on a codebase toward a user-defined goal. Every cycle ends in `keep`, `discard`, or `idle` — a kept cycle is one git commit; everything else leaves no trace. Humans evolve the agent prompts; agents evolve the project.

**Status:** draft specification. Implementation has started (`src/`) but is early — currently a CLI entry point, a loop detector primitive, and the surrounding cycle scaffolding.

## Repo layout

```
CLAUDE.md         You are here. Project identity + universal rules.
README.md         Public-facing project description.
LICENSE           Apache-2.0.
src/              Implementation.
  cli.ts          CLI entry point.
  agent-runtime/  Agent runtime primitives (prompts, history, loop detector, crash learnings).
wiki/             Living project documentation. See wiki/index.md for the page list.
```

## Working rules

These apply to all work in this repo, not just documentation.

### Run `bun run gate` before pushing — and never pipe `bun test` to `tail`.

The pre-push hook runs the full test suite with the 95% per-file coverage threshold. To match it locally, run **`bun run gate`** (= `check && typecheck && bun test` with no piping). This is the single canonical pre-push validation.

**Never** wrap `bun test` in `| tail -N` during dev iteration. The pipe replaces bun's exit code with tail's, so a coverage-threshold failure (exit 1) silently shows up as "all green" — you see "411 pass" and miss that bun was already failing the gate. If you want a shorter view, use `bun test 2>&1; echo "exit=$?"` instead, or just run `bun run gate` and trust the chain.

### Write Conventional Commit messages — they drive releases.

Every commit subject MUST follow [Conventional Commits](https://www.conventionalcommits.org/): `type(optional-scope): summary`, e.g. `feat(cli): add --json flag` or `fix: reject empty patterns`. Use the types already in the log — `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `ci`, `build`, `perf`.

This is not cosmetic. Releases are automated with release-please ([ADR 031](./wiki/decisions/031-release-please-automation.md)), which **parses commit subjects** to compute the next version and generate `CHANGELOG.md`. Consequences you must respect:

- `feat:` and `fix:` are the only types that trigger a release (both bump the patch digit while the project is pre-1.0). `refactor`/`docs`/`chore`/`test`/etc. are hidden by release-please's defaults — they neither bump the version nor appear in the changelog (so a Release PR won't visibly update until a `feat:`/`fix:` lands), though the commits still ride along in whatever release ships.
- A **breaking change** must be marked with `!` after the type (`feat!:`) or a `BREAKING CHANGE:` footer — that's what bumps the minor digit pre-1.0. Never ship a breaking change under a plain `feat:`.
- A subject that doesn't parse as a Conventional Commit is **silently dropped** from the changelog and the version calculation. A malformed release-relevant commit is therefore worse than a verbose one.

Full release flow is in [`CONTRIBUTING.md` → Releasing](./CONTRIBUTING.md#releasing). When a cycle produces a `keep` commit, its message is held to this same standard.

### Keep `src/` and `wiki/` in sync.

The wiki describes the contract; the code implements it. Implementation is allowed to be a superset — internal helpers, performance tweaks, refactor scaffolding, and other private details deliberately don't appear in the spec. The hard rule is that `src/` must never **contradict** the wiki at the contract level (e.g., agent outputs, log entries, CLI surface, tools, runtime guarantees).

- If `src/` changes the contract and the spec is stale → update the spec.
- If the spec changes and `src/` is stale → update the code (open an ADR first if the change is non-obvious).
- If a `src/` change is purely internal (no contract-visible effect) → no wiki edit needed.
- Never write code that contradicts the spec without updating the spec first, or flagging the conflict to the human.

### Check the spec before adding new agents, tools, or commands.

New agent roles, new tools, new CLI commands, and new log entry types are all defined in `wiki/specification.md`. Extend the existing schemas; don't invent parallel ones. If the spec doesn't cover what you need, say so explicitly rather than improvising.

### Match the repo's existing conventions before adding files.

Before creating a new file or choosing where code lives, find where similar code already lives and mirror it — directory layout, file granularity, naming, and error-handling idioms. Do not impose a pattern the repo doesn't already use.

One convention that's easy to miss because it isn't enforced by a failing test:

- **Tests live under `tests/`, never beside the source.** The tree is `tests/{unit,integration,e2e}/` mirroring `src/` (see `wiki/architecture.md` → "Test tiers"). A test for `src/foo/bar.ts` goes in `tests/unit/foo/bar.test.ts` (or the matching tier), **not** `src/foo/bar.test.ts`. The diff classifier counts any `*.test.ts` as a test file regardless of location, so a misplaced test still "passes" — placement is a convention you must follow deliberately, not one the gate catches.

### Group helpers by concern; don't add a file per function.

Shared helpers are organized by concern, not one-function-per-file. Generic, domain-agnostic helpers live in `src/util/<concern>.ts` (`errors.ts`, `strings.ts`, `coerce.ts`). A helper that depends on a domain type lives with the module that owns that type — `assertGitError` sits in `git/run-git.ts` beside `GitError`; the adapter result/stderr builders sit in `agent-runtime/adapters/result.ts`. To add a helper, extend the module whose concern it shares; create a new file only when a genuinely new concern arrives (the same "promote when earned, don't pre-decompose" rule the wiki applies to pages). See [ADR 029](./wiki/decisions/029-grouped-helper-modules.md).

### Fix the root cause. Don't patch around it.

**Patching the symptom is tech debt.** Find the root cause and fix it — the optimal, future-proof solution, even when it's bigger. "Minimum-necessary change" prevents scope creep; it does not license symptomatic patches. When both options exist, name them and let the human pick.

### Intermediate docs are scratch; the wiki is the durable home.

Design specs, implementation plans, brainstorm notes, and other intermediate artifacts (e.g., anything a session-level workflow produces under `docs/`, `docs/superpowers/`, `docs/plans/`, or similar) may be created freely during work on a section — they are useful working memory.

**They must never be committed to the repo.**

At section completion, extract whatever durable content belongs in the wiki (following `wiki/CLAUDE.md` conventions — update core pages, add an ADR for non-obvious decisions), then discard the intermediate file. Not every section produces a wiki change; if an intermediate doc is purely process scaffolding with nothing durable, discard it without a wiki update.

The repo's durable record is `wiki/` + git history. Fossilized parallel documentation under `docs/` is exactly what `wiki/CLAUDE.md` convention #1 (edit in place, no dated copies) exists to prevent — this rule extends the same discipline to intermediate artifacts.

## Editing the wiki

When working on any file under `wiki/`, read `wiki/CLAUDE.md` first. It defines the conventions for keeping the wiki a healthy, living knowledge base (when to create new pages, when to write an ADR, how to cross-reference, etc.). Those rules aren't repeated here — this file is about the project, not about maintaining its documentation.

---
> Source: [JINGBANZ/evolving](https://github.com/JINGBANZ/evolving) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
