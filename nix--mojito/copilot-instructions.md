## mojito

> Read the first section of README.md for project context.

Read the first section of README.md for project context.

## Testing

Two independent test suites:

- **JS unit tests**: `bun test tests/` — fast, for pure JS modules
- **Playwright browser tests**: `.venv/bin/pytest tests/ -xvs` — starts datasette, tests the app in a real browser

After JS-only changes (urlstate, ddl, mjsql, mjtemplate), bun tests
are sufficient. After changes to HTML, templates, rendering, or
interaction code, run pytest. If the task involves a UI bug, run
pytest even if the fix was purely in JS. When in doubt, run both.

## References

The `references/` directory has complete clones of several 
useful packages including `sqlite` and `sqlite-ast` repos for fast
search.

---
> Source: [nix/mojito](https://github.com/nix/mojito) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
