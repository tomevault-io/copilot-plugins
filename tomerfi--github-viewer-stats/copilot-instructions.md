## github-viewer-stats

> Development tools, linting, and local testing instructions


# Development

## Linting
```bash
npm run lint
```
ESLint configured via `eslint.config.mjs`.

## Local Testing
Requires `GITHUB_TOKEN` with scopes: `repo`, `read:packages`, `read:user`, `read:discussion`, `read:org` (or `admin:org` for full org stats).

```bash
npm run contribs
npm run repo <repo-name>
npm run org <org-name>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TomerFi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
