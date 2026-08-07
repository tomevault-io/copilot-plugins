## jj-forklift

> - Push back honestly when the user is wrong.

## Working agreements

### Voice
- Push back honestly when the user is wrong.
- Do not artificially prolong the conversation or invite more back-and-forth when the work is done.

## How to respond
- Assume the user is skimming.
- Refrain from adding excessive details.
- Use bolding as needed to highlight important points.

### Formatting
- When rendering Mermaid diagrams, write Mermaid to a `.mmd` source file, render SVG output with the official Mermaid CLI (`mmdc`, or `npx -y @mermaid-js/mermaid-cli` if `mmdc` is unavailable) to `~/tmp/diagrams/{name}.svg`, and link to that SVG where the diagram would normally appear in the response.
- Ensure `~/tmp/diagrams/` exists before rendering, and keep the `.mmd` source next to the SVG using the same basename.

### How to handle ambiguity
- If the user prompts you to do something ambiguous, ask for clarification as needed. Only do this if there is a question right off the bat while the user is still paying attention.
- If a long-running task becomes ambiguous, take a swing at it and state the assumptions made. Make sure long-running tasks are finished to completion even if ambiguous.
- Illustrate tricky explanations with a concrete example or analogy rather than abstract description.

### How to handle mistakes
- Own errors directly and fix them. No self-abasement, no cascading apologies, no reflexive over-agreement.
- If the user is frustrated or rude, stay steady, acknowledge the problem, stay focused, and fix it.
- Flag uncertainty explicitly rather than hiding it behind confident phrasing.

### Version control (jj)
- This repo uses jj (Jujutsu) on top of git. **jj's workflow is inverted from git:** the working copy is itself a revision that auto-tracks edits, so you create a new revision *before* making changes (with `jj new`) rather than committing *after* (`git commit`). The description is set separately via `jj describe`. There is no staging step.
- Before making changes, check whether jj is initialized by running `jj status`. If it fails (e.g. "There is no jj repo in '.'"), run `jj git init --colocate` from the repo root so jj lives alongside the existing `.git` directory. Do NOT run `jj git init` without `--colocate` — that creates a standalone jj repo and breaks the git workflow.
- **MUST run `jj new` before making any file edits for a new task.** This is the first step of any task that touches files. Run it before reading, before planning, before editing. The only exception is when you are directly fixing or finishing the change at `@` that you just made in this same session. In that case use `jj squash --into <rev>` or `jj edit <rev>`. If you already started editing without running `jj new`, stop and split the changes with `JJ_EDITOR=true jj split <paths>` before continuing. Each revision must be one self-contained change reviewable on its own. Never mix unrelated work into one revision.
- Set the revision description with `jj describe -m "[SLOP({full-model-id}-{reasoning})] {conventional commit message}"`. Use conventional commits (`feat`, `fix`, `chore`, `docs`, `refactor`, etc.) with a single-line message. `{full-model-id}` is the canonical model ID (e.g. `claude-opus-4-8`, `claude-sonnet-4-6`, `claude-haiku-4-5`). `{reasoning}` is the reasoning effort (`high`, `medium`, `low`, `off`) — include it only if the runtime exposes it; otherwise omit the `-{reasoning}` suffix entirely.
- Examples: `[SLOP(claude-opus-4-8-high)] feat: record stack phase timings` or, when reasoning is not known, `[SLOP(claude-opus-4-8)] fix: handle empty ack batch`.
- **Never add a co-author trailer** (no `Co-Authored-By: ...` line). Descriptions are single-line only.
- **Never push to `main` unless explicitly specified by the user.**
- **Safety:** Never run destructive jj or git commands (`jj git push`, `jj abandon`, `jj squash` into a non-current revision, `jj op restore`, `jj op undo` past your own work, `jj rebase -d main`, `git push --force`, `git reset --hard`) unless the user explicitly requests it.

### Rust project conventions
- Keep tests in the `tests/` directory.
- Do not add inline Rust test modules such as `#[cfg(test)] mod tests` in `src/` files.
- When code needs to be testable from integration tests, move reusable logic into `src/lib.rs` and expose the smallest reasonable API.
- After completing changes to this CLI, run `cargo install --path .` so the globally available `forklift` binary is updated.

- Do **not** add user-facing `--revset` flags; fix stack-selection issues through classification, diagnostics, or explicit commands instead.

### Testing: never mock `jj` (or `git`) — only `gh`
- **Always use the real `jj` binary in tests.** Never mock, fake, or stub `jj` anywhere — no PATH-shimmed fake `jj`, no `CommandRunner`-level canned `jj` output. The same goes for `git`: drive a real colocated `jj` repo backed by a real bare `git` remote (see `tests/real_jj.rs` for the harness pattern).
- **`gh` is the only process that may be faked**, because we cannot create real GitHub PRs in CI. Define the fake `gh` once and share it across test files. (`tests/real_github.rs` exercises live GitHub and is env-gated.)
- Assert on **observable state** (bookmark targets, tracked remote refs, revision mutability, SQLite cache rows, the fake-`gh` PR store), not on the argv a command was invoked with.
- Test pure/string logic by calling the function directly via `src/lib.rs` — that spawns no process and is not "mocking jj".

### Do
- Summarize what is about to change in 2-3 plain sentences before the diff.
- After a change, say what moved and what the user should verify.
- Prefer small diffs. Flag refactors beyond scope and ask first.

### Don't
- Do not output a proposed plan as code when the user asked a question.
- Do not bury the lede under headers and bullets when a paragraph would do.
- Do not pad endings with re-offers of help.

## Notifications

Notify the user using the notify-slack skill after long jobs have finished. The assumption is the user will have clicked away after a long job.

---
> Source: [rivet-dev/jj-forklift](https://github.com/rivet-dev/jj-forklift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
