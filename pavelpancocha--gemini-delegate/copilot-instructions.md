## gemini-delegate

> Delegate well-specified helper tasks to Gemini CLI (headless). Gemini handles answer/research/review/tests/alt or scoped patch proposals, outputs unified diffs only, and Codex applies edits and runs tests.


## What this skill is for
Use when:
- Task is well-specified and chunked down, and you want a fast second opinion.
- You want bounded answers, research, review, test planning, or alternatives.
- You want a unified diff proposal for explicitly scoped files.
- You want parallel roles and then a consolidated summary.

## Safety boundary
- Gemini MUST NOT directly modify files.
- Gemini outputs diffs; Codex applies and validates.

## Practical workflow
- For patch: build payload and include file(s) with `--files`.
- Apply diff with `git apply --check` then `git apply`.
- If tests fail: feed error output back into another round.

---
> Source: [PavelPancocha/gemini-delegate](https://github.com/PavelPancocha/gemini-delegate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
