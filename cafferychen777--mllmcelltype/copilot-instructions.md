## mllmcelltype

> mLLMCelltype is a multi-LLM consensus framework for scRNA-seq cell type annotation. The associated Communications Biology article was published on June 8, 2026 (DOI: 10.1038/s42003-026-10420-8).

# Project

mLLMCelltype is a multi-LLM consensus framework for scRNA-seq cell type annotation. The associated Communications Biology article was published on June 8, 2026 (DOI: 10.1038/s42003-026-10420-8).

The repository has four maintained domains:

- `R/`: R package.
- `python/`: Python package.
- `web/`: browser application and deployment configuration.
- `paper/`: accepted-paper analysis, manuscript sources, local data, and results.

The top-level R and Python packages are the only maintained software implementations. Do not recreate historical package copies inside `paper/`.

# Verification before pushing

- Run the relevant tests for every changed domain.
- Run `ruff check` for changed Python code.
- Check for secrets and private publication records.
- Do not commit `uv.lock`.
- Do not mention AI coding tools in commit messages or authorship.
- Never use `git add .`; inspect `git status`, add explicit paths, and review `git diff --cached`.

# Research execution

- Run large paper computations on arseven, not locally.
- Use `uv` for Python environments.
- On arseven, load `R/4.4.1-gfbf-2023b` and prepend `~/R/4.4` to `.libPaths()`.

---
> Source: [cafferychen777/mLLMCelltype](https://github.com/cafferychen777/mLLMCelltype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
