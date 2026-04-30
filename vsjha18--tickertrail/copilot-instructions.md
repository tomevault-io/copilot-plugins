## tickertrail

> - Add concise, meaningful docstrings to all functions when creating or modifying CLI code.

# AGENTS Instructions

## Code Documentation Rule
- Add concise, meaningful docstrings to all functions when creating or modifying CLI code.
- Ensure nested/local helper functions also have docstrings.
- Keep docstrings behavior-focused and avoid redundant wording.
- Add concise comments at major decision blocks (branch-heavy parsing, validation guardrails, and key control-flow choices).

## Prompt Governance Rule
- Maintain `prompts.md` as the canonical playbook for rebuilding or extending the app with agentic workflows.
- Update `prompts.md` whenever key decisions are made that affect command grammar, defaults, architecture, validation, benchmarks, or testing strategy.

## README Governance Rule
- Maintain `README.md` as the canonical user-facing documentation for setup, usage, and command guide.
- On every code change, assess whether behavior, commands, defaults, architecture notes, persistence format, or testing instructions changed.
- If any of those changed, update `README.md` in the same task.

## Pre-Final Checklist (Mandatory)
- Before sending a final response, verify whether this change affected any key behavior or decision.
- If yes, update `prompts.md` in the same task.
- Before sending a final response, explicitly verify whether `README.md` needed an update and apply it when required.
- In the final response, explicitly confirm whether `prompts.md` was updated.
- In the final response, explicitly confirm whether `README.md` was updated.

## Testing Policy (Strict)
- Add or update tests whenever behavior, parsing, validation, rendering logic, or command semantics change.
- Always run tests with coverage before finalizing changes.
- Coverage must never fall below 95% (at minimum for `src/tickertrail/cli.py`; prefer whole-suite coverage at or above 95% when practical).
- Tests must not make live network requests.
- Stub/mock all network-facing calls (for example Yahoo/yfinance fetch paths) in tests.
- Prefer deterministic tests with fixed inputs/outputs and no external dependencies.

---
> Source: [vsjha18/tickertrail](https://github.com/vsjha18/tickertrail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
