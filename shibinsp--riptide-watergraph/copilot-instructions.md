## riptide-watergraph

> A reusable, "like water" multi-agent framework built as a thin layer on LangGraph. Pure Python.

# riptide-watergraph

A reusable, "like water" multi-agent framework built as a thin layer on LangGraph. Pure Python.

## Git commit conventions

- **Granular commits.** Make one logical change per commit — ideally one file when building
  out features — so the history reads as an understandable, step-by-step build. Prefer more,
  smaller commits over a few large ones.
- **Clear Conventional Commit messages.** Use `type(scope): summary`, where `type` is one of
  `feat`, `fix`, `docs`, `test`, `build`, `chore`, `refactor`. The summary must plainly
  describe what the change does so the log is easy to follow at a glance.
- **No AI watermark.** Never add AI attribution or tool footers to commits or PRs
  (no `Co-Authored-By: Claude`, no "Generated with Claude Code"). The author is the user.
- **Branch + push.** Default branch is `main`.

## Project conventions

- Layout: `src/` layout, package `riptide_watergraph`, setuptools build (no compiler needed).
- Every layer sits behind an ABC in `interfaces/` so it stays swappable ("like water").
- Lean core deps; `litellm` and observability are optional extras imported lazily.
- Run the suite with `pytest`; smoke-test offline with `riptide run "<task>" --offline`.

---
> Source: [shibinsp/riptide-watergraph](https://github.com/shibinsp/riptide-watergraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
