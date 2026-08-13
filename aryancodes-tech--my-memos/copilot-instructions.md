## file-naming

> File naming conventions for new and renamed source files


# File naming

Use one convention per file kind. Do not mix kebab-case and camelCase for the same kind of module.

| Kind | Convention | Examples |
|------|------------|----------|
| React component modules | **PascalCase** | `Sidebar.tsx`, `LandingHero.tsx`, `AttachmentImageNodeView.tsx` |
| Hooks | **camelCase**, `use` prefix | `useStore.ts`, `useAttachmentImage.ts` |
| Non-component TS/TSX modules | **camelCase** | `attachmentManager.ts`, `workspaceTree.ts`, `errorCapture.ts`, `themeTypes.ts` |
| Tests | Match source basename + `.test.ts` | `workspaceTree.test.ts` under `tests/<surface>/…` |
| CLI / Node scripts | **kebab-case** | `generate-sitemap.mjs`, `guard-production-build.mjs` |
| CSS / static assets | Existing patterns | `index.css`; prefer kebab for new asset files |
| CSS class names / HTML ids | **kebab-case** (unchanged) | `ko-workspace-tree`, `landing-faq` - not the same as TS filenames |

## Exceptions (do not rename to force the rule)

- Framework / generated: `__root.tsx`, `routeTree.gen.ts`, TanStack route files like `llms[.]txt.ts`
- Ambient typings that mirror package ids: `markdown-it-mark.d.ts`, `file-system-access.d.ts`
- Entry stubs: `main.tsx`, `index.tsx`, `App.tsx` (PascalCase `App` is intentional)

## When adding a file

1. Component that renders UI → PascalCase `.tsx`
2. Hook → `useSomething.ts`
3. Anything else in `src/`, `extension/src/`, `shared/` → camelCase
4. Anything in `scripts/` or `extension/scripts/` → kebab-case `.mjs`

## Imports

Update import paths when renaming. Prefer `@/` / `@shared/` aliases; do not leave stale kebab paths.

---
> Source: [aryancodes-tech/my-memos](https://github.com/aryancodes-tech/my-memos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
