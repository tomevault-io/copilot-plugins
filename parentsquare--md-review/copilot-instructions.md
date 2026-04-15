## md-review

> A collaborative web app that brings Google Docs-style inline commenting to Markdown files on GitHub. Comments are stored as GitHub PR review comments, not in a database. There is no persistent backend state.

# Markdown Reviewer — Cursor Project Context

## What This Project Is

A collaborative web app that brings Google Docs-style inline commenting to Markdown files on GitHub. Comments are stored as GitHub PR review comments, not in a database. There is no persistent backend state.

## Tech Stack

- **Frontend**: React 19 + TypeScript + Vite
- **Styling**: Tailwind CSS v4 (via `@tailwindcss/vite` plugin, NOT PostCSS)
- **Backend**: Express.js (only for OAuth token exchange at `/api/auth/github`)
- **Auth**: GitHub App OAuth flow (token stored in sessionStorage)
- **Deployment**: Docker → Google Cloud Run (port 8080)
- **Tests**: Vitest + React Testing Library + jsdom

## Architecture Overview

- `src/App.tsx` — Single orchestrator component (768 lines). Manages auth state, file selection, comment CRUD, and approval workflow. All state is React `useState`.
- `src/lib/github.ts` — `GithubApi` class. All GitHub REST + GraphQL interactions. Singleton accessed via `getGithubApi()`.
- `src/lib/markdown.ts` — Comment data model. `encodeCommentBody()` / `decodeCommentBody()` serialize comment metadata into PR review comment bodies using HTML comment blocks.
- `src/lib/lineMap.ts` — Maps user text selection back to source line numbers for GitHub review comments.
- `server/index.js` — Minimal Express server. Only endpoint: `POST /api/auth/github` (exchanges OAuth code for token).

## Key Patterns

1. **No database** — Everything is stored on GitHub (file content, comments, approvals).
2. **Draft PR pattern** — First comment on a branch auto-creates a Draft PR. All comments attach to that PR.
3. **Metadata in comment bodies** — Thread replies, resolve status, and quoted text are encoded as JSON inside `<!-- md-review ... -->` HTML comments within the PR review comment body.
4. **File ID format** — `owner/repo/branch/path/to/file.md`. Parsed by `GithubApi.parseFileId()`.
5. **Client-side API calls** — The frontend calls GitHub API directly using the user's OAuth token. The backend only proxies the initial token exchange.
6. **sessionStorage auth** — Access token and user profile cached in sessionStorage. No persistent sessions.

## Coding Conventions

- All frontend code is TypeScript (`.tsx` / `.ts`). No `.jsx` or `.js` in `src/`.
- React 19 patterns — functional components, hooks, no class components.
- No state management library — all state lives in `App.tsx` via `useState` / `useCallback`.
- Tailwind CSS v4 — utility classes directly in JSX. Uses `cn()` helper from `src/lib/utils.ts` (wraps `clsx` + `tailwind-merge`).
- Components are PascalCase, utilities are camelCase.
- Test files are colocated: `Component.tsx` alongside `Component.test.tsx`.

## Important Files to Understand First

1. `src/App.tsx` — Start here. It's the entire app flow.
2. `src/lib/github.ts` — The API client. Understand `findOrCreateDraftPR()`, `createReviewComment()`, and `parseReviewComments()`.
3. `src/lib/markdown.ts` — The encode/decode functions are critical to understanding how comments persist.
4. `ARCHITECTURE.md` — Full structural documentation.

## Running Locally

```bash
npm run server   # Terminal 1: Express OAuth proxy (port 8089)
npm run dev      # Terminal 2: Vite dev server (port 8080, proxies /api → 8089)
```

## Common Tasks

- **Add a new component**: Create in `src/components/`, import in `App.tsx`
- **Add a GitHub API call**: Add method to `GithubApi` class in `src/lib/github.ts`
- **Modify comment structure**: Update types in `src/lib/markdown.ts`, update encode/decode functions
- **Run tests**: `npm test`
- **Type check**: `npm run lint`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ParentSquare) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
