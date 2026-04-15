## bitvolt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bitvolt is a browser-based S3 file explorer that provides a Dropbox-like UI for AWS S3 buckets. It operates entirely client-side with no backend — the React app talks directly to the S3 API using user-provided AWS credentials.

## Commands

```bash
pnpm dev              # Start Next.js dev server
pnpm build            # Production build (static export to out/)
pnpm lint             # ESLint (flat config with next/core-web-vitals + typescript)
pnpm test             # Run all tests once (vitest)
pnpm test:watch       # Run tests in watch mode
pnpm test:cov         # Run tests with coverage
npx vitest run src/hooks/useNetworkStatus.test.ts  # Run a single test file
```

Package manager is **pnpm** (v10.29.1). Use `pnpm add` to install dependencies.

## Tech Stack

- **Next.js 16** (App Router) with React 19, configured for static export (`output: 'export'`)
- **MUI 7** + **TailwindCSS 4** for UI (MUI components with Tailwind utility classes mixed in)
- **TanStack React Query** for server state / data fetching
- **AWS SDK v3** (`@aws-sdk/client-s3`, `@aws-sdk/client-iam`, `@aws-sdk/s3-request-presigner`)
- **IndexedDB** (via `idb`) + **Service Worker** for offline support
- **Vitest** + **Testing Library** for tests (jsdom environment)
- **TypeScript** with strict mode and aggressive `no*` checks enabled

## Architecture

### Routing (`src/app/`)

- `/` — Root page: S3 credentials form (access key, secret, bucket, region). Stored in localStorage via `S3ConfigProvider`.
- `/app` — Authenticated shell with permanent sidebar drawer + app bar (`src/app/app/layout.tsx`). Redirects to `/` if no config.
- `/app` page — Main file explorer view
- `/app/recent`, `/app/starred`, `/app/shared`, `/app/trash` — Placeholder pages (ComingSoon)

### Provider Hierarchy (`src/app/layout.tsx`)

```
ThemeProvider → CssBaseline → AppQueryProvider → S3ConfigProvider → children
```

- **S3ConfigProvider** (`src/context/s3config.tsx`): Stores AWS credentials in localStorage. All S3 operations read from this context.
- **AppQueryProvider** (`src/context/queryProvider.tsx`): React Query client configured conservatively for cost — 10min staleTime, 30min gcTime, no auto-refetch on window focus/reconnect.

### S3 Service Layer

- `src/services/s3.ts` — `createS3Service(client, bucket)` returns an `S3Service` interface with: `listObjects`, `createFolder`, `uploadFile`, `deleteObject`, `deleteObjects`, `getDownloadUrl`, `renameObject`. All operations go directly to S3 API.
- `src/hooks/useS3Service.ts` — Creates a memoized S3Service from config context.

### Offline-First System

- `src/services/cache.ts` — IndexedDB cache with two object stores: `listings` (cached S3 directory listings) and `mutations` (pending offline operations).
- `src/services/mutationQueue.ts` — Queues mutations (create/upload/delete/rename) when offline. `flushMutationQueue()` replays them when back online.
- `src/hooks/useNetworkStatus.ts` — Tracks browser online/offline state via `useSyncExternalStore`.
- `public/sw.js` — Service worker: cache-first for static assets, network-first for navigation.

### File Explorer

- `src/hooks/useFileExplorer.ts` — Central hook managing state: current path, objects, selection, search, sort, view mode, upload tasks, and optimistic updates with React Query cache manipulation.
- `src/components/file-explorer/` — UI components: `FileExplorer` (orchestrator), `ExplorerToolbar`, `FileGrid`, `FileList`, `Breadcrumbs`, dialogs (`CreateFolderDialog`, `DeleteConfirmDialog`, `RenameDialog`), `FileIcon`, `UploadProgress`, `EmptyState`.

### Path Alias

`@/*` maps to `./src/*` (configured in `tsconfig.json`). Always use `@/` imports.

## Code Style

- Prettier: 100 char width, single quotes, trailing commas (es5), no arrow parens
- Tests collocated next to source files (e.g., `useNetworkStatus.test.ts` alongside `useNetworkStatus.ts`)
- MUI component imports use deep imports (`@mui/material/Button`, not `@mui/material`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laxsuryavanshi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/laxsuryavanshi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
