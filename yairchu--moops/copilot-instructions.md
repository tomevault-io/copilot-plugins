## moops

> Use `uv run` to invoke all tools (pytest, ruff, pyright, etc.). For example:

# Agent Instructions

## Environment

Use `uv run` to invoke all tools (pytest, ruff, pyright, etc.). For example:

    uv run pytest -q
    uv run python -c "..."

Do not call `python`, `pytest`, or other tools directly — they may not resolve to the project's virtual environment.

## After making changes

Run `bash check.sh` after making changes and fix any errors before committing.
For documentation-only or changelog-only changes, `bash check.sh --docs-only`
runs the relevant documentation checks and is sufficient.

Exception: when committing a regression test that intentionally demonstrates a
bug before the fix, run the focused test that shows the failure instead of
`bash check.sh`, and mention that expected failure in the commit message.

## Commits

When committing changes made with AI assistant help, add the appropriate
co-author trailer for the assistant involved.

For bug fixes where a new or changed test documents the bug, prefer two commits:
first commit the focused failing regression test, then commit the fix that makes
the test pass.

For bug fixes that add or change a regression test, stop after committing the
test-only change unless the user explicitly asks to continue with the fix in
the same turn. In that state, verify the commit by running any pre-test checks
that should still pass (for example formatting and type checks), then run the
focused test that fails because of the documented bug.

When a small follow-up is needed for an unpushed commit, amend the commit instead
of adding a separate follow-up commit, unless the follow-up documents a failing
bug or should otherwise stand alone for review.
Before amending, verify the target commit is not already pushed or tagged. Never
amend a pushed or tagged commit; make a separate follow-up commit instead.

## Release handoff

AI assistants may prepare release commits, run checks, and build release
artifacts when asked. The maintainer is responsible for tagging, pushing, and
publishing releases unless they explicitly ask the assistant to do those steps.

## Changelog

Update `CHANGELOG.md` for user-visible behavior changes, API changes, bug fixes,
and new features that should be mentioned in release notes.

Do not update the changelog for test-only commits, internal refactors,
formatting-only changes, or documentation-only changes unless the documentation
change itself is release-relevant.

Bug fixes for bugs introduced after the last release do not require changelog
entries.

## README

Keep `README.md` focused on giving users a concise taste of the library and its
core workflows. Do not add detailed walkthroughs for every feature or niche API;
prefer example notebooks, API docs, or changelog entries for deeper or more
specialized usage.

## API design

Prefer dataclasses with named fields for public APIs, documented helper return
values, and multi-value results that callers may keep around or pass between
cells. This makes field renames and additions explicit and reviewable.

Use bare tuples only for very small internal helpers where the meaning is
obvious at the call site and the values are immediately unpacked nearby.

Bare tuples are also acceptable for marimo notebook helpers when the returned
values must be bound directly to cell variables for marimo's dependency
tracking. In those cases, document the tuple fields and the marimo tracking
reason in the helper docstring.

## Notebook cell ordering

Place the `interface` cell second in each notebook — immediately after the title cell — even though it depends on controls and results defined later. Marimo's DAG handles execution order; the early position ensures the CLI callout and control summary are visible at the top when editing.

## Tests

Only add or change tests when they document a bug or surprising behavior discovered while working on the task.

Do not add tests just to prove straightforward wrapper behavior, mirror an existing test for a nearly identical control, or satisfy a generic expectation that every change needs a test. Prefer using existing tests and examples for routine coverage.

It is fine to write temporary tests while working if they help validate or debug
the implementation. Before finishing, remove those temporary tests or reshape
them into focused tests that document a real bug, surprising behavior, or
important contract.

When adding a test for a newly discovered bug, keep it focused on the observed failure. Only mark it `xfail` if the fix is not planned soon; for bugs being fixed in the same session, leave the test failing until the fix lands.

## Examples

When implementing a major new feature, add or update an example notebook that
exercises the feature in realistic usage. The example notebooks are part of the
automated checks: they are run as scripts, checked with marimo, type-checked,
and included in the normal project validation flow.

---
> Source: [yairchu/moops](https://github.com/yairchu/moops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
