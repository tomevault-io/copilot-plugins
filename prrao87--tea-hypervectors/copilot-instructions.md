## tea-hypervectors

> - Run all Python scripts through `uv run`; do not invoke `python` or `python3`

# Instructions

- Run all Python scripts through `uv run`; do not invoke `python` or `python3`
  directly.
- Use `pytest` for all tests; do not use `unittest`.
- Format Python changes with `uv run ruff format <files>` before handing off.
- Always refer to high-dimensional vectors (10K dims and beyond) as **hypervectors** in documentation.
  This is important because the term "vectors" can be confused for traditional text/image embeddings,
  which are of lower dimensionality and are trained from data, which is very different from what HDC uses.

---
> Source: [prrao87/tea-hypervectors](https://github.com/prrao87/tea-hypervectors) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
