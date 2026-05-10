## frontend-patterns

> Frontend conventions for the dbxmetagen React dashboard


# Frontend Patterns

## Stack

React 19 + Vite + Tailwind CSS. JSX (not TSX). Entry point: `main.jsx`.

## Build Requirement

`dist/` is committed to the repo and synced to Databricks via DAB. After any frontend change:

```bash
cd apps/dbxmetagen-app/app/src
npm run build
```

Forgetting this means the deployed app won't reflect your changes.

## API Layer

All backend calls go through `apiCache.js`, which handles fetch + caching. Add new API calls there rather than using raw `fetch` in components.

## Structure

- `components/` -- reusable UI components
- `hooks/` -- custom React hooks
- `utils/` -- helper functions
- `public/` -- static assets

---
> Source: [databricks-industry-solutions/dbxmetagen](https://github.com/databricks-industry-solutions/dbxmetagen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
