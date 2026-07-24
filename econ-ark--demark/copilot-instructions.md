## demark

> NEVER use exclamation marks (!) in Python code that will be executed in terminal commands.

# Cursor AI Rules

## Critical Terminal Command Rule
NEVER use exclamation marks (!) in Python code that will be executed in terminal commands.

Examples of what NOT to do:
- `python -c "print('Success!')"`  ❌
- `python -c "print('Error!')"`   ❌

Examples of what TO do:
- `python -c "print('Success')"`   ✅
- `python -c "print('Error')"`     ✅

Reason: Shell interprets ! as history expansion, causing commands to hang or fail.

This rule applies to:
- python -c commands
- Any Python strings in terminal execution
- run_terminal_cmd tool usage

This rule does NOT apply to:
- Python files (.py)
- Jupyter notebooks
- Regular conversation text 

---
> Source: [econ-ark/DemARK](https://github.com/econ-ark/DemARK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
