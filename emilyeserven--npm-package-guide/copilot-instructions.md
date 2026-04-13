## npm-package-guide

> Educational single-page application (SPA) with multiple guides for backend engineers learning frontend development. Deployed as a static site to GitHub Pages.

# CLAUDE.md

## Project Overview

Educational single-page application (SPA) with multiple guides for backend engineers learning frontend development. Deployed as a static site to GitHub Pages.

## Guides

Multiple independent guides plus top-level resource pages. Guide metadata is co-located in each guide's data file as a `*_GUIDE_MANIFEST` export (auto-discovered by `src/data/guideRegistry.ts` via `import.meta.glob`). Each guide has its own `CLAUDE.md` in its content directory with guide-specific audience, conventions, and component usage.

Guide IDs match content directory names — run `ls src/content/` to discover all guides. Full metadata (titles, start pages, sections, dates) is in each guide's `*_GUIDE_MANIFEST`. Guides are multi-page by default; those with `singlePage: true` in their manifest are single-page. Every guide has `dateCreated` and `dateModified` (ISO `YYYY-MM-DD` strings) — displayed on home page tiles and used for date-based sorting.

Guide categories (set in `*_GUIDE_MANIFEST.def.category`): `frontend`, `infrastructure`, `security`, `ai-tooling`, `fundamentals`. Labels are in `src/data/guideTypes.ts`. The scaffold script defaults to `fundamentals` — set the correct category after scaffolding.

Every guide follows the same file layout:
- **Data:** `src/data/<guideId>Data.ts` (or `src/data/<guideId>Data/` directory)
- **Content pages:** `src/content/<guide-id>/*.mdx` (auto-discovered)
- **Interactive components:** `src/components/mdx/<guide-id>/`
- **Glossary terms:** `src/data/glossaryTerms/<guideId>Terms.ts`
- **Link registry:** `src/data/linkRegistry/<guideId>Links.ts`
- **Guide-specific docs:** `src/content/<guide-id>/CLAUDE.md`

Guide CLAUDE.md files document only guide-specific information: audience context, interactive component APIs (props and data sources), and conventions unique to that guide. Standard patterns (file layout, shared components, dark mode, adding pages) are covered here and in `docs/`.

### Top-Level Resources

- **External Resources** (`src/components/ExternalResourcesPage.tsx`) — searchable, filterable table derived from `src/data/linkRegistry/`.
- **Glossary** (`src/components/GlossaryPage.tsx`) — searchable glossary. Data in `src/data/glossaryTerms/`.
- Both support `?guide=` URL param to pre-select a guide filter.

## Tech Stack

React 19 · TanStack Router (hash-based) · TanStack Table · Zustand · TypeScript (strict) · Vite 7 · pnpm · Tailwind CSS v4 + `clsx` · GitHub Pages at `/npm-package-guide/`

## Commands

| Command | Purpose |
|---------|---------|
| `pnpm dev` | Start dev server |
| `pnpm build` | TypeScript check + Vite build |
| `pnpm lint` | Run ESLint |
| `pnpm validate` | Full pipeline: `validate:data` + `lint` + `build` |
| `pnpm fix-and-validate` | Auto-fix lint errors then run full validation pipeline |
| `pnpm scaffold-guide` | Create boilerplate files for a new guide (see `/add-guide` skill) |

## Project Structure

- `src/components/` — Shared React components; `src/components/mdx/` has MDX-available components (auto-discovered from guide barrel files)
- `src/content/<guide-id>/` — MDX pages, auto-discovered by `src/content/registry.ts`
- `src/data/` — TypeScript data objects; `linkRegistry/` and `glossaryTerms/` split by guide
- `src/data/guideRegistry.ts` — Central guide registry (auto-discovers `*_GUIDE_MANIFEST` and `*_CHECKLIST_MANIFEST` exports, provides lookup helpers)
- `src/helpers/` — Utilities (`cmd.ts`, `fnRef.ts`, `darkStyle.ts`, `themeColors.ts`, `copyAsMarkdown.ts`)
- `src/hooks/` — Zustand stores and hooks (`useTheme.tsx`, `usePMContext.tsx`, `useUIStore.ts`)

## Key Patterns

