## opencodex-account-importer

> This repository imports local credential exports into OpenCodex. Treat every user-supplied JSON file as secret.

# Agent Instructions

## Scope

This repository imports local credential exports into OpenCodex. Treat every user-supplied JSON file as secret.

## Required Safety

- Never print, upload, commit, summarize, or paste access tokens, refresh tokens, ID tokens, or private keys.
- Use `--dry-run` before a real import unless the user explicitly requests otherwise.
- Do not run imports against the real `~/.opencodex` directory during tests; set `OPENCODEX_HOME` to a temporary directory.
- Fixtures must use placeholders and must pass `npm run check:secrets`.
- Do not infer source formats from weak field similarity. Keep CPA, Sub2API, and Antigravity parsers separate and fail closed.
- Do not publish to GitHub or npm without explicit user authorization.

## Verification

Run before completion:

```bash
npm run check
npm pack --dry-run
```

---
> Source: [feiyulc/opencodex-account-importer](https://github.com/feiyulc/opencodex-account-importer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
