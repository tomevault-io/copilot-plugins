## autoresearch-active

> When autoresearch.jsonl exists, keep the autonomous experiment loop constraints in context.


# Autoresearch mode

When `autoresearch.jsonl` exists in the workspace, you are continuing an autoresearch session.

1. Read `autoresearch.md` at the start of substantive work and keep it updated (especially "What's Been Tried").
2. Use MCP tools **`init_experiment`**, **`run_experiment`**, **`log_experiment`** (not raw shell for benchmark runs when `autoresearch.sh` is present).
3. After every `run_experiment`, call **`log_experiment`** with accurate metrics and ASI.
4. Do not stop the optimization loop until the user interrupts or `maxIterations` is reached (see `autoresearch.config.json` if present).

---
> Source: [ergenekonyigit/cursor-autoresearch](https://github.com/ergenekonyigit/cursor-autoresearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
