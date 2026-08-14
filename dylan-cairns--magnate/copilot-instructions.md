## magnate

> Start every session here.

# Agent Manifest

Start every session here.

## Session Checklist

1. Read this file.
2. Read all core Memory Bank files in `memoryBank/`.
3. Create and maintain a plan for any non-trivial task.
4. Update Memory Bank files when project context or decisions change.
5. End with a concise summary and concrete next steps.

Detailed workflow: `docs/AGENT_GUIDE.md`

## Locked Decisions (2026-02-19)

- TypeScript engine is the canonical rules implementation.
- Python training calls TS through a stable Node bridge.
- Shared boundary is a small interface contract, not a full cross-language rules schema.
- Native Python rules are out of scope unless throughput becomes a real bottleneck.

## Working Rules

- Keep engine behavior deterministic (seeded RNG only).
- Keep rule semantics in TypeScript.
- Keep bridge contract stable (`memoryBank/bridgeInterfaceContract.md`).
- Use the project `.venv` for any Python command in this repo.
- When changing Python code, run targeted pytest tests for touched behavior plus Ruff and Pyright before handoff.
- Keep docs aligned with code changes.

## Project Context

- Browser app is playable with selectable bot profiles.
- Training loop is TD-focused and runs as `collect -> train -> promotion eval`.
- Loop orchestration is staged: bootstrap/recalibration with `python -m scripts.run_td_loop`, then ongoing self-play with `python -m scripts.run_td_loop_selfplay`.
- Current Python policy surface: `random`, `heuristic`, `search`, `td-value`, `td-search`.

## Training Runtime Expectations

- Python training/eval is fail-fast:
  - invalid bridge payloads, missing checkpoints, or malformed policy probabilities are hard errors.
- Training scripts require explicit policy selection and the active project `.venv`.
- Prefer promoted checkpoints as warm start for subsequent loops.

## Reference Map

- Overview: `README.md`
- Memory Bank: `memoryBank/`
- Rules reference: `memoryBank/magnateRules.md`
- Bridge contract: `memoryBank/bridgeInterfaceContract.md`

---
> Source: [Dylan-Cairns/magnate](https://github.com/Dylan-Cairns/magnate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
