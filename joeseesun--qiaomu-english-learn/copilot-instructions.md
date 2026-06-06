## qiaomu-english-learn

> This project is a `qiaomu-english-learn` fork/skin of Read Frog, not a rewrite that strips Read Frog down.

# AGENTS.md

## Project Direction

This project is a `qiaomu-english-learn` fork/skin of Read Frog, not a rewrite that strips Read Frog down.

- Preserve existing Read Frog translation, selection, subtitle, TTS, provider, and configuration capabilities unless a change is explicitly needed.
- Add English vocabulary learning as a new translation + learning mode layered on top of existing reading scenarios.
- Treat webpage reading, selection translation, subtitle translation, and TTS as valid vocabulary acquisition contexts.
- The core learning behavior is: default-help users with words at CET4 difficulty and above, let users mark words as already known, then reduce future assistance until English pages become readable without prompts.
- Prefer additive modules, settings, and UI over invasive rewrites of existing translation flows.

## Working Rules

- Be direct. Push back with specific reasons when a proposed change would damage existing Read Frog behavior or over-narrow the product.
- Verify technical facts before relying on them, including package versions, APIs, CLI flags, and service behavior.
- Do not claim a task is done until the relevant typechecks, linters, tests, builds, or live UI checks have passed. If a check cannot be run, say so.
- Before coding, state meaningful assumptions when ambiguity matters. Ask only when a wrong assumption would be risky or expensive.
- Write the minimum code that moves the project toward the product direction. Match existing style and touch only required files.
- Prefer project-provided commands from `package.json`, scripts, or CI config.
- Never bypass hooks with `--no-verify` or equivalent flags unless explicitly approved.
- Never push directly to `main`, `master`, or the default branch.
- Before committing or pushing, inspect staged changes for secrets.

## Testing Notes

- `src/utils/host/translate/api/__tests__/free-api.test.ts` depends on live external translation services.
- When running tests locally as an AI agent, set `SKIP_FREE_API=true`.
- If `SKIP_FREE_API=true` is set, treat `free-api.test.ts` as intentionally skipped during local validation.

---
> Source: [joeseesun/qiaomu-english-learn](https://github.com/joeseesun/qiaomu-english-learn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
