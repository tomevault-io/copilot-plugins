## docflow

> Before considering work complete, run:

# AGENTS.md

## Validation (Monorepo)

Before considering work complete, run:

```bash
pnpm type-check
```

Note: Project runs continuously; do not use `pnpm dev`.

## Conventions

- Next.js App Router for routing only; no API routes.
- Global components: `components/` ; Page-level: `app/[route]/_components/`
- Utilities: `utils/`
- Avoid `useMemo`/`useCallback` unless necessary.

---
> Source: [xun082/DocFlow](https://github.com/xun082/DocFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
