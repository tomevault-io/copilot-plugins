## icons

> YunLeFun Icons is the canonical SVG and Iconify collection for YunLeFun brands and applications.

# AGENTS.md

## Project Overview

YunLeFun Icons is the canonical SVG and Iconify collection for YunLeFun brands and applications.

- `packages/icons/svg` is the source of truth.
- `packages/icons/icons.json` is generated in IconifyJSON format.
- `packages/icons/metadata.json` provides searchable names, tags, and source provenance.
- `docs` is a VitePress catalog that verifies UnoCSS consumption.

## Commands

```bash
pnpm build       # SVG -> icons.json -> package build
pnpm test        # Iconify data and UnoCSS integration tests
pnpm lint        # ESLint
pnpm typecheck   # Strict package type checking
pnpm docs:dev    # Build icons, then start the catalog
pnpm docs:build  # Build icons and the production catalog
```

## Conventions

- Use pnpm workspaces and `catalog:` for shared dependency versions.
- ESM only, strict TypeScript, single quotes, no semicolons.
- Edit canonical SVG files and metadata; do not hand-edit generated `icons.json` or `src/generated.ts`.
- Every icon requires source provenance in `metadata.json`.
- Use lowercase kebab-case names. Keep application marks in their original palette unless a monotone source is intentional.
- Run `pnpm build`, `pnpm test`, `pnpm lint`, `pnpm typecheck`, and `pnpm docs:build` before release.
- Never publish, deploy, or push without explicit user authorization.

---
> Source: [YunYouJun/icons](https://github.com/YunYouJun/icons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
