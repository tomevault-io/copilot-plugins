## soliloquy

> Operational guide for coding agents working on Soliloquy.

# AGENTS.md

Operational guide for coding agents working on Soliloquy.

## Project Overview

Soliloquy is a privacy-focused, local-first personal notes app presented as a
messenger. It has no account or backend: chats and messages live in the
browser's IndexedDB database. The current product supports chat and message
management, pinned content, drag-and-drop ordering, chat icon/color
personalization, Markdown rendering, single-chat JSON/Markdown transfer, full
workspace backup/restore, and a multi-page Settings modal.

Stack: Astro 5, React 19, strict TypeScript, Tailwind CSS 4, Radix-based UI
primitives, Dexie 4 with `dexie-react-hooks`, dnd-kit, lucide-react,
react-markdown/remark-gfm, and Biome. Use Bun; `bun.lock` is authoritative.

## Commands

Only these package scripts are available:

```bash
bun install
bun run dev
bun run build
bun run preview
bun run astro
bun run format
bun run lint
bun run check
```

There is no `test` or `typecheck` script. Run focused tests with Bun, for
example `bun test src/lib/utils/sidebar-chats/sidebarChats.test.ts`, and use
`bunx tsc --noEmit` for a manual type check when needed.

## Project Structure

```text
src/pages/                 Astro routes; index.astro mounts the React app
src/styles/                Global CSS variables and Tailwind base styles
src/content/changelog/     Manual Markdown release notes
src/lib/components/
  messenger/               Messenger layout, chat window, sidebar, feature UI
    chat/                  Chat-window feature components and local helpers
    sidebar/               Sidebar list, dialogs, selection, and settings/
      settings/            Settings modal shell, pages, and local rows/types
  shared/                  Reusable app-level presentation components
  ui/                      Radix/shadcn-style primitives
src/lib/hooks/             Thin composition hooks and focused chat/sidebar hooks
src/lib/services/          Only layer that accesses Dexie
  import-export/           JSON, Markdown, and workspace-backup transforms
src/lib/db/                Dexie schema and migrations
src/lib/types/             Domain types for Chat and Message
src/lib/utils/             Pure UI/data helpers
```

Focused tests are colocated with their utility, service transform, or feature
helper as `*.test.ts`; there is no global test directory or DOM test harness.

## Architecture And Style

- `src/pages/index.astro` mounts `MessengerLayout` with `client:only='react'`.
  The layout owns the active chat; `Sidebar` coordinates sidebar dialogs,
  toasts, workspace restore flow, and `SidebarSettings`.
- Keep orchestration components responsible for state and service callbacks.
  Keep page/row components presentational and feature-local when they are not
  reused elsewhere. `SidebarSettings` owns modal/page state; its Data page owns
  file inputs.
- Do not split a component mechanically by line count. Split when it mixes
  independent UI sections, dialog logic, transformations, or business state;
  orchestration components may remain larger when they only coordinate focused
  children and callbacks.
- Keep `useChatWindow.ts` and `useSidebar.ts` as thin composition facades.
  Put focused responsibilities under `hooks/chat/` and `hooks/sidebar/`.
- Components and hooks must call services, never `db` directly. Dexie queries,
  transactions, schema versions, and migrations belong in `src/lib/db/` or
  `src/lib/services/`.
- `importExportService` coordinates browser files and database writes.
  `services/import-export/` owns pure format parsing/serialization. Keep
  single-chat transfer distinct from workspace backup/restore.
- Release notes are manual Markdown files in `src/content/changelog/`. Each
  entry requires `version`, `date`, and `title` frontmatter, is loaded at build
  time, and describes user-facing results rather than raw commits. Keep the
  release entry synchronized with the package version; do not generate notes
  from Git history automatically.
- The database has `chats` and `messages` tables. The seeded `Soliloquy Info`
  chat is a read-only system chat. Its visibility preference is stored in
  `localStorage` and applied only to the sidebar view.
- Use `$lib/*` aliases outside a local feature and relative imports inside a
  feature. Use tabs, single quotes, no semicolons, TypeScript interfaces for
  component props, and `cn` from `$lib/utils` for conditional classes.
- Reuse existing UI primitives and Lucide icons. Preserve keyboard behavior,
  visible focus states, labels, dialog semantics, and accessible names.

```tsx
interface ExampleRowProps {
	title: string
	onClick: () => void
}

export function ExampleRow({ title, onClick }: ExampleRowProps) {
	return <Button variant='ghost' onClick={onClick}>{title}</Button>
}
```

## Testing And Verification

- Add focused tests for pure transforms, validation, and boundary logic when
  practical. Unit and component tests live beside the implementation they
  exercise.
- When a production file has its own test, place both in a feature-local
  directory. Do not leave many production/test pairs mixed in one broad
  directory. Do not create a directory for a simple untested file unless it
  also has closely related local files. Integration and end-to-end tests may
  use dedicated directories when the project adds them.
- Run the narrowest relevant `bun test <files>` command after behavioral
  changes. Include the exact command and result in the handoff.
- Run `bun run lint` for changed TypeScript/TSX and `bunx tsc --noEmit` when
  type contracts change. Run `bun run build` for route, build, or core UI
  composition changes.
- Manually verify affected browser flows, especially IndexedDB migrations,
  imports, destructive restore confirmation, and persisted preferences.
- Report warnings and checks that could not run honestly; do not invent results.

## Git Workflow

- Make focused, reviewable changes and do not mix unrelated cleanup.
- Use Conventional Commits when proposing or creating commits, for example
  `feat(settings): add workspace restore action`.
- In every handoff, list changed files and their purpose, verification results,
  risks or assumptions, and concise manual testing steps.
- Do not commit unless the user explicitly requests it.

## Boundaries

### Always

- Inspect the existing implementation, conventions, and available scripts
  before editing.
- Reuse existing services, components, constants, and feature patterns.
- Preserve local-first behavior and backward compatibility unless the task says
  otherwise.
- Run relevant automated checks and compare completed behavior with `SPEC.md`.

### Ask First

- Changing Dexie schema versions, migrations, or table definitions.
- Adding or removing dependencies.
- Changing JSON/Markdown/workspace-backup formats or restore semantics.
- Changing CI/CD, deployment configuration, public APIs, or broad architecture.
- Deleting functionality or performing a large cross-feature refactor.

### Never

- Edit secrets, environment files, generated output, dependencies, or vendor
  directories.
- Delete or weaken failing tests merely to pass checks.
- Query Dexie directly from components or hooks, or duplicate service business
  logic in UI.
- Add server persistence, analytics, authentication, tracking, or out-of-scope
  product functionality without explicit approval.

---
> Source: [stkossman/soliloquy](https://github.com/stkossman/soliloquy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
