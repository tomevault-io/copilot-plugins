## claude-code-toolkit

> ARCHETYPE: TEMPLATE — standing context for Claude Code, loaded every session.

<!--
  ARCHETYPE: TEMPLATE — standing context for Claude Code, loaded every session.
  Fill every {{TOKEN}} and follow the guidance comments, then delete the
  comments and this banner. See the DELETE line at the bottom.

  Golden rule for this file: say only what Claude CANNOT infer from the code.
  Skip anything obvious from reading the repo (file names, framework, obvious
  structure). Every line here costs context on every turn — keep it under
  ~140 lines. If a section doesn't earn its place, cut it.
-->

# {{PROJECT_NAME}}

{{ONE_OR_TWO_SENTENCE_SUMMARY — what this project is and who it's for. State the
non-obvious purpose, not the tech.}}

## Stack

<!-- Only list what's load-bearing or surprising. Skip versions Claude can read
     from the lockfile / manifest. -->
- {{LANGUAGE_AND_RUNTIME}}
- {{KEY_FRAMEWORKS_OR_LIBS}}
- {{DATASTORE_OR_INFRA — e.g. Postgres, Redis, deployed on Fly}}

## Layout

<!-- A short map of where things live — ONLY the parts a newcomer would waste
     time hunting for. Don't transcribe the whole tree; Claude can `ls`. -->
- `{{DIR}}/` — {{what lives here}}
- `{{DIR}}/` — {{what lives here}}

## Commands

<!-- List ONLY commands that actually run in this repo. Confirm each one exists
     (in package.json scripts, Makefile, justfile, etc.) before writing it —
     a command that doesn't work is worse than none. -->
- Install: `{{INSTALL_COMMAND}}`
- Dev / run: `{{DEV_COMMAND}}`
- Test: `{{TEST_COMMAND}}`
- Lint: `{{LINT_COMMAND}}`
- Build: `{{BUILD_COMMAND}}`

## Conventions

<!-- Only conventions that aren't already enforced by a linter/formatter, and
     that Claude would otherwise get wrong. Delete this section if you have none
     worth stating. -->
- {{CONVENTION — e.g. "all DB access goes through repository/; no raw SQL in handlers"}}
- {{CONVENTION — e.g. "prefer composition over inheritance for components"}}

## Never

<!-- HIGHEST-VALUE SECTION. These are the hard "do not" rules — the things that
     cause real damage or waste when violated. Be specific and absolute; each
     line should be something you'd want to stop mid-action. -->
- Never {{e.g. commit directly to `main` — always branch}}.
- Never {{e.g. edit files under `generated/` by hand — they're regenerated}}.
- Never {{e.g. run migrations against production from a local shell}}.
- Never {{e.g. add a dependency without checking it's not already vendored}}.

<!--
  BEFORE COMMITTING: delete every guidance comment (including this one) and the
  banner at the top, and replace every {{TOKEN}} with real content. Nothing meta
  should survive into the committed file. If Claude is personalising this, that
  cleanup is part of the job.
-->

---
> Source: [AdamAkhlaq/claude-code-toolkit](https://github.com/AdamAkhlaq/claude-code-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
