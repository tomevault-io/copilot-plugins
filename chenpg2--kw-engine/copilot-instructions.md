## kw-engine

> kw-engine: methodology evolution engine substrate. Distill → abstract → synthesize → search by problem structure.

# CLAUDE.md

kw-engine: methodology evolution engine substrate. Distill → abstract → synthesize → search by problem structure.

## Commands

```
uv run kw reindex --memory-dir path   # rebuild SQLite + index.json from markdown
uv run kw verify --memory-dir path    # run SCHEMA §6 invariants
uv run kw status --memory-dir path    # show counts + pending/stale state
uv run pytest                          # all tests
uv run ruff check .                    # lint
uv run mypy src/                       # type check
```

## Conventions

- Markdown is truth; SQLite + index.json are derived (kw reindex rebuilds both)
- Paper id = PDF filename stem
- Principle id = P-#### (zero-padded)
- All models in models.py; store/ never imports from cli.py
- No silent fallback: validation errors raise, never coerce.

---
> Source: [chenpg2/kw-engine](https://github.com/chenpg2/kw-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
