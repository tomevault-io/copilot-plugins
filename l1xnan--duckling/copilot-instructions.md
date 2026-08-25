## duckling

> Rspress v2 docs site for Duckling, a pnpm workspace package.

# AGENTS.md — website (Duckling docs)

Rspress v2 docs site for Duckling, a pnpm workspace package.

## Conventions

- Use `pnpm` for all commands (workspace is pnpm-only; never npm/yarn).
- Content is mirrored in `docs/en/` (default locale) and `docs/zh/`.
  Any new page, nav entry (`_nav.json`), or sidebar entry (`_meta.json`)
  must be added in **both** locales.
- Every page needs `title` and `description` frontmatter in its own language.
- Static assets with stable URLs (screenshots, logos) go in `docs/public/`.
- Prefer MDX but keep the narrative readable as plain Markdown.

## Commands

```bash
pnpm docs:dev        # dev server
pnpm docs:build      # static build -> doc_build/
pnpm docs:preview    # preview the built site
pnpm run format      # Prettier formatting
```

Production success criterion: `pnpm docs:build` must pass.

## Tools

### Prettier

- Run `pnpm run format` to format your code.

---
> Source: [l1xnan/duckling](https://github.com/l1xnan/duckling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
