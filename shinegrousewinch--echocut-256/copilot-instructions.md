## echocut-256

> This repository's primary agent guide lives in **[CLAUDE.md](CLAUDE.md)** — it works for any

# AGENTS.md

This repository's primary agent guide lives in **[CLAUDE.md](CLAUDE.md)** — it works for any
AI coding tool (Claude Code, Cursor, Codex, …). Read it first for architecture, the burn
pipeline, layout rules, and the design constraints to respect before changing code.

## Quick reference

```bash
npm run check       # syntax check — MUST pass after any code change
npm run test:unit   # node:test unit tests (tests/*.test.js)
npm test            # check + unit tests
```

- **Code style:** match the surrounding file — CommonJS (no ESM), existing naming/comment density.
- **Before a PR:** `npm run check` and `npm run test:unit` must pass. See [CONTRIBUTING.md](CONTRIBUTING.md).
- **Don't:** commit secrets/`.env`, add personal/brand data to example configs, or break the
  "brand overrides content, not layout" rule (see CLAUDE.md → Key design constraints).
- **User-facing docs:** [README.md](README.md) · [docs/CLI.md](docs/CLI.md).

---
> Source: [shinegrousewinch/echocut-256](https://github.com/shinegrousewinch/echocut-256) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
