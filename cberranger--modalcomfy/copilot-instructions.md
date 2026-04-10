## modalcomfy

> - Place executables in `scripts/`, model backends in `backends/`, and docs in `docs/`.

# Agent Guidelines

- Use Python 3.10.
- Place executables in `scripts/`, model backends in `backends/`, and docs in `docs/`.
- Run `python -m py_compile` on changed Python files before committing.
- Keep shell scripts portable by resolving paths relative to the script file.
- Ensure new backends expose a `generate` function returning JSON-serializable data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cberranger)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cberranger)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
