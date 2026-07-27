## context-kit

> - For unittest, please use `unittest` instead of `pytest`.

# Claude Memory

## Dev Guidance

- For unittest, please use `unittest` instead of `pytest`.
- For examples with terminal output, use `examples/util.py` for consistent colored display (wraps `rich`).
- Error handling rule: keep `raise` in core Python functions (e.g., `context_kit/memory.py`), and only convert exceptions to user/tool-facing error messages at the tool/wrapper boundary via `try/except` to avoid breaking the agent loop.

## Examples

For one example, just contain single `main` function. Don't contain multiple example in one file.

---
> Source: [keli-wen/context-kit](https://github.com/keli-wen/context-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
