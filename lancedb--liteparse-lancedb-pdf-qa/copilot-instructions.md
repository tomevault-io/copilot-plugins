## liteparse-lancedb-pdf-qa

> Local ESG report analyst demo for a LiteParse + LanceDB technical blog post, built on a Climate Finance Bench subset.

# AGENTS.md

Local ESG report analyst demo for a LiteParse + LanceDB technical blog post, built on a Climate Finance Bench subset.

## Conventions

- **Dependency management: use `uv`.** Add packages with `uv add <pkg>`; never edit `pyproject.toml` deps by hand or use `pip`. Run scripts with `uv run python scripts/<name>.py`.
- **Code formatting: use `ruff` at line-length 100.** Before finalizing any Python code, run:

```bash
uvx ruff format --line-length 100 .
uvx ruff check --line-length 100 .
```

- Python 3.13+.
- Run project workflows as standalone scripts under `scripts/`, not as an installed CLI.

## Layout

- `scripts/` — runnable workflow steps (data download, benchmark inspection, LiteParse parsing, LanceDB table creation, retrieval evals). These are the orchestration layer; keep reusable logic in `src/`.
- `src/` — flat helper modules holding the reusable pipeline logic: config/paths, benchmark and report loading, the LiteParse parser wrapper, the LanceDB schema, embeddings, indexing, retrieval, and evaluation. No installed package or CLI.
- `data/` — downloaded benchmark subset (raw reports, manifest, eval questions); gitignored.

---
> Source: [lancedb/liteparse-lancedb-pdf-qa](https://github.com/lancedb/liteparse-lancedb-pdf-qa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