- **State management:** Zustand stores in `src/hooks/` — no React Context providers. `useTheme()` and `useIsDark()` for theme, `usePM()` for package manager, `useUIStore` for sidebar/command-menu/scroll/pin state. Stores sync to DOM and localStorage via module-level subscriptions.
- **Content as data:** Page content in `src/data/` as TypeScript objects with HTML strings, not inline JSX.
- **Styling:** Tailwind utility classes. CSS only for pseudo-elements, animations, and third-party selectors.
- **Dark mode:** Use Tailwind `dark:` variants for static colors. Use `ds()` from `src/helpers/darkStyle.ts` for dynamic inline styles from data. For common color pairs (text, backgrounds, borders, shadows), prefer `tc(theme.X, isDark)` from `src/helpers/themeColors.ts` instead of raw `ds()` calls. Dark palette: bg `#1e293b`, text `#e2e8f0`, borders `#334155`.
- **Checklists:** Use `<GuideChecklist checklistId="..." />` in MDX. Checklist data is auto-discovered from `*_CHECKLIST_MANIFEST` exports in guide data files. Do not create per-guide checklist wrapper components. See `src/content/checklist/CLAUDE.md` for adding new checklists.
- **YAML explorers:** Use `YamlExplorerBase` (`src/components/mdx/YamlExplorerBase.tsx`) for interactive YAML annotation UIs. Guide-specific wrappers pass data and file name to the base.
- **Shared over guide-specific:** Always prefer shared components (`SectionLayout`, `TimelineFlow`, `ProsCons`, `DefinitionTable`, etc.) over creating guide-specific ones. Only create a component in `src/components/mdx/<guide-id>/` when the UI is genuinely unique to that guide or when a thin data-lookup wrapper is needed. Use the `/find-component` skill to discover the right shared component for a given need. See `docs/COMPONENT_REFERENCE.md` for the full list of shared bases, pattern-matching quick reference, and consolidation guidance.
- **Page ordering:** Each guide's `*_GUIDE_SECTIONS` array is the single source of truth — sidebar, command menu, prev/next, and home tiles derive automatically.
- **MDX titles:** Every `title` must end with an emoji suffix (parsed by sidebar and command menu).
- **Checklists content:** Checklist MDX pages live in `src/content/checklist/` (shared directory, not a guide). See `src/content/checklist/CLAUDE.md` for template and rules. Do not add `guide:` frontmatter to checklist pages.

## Skill Quick Reference

- **New guide from scratch** → `/add-guide` (runs `pnpm scaffold-guide`, then fill in content)
- **Modifying an existing guide** (pages, data, components, glossary, links, checklists) → `/update-guide`
- **Before creating ANY new component** → `/find-component` first (prevents duplication)

## Updating an Existing Guide

Use the `/update-guide` skill for a structured workflow covering all update scenarios — adding data, pages, components, links, glossary terms, checklists, or restructuring sections.

Quick reference for adding pages:

1. Create `src/content/<guide-id>/<page-id>.mdx` with frontmatter: `id`, `title` (emoji suffix), `guide`.
2. Add the page ID to the guide's `*_GUIDE_SECTIONS` in its data file.
3. Add new components to `src/components/mdx/<guide-id>/` and export them from the directory's `index.ts` barrel file (auto-discovered — no need to edit `src/components/mdx/index.ts`). Use `/find-component` before creating new components to check for shared bases.
4. Add link entries to `src/data/linkRegistry/<guideId>Links.ts` as needed.

## Adding a New Guide

Use the `/add-guide` skill, which runs `pnpm scaffold-guide`. The scaffolded data file includes a `*_GUIDE_MANIFEST` export — auto-discovered by `guideRegistry.ts`. No manual registration in `guideRegistry.ts` or `mdx/index.ts` is needed. `dateCreated` and `dateModified` are auto-set to the scaffold date; update `dateModified` when making significant content changes to an existing guide. Naming conventions for `--prefix` (UPPER_CASE constants) and `--camel` (camelCase filenames): derive short forms from the guide ID. Examples: `kubernetes` → `K8S`/`k8s`, `tanstack-query` → `TSQ`/`tsq`, `coolify-deploy` → `COOLIFY`/`coolify`, `ci-cd` → `CICD`/`cicd`. See `.claude/skills/add-guide/REFERENCE.md` for more detail.

For component discovery, use the `/find-component` skill.

## Pre-Push Checklist

1. `pnpm install` (if dependencies changed).
2. `pnpm lint --fix` — always run first to auto-fix lint errors.
3. `pnpm validate` — runs `validate:data` + `lint` + `build`. Fix any remaining errors.
4. `pnpm test:e2e` — run if making sidebar, routing, theme, or checklist changes.

Update `dateModified` in the guide's `*_GUIDE_MANIFEST` if you: add/remove pages, restructure sections, add interactive components, or make substantial content changes. Don't update for typo fixes or minor rewording.

## Reference

Detailed conventions (read on-demand, not needed for most tasks):

- `docs/CONTENT_REFERENCE.md` — MDX frontmatter, link registry, glossary, footnotes, cross-page links, navigation formatting
- `docs/DEVELOPMENT_REFERENCE.md` — `ds()` helper, routing, page ordering, build validation, TypeScript/ESLint config, error troubleshooting
- `docs/COMPONENT_REFERENCE.md` — Shared base components, when to create (or avoid) guide-specific components, consolidation opportunities

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emilyeserven)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/emilyeserven)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
