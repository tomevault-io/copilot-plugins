## 00-project-core

> Core MyMemos invariants - surfaces, artifacts, constants, and scope discipline


# MyMemos Core Rules

## Product identity

- **Product name:** MyMemos (`mymemos` in package.json)
- **CSS prefix** `ko-` is legacy but still canonical for extension UI

## Three surfaces - never conflate

| Surface | Path | React | Constants |
|---------|------|-------|-----------|
| Landing | `src/` | 19 | `@/lib/constants` → `shared/constants.ts` |
| Extension | `extension/src/` | 19 | `@/lib/constants` → `shared/constants.ts` |
| Web demo | `extension/src/` (web build) | 19 | same as extension |

`@/` alias points to different roots per package. Check `tsconfig.json` before importing. Product constants live in `shared/constants.ts` (`@shared/*`).

## Generated artifacts - do not edit

- `public/demo/**`
- `public/mymemos-extension.zip`
- `public/robots.txt`, `public/sitemap.xml`, `public/llms.txt`
- `src/routeTree.gen.ts`
- `extension/dist/**` (dev build output)

Regenerate via `npm run build:web`, `npm run package:extension`, `npm run generate:seo`, or `npm run dev:web`.

## Constants & literals

**Policy:** `.cursor/rules/constants-policy.mdc` (always apply). Read it before any copy/UI/tunable change.

- **Source of truth:** `shared/constants.ts`
- Package files `src/lib/constants.ts` and `extension/src/lib/constants.ts` are re-exports only
- Hardcoded user-facing strings, error messages, and magic numbers → named export in `shared/constants.ts` with JSDoc, then import via `@/lib/constants`
- Never add parallel copy modules (`strings.ts`, `messages.ts`, etc.)
- Extension empty-string checks → `len(value) === 0` from `@/lib/text`
- FAQ/AI crawler prose → `src/lib/ai-content.json`; SEO builders → `src/lib/seo.ts`

## Change discipline

- Smallest correct diff; no drive-by refactors
- Match sibling file patterns (imports, error handling, naming)
- **Filenames:** PascalCase components, camelCase modules, kebab-case scripts - see `.cursor/rules/file-naming.mdc`
- Run `npm run ci` before declaring PR-ready
- Do not commit unless user explicitly asks

## Dev server selection

- Extension UI → `npm run dev` (never `build:extension` during active dev)
- Landing + demo → `npm run dev:web`
- Demo only → `npm run dev:app`

## When stuck

1. Read `AGENTS.md` decision trees (§4)
2. Route via `.cursor/SKILLS.md`
3. Grep for an existing pattern before inventing one

---
> Source: [aryancodes-tech/my-memos](https://github.com/aryancodes-tech/my-memos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
