## codex-proof

> * Keep `codex-proof` dependency-free at runtime.

# Repository instructions

## Scope

* Keep `codex-proof` dependency-free at runtime.
* Support Windows, macOS, and Linux with Python 3.11 or newer.
* Preserve the evidence schema unless a versioned migration is included.
* Never record environment variables or unredacted credentials in evidence output.

## Workflow

* Inspect existing changes before editing.
* Add or update tests for behavior changes.
* Run the Skill validator after changing `.agents/skills/codex-proof`.
* Run `python -m unittest discover -s tests -v` before completion.
* Exercise the installed console entry point for CLI changes.

## Completion

* Keep structural, test, runtime, behavior, visual, and artifact claims separate.
* Report exact commands and observable results.
* Leave failed or pending claims visible.

---
> Source: [serein-nyx/codex-proof](https://github.com/serein-nyx/codex-proof) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
