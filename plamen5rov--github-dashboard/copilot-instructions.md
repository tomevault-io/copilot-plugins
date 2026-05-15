## github-dashboard

> React + TypeScript + Vite app that surfaces trending GitHub repositories with rich filtering, sorting, personalization, and discovery. Fully client-side — user provides their own GitHub PAT via Settings.

# AGENTS.md — GitHub Trending Repos Explorer

## Project Overview

React + TypeScript + Vite app that surfaces trending GitHub repositories with rich filtering, sorting, personalization, and discovery. Fully client-side — user provides their own GitHub PAT via Settings.

## Tech Stack

| Layer | Choice |
|-------|--------|
| Framework | React 19 + Vite 7 |
| Styling | Tailwind CSS 3 |
| Data Fetching | TanStack Query 5 (infinite scroll) |
| Routing | React Router 7 |
| Testing | Vitest 4 + React Testing Library + MSW 2 |

## Key Architecture Decisions

- **REST search** for repo listing (`/search/repositories`), **GraphQL** for enrichment (PR counts, contributor data, README text)
- **Filter/sort state in URL search params** — every combination is bookmarkable
- **localStorage** for personalization (bookmarks, collections, followed topics, ignore list, theme, PAT)
- **No backend** — PAT stored in localStorage, never sent anywhere except GitHub
- **Client-side license filtering** for "Open Source Only" / "No License" (GitHub API doesn't support `license:null` or `OR` between qualifiers)
- **`stars:>=N` replaces default `stars:>50`** — GitHub ignores duplicate star qualifiers
- **Ignored languages use separate `-language:X`** per language (not grouped)

## API Quirks

- `license:null` is **not** a valid qualifier (returns 422)
- GitHub REST search **does not support `OR`** between qualifiers
- Repeated `stars:` qualifiers — only the **first** is used
- Time range maxes at **30 days** — beyond that `pushed:>date stars:>50 sort:stars` returns all-time popular repos

## UI/UX

- Dark mode default, light toggle (persisted)
- Responsive: 1-col mobile, 2-col tablet, 3-col desktop
- Skeleton loading, error states with retry, empty states with reset CTA
- All interactive elements keyboard-navigable with ARIA labels

## Agent Workflow Rules

1. **Auto commit/push**: After each feature, suggest commit+push. When user says "ok", do it.
2. **DONE.md changelog**: Log every change at project root.
3. **Check README after every feature add/remove** — scan for mentions in feature lists, structure, test counts.
4. **Run `tsc --noEmit` after multi-file refactors** before committing.
5. **`grep` for cross-file references before deleting** any type, function, or file.

## Lessons Learned (Agent Mistakes to Avoid)

1. **Orphaned `try`/`catch` blocks** — when removing code inside a `try`, always verify the structure is still valid. A lone `try {` without `catch`/`finally` breaks the file.
2. **Always check README after feature additions/removals** — scan for mentions: feature lists, project structure, test counts, descriptions. Don't wait to be told.
3. **Run typecheck after multi-file refactors** — `tsc --noEmit` catches syntax errors that grep-based edits miss.
4. **`grep` for cross-file references before deleting** — search the entire codebase for imports and usages first. Don't rely on memory.
5. **Verify `try`/`catch` nesting after edits** — nested try/catch blocks in `github.ts` are fragile. Check indentation and block boundaries.

---
> Source: [plamen5rov/github-dashboard](https://github.com/plamen5rov/github-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
