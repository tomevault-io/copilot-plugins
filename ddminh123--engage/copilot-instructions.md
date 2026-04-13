## engage

> IIA-standard internal audit platform. On-premises deployment.

# Engage — Internal Audit Management System

IIA-standard internal audit platform. On-premises deployment.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 16 (App Router) |
| UI | React, TypeScript, Shadcn/ui, TailwindCSS |
| ORM | Prisma v7 (`prisma-client` generator, `@prisma/adapter-mariadb`) |
| Database | MySQL |
| Auth | NextAuth.js (Credentials + Entra ID ready) |
| State | TanStack Query + Zustand (minimal) |
| Validation | Zod |
| Forms | React Hook Form + Zod + Shadcn Form |

## Architecture

```
UI (React) → API Routes → Actions (business logic) → Prisma → MySQL
                              ↓
                         Services (external only: file storage, Sharepoint, Entra ID)
```

- UI calls API routes only. Never import Actions in client components.
- All routes protected by auth + access middleware.
- Services are for external integrations only, not DB operations.

## Folder Structure

```
src/
├── app/(main)/[module]/     # Pages: universe, plan, engagement, finding, settings
├── app/api/                 # API routes
├── components/ui/           # Shadcn (DO NOT MODIFY)
├── components/shared/       # Reusable: DataTable, FormDialog, DetailSheet, ConfirmDialog
├── features/[module]/       # components/, hooks/, api.ts, types.ts
├── server/actions/          # Business logic (Prisma)
├── server/services/         # External integrations only
├── server/middleware/        # Auth & access control
├── constants/labels/        # UI text per module (Vietnamese)
├── lib/                     # Utilities (prisma client, etc.)
└── types/                   # Shared types
```

## Key Rules

### Prisma v7
- Import from `@/generated/prisma/client` (NOT `@prisma/client`)
- Always pass driver adapter to `PrismaClient` constructor
- `datasource.url` in `prisma.config.ts`, not schema

### Naming
- `.tsx` files exporting React components: **PascalCase** (`OrgUnitList.tsx`)
- All other `.ts` files: **camelCase** (`api.ts`, `types.ts`)
- Shared components use `Engage` prefix: `EngageDataTable`, `EngagePageHeader`
- DB tables/columns: `snake_case`

### Components
- **Shadcn only** — never create custom basic components
- **Check `src/components/shared/` first** before creating anything new
- Self-sustained: components fetch own data via hooks
- Extract business logic into hooks; components only render

### Constants — No Hardcoding
- API routes: use `API_ROUTES` from `@/constants`
- Page paths: use `PAGE_ROUTES`
- UI text: use label constants (`COMMON_LABELS`, `[MODULE]_LABELS`) — all Vietnamese

### Select Component
Always pass explicit `label` prop to `<SelectItem>`. Without it, Base UI shows raw value ID when children contain JSX.

### Base UI `render` Prop
Shadcn v2 uses `@base-ui/react`. No `asChild` — use `render` prop instead:
```tsx
// ✅ Correct
<DropdownMenuTrigger render={<Button variant="ghost" />}>Open</DropdownMenuTrigger>
// ❌ Wrong
<DropdownMenuTrigger asChild><Button>Open</Button></DropdownMenuTrigger>
```

### Forms
- All form dialogs use shared `FormDialog` (sticky header/footer)
- `<form>` needs `id`, submit button uses `form="id"`
- No nested modals — hide parent, show child, return on close
- >4 fields → `size="lg"` or larger

### Inline Editing
- Single-line: `Enter` to save, `Escape` to cancel
- Multi-line/rich text: `Ctrl+Enter`/`Cmd+Enter` to save, `Escape` to cancel

## Shared Component Registry

| Component | Path | Purpose |
|-----------|------|---------|
| `DataTable` | `components/shared/DataTable` | List/table views with sort, filter, pagination |
| `FormDialog` | `components/shared/FormDialog` | Form dialog with sticky header/footer |
| `DetailSheet` | `components/shared/DetailSheet` | Detail view sheet (sm/md/lg) |
| `ConfirmDialog` | `components/shared/ConfirmDialog` | Destructive/confirm/info dialogs |

## Styling
- **Tailwind only** — no custom CSS files
- **Inter** font (Latin + Vietnamese)
- Nav/link items: `text-foreground` (black), never `text-muted-foreground`
- Org units: `Building2` icon + `Badge variant="secondary"` + `OrgUnitCardPopover`
- Contacts: `User` icon + `Badge variant="secondary"` + `ContactCardPopoverById`

## Modules

```
universe → plan → engagement → section → objective → procedure
                      ↓
                   finding, document
```

Core: universe, plan, engagement, finding
Support: access, document, settings, teams

## Engagement V2 (Feature-Flagged)

Methodology-driven tree structure. Feature flag: `ENGAGEMENT_V2` in `SystemSetting`.

**Design docs**: `docs/methodology-driven-engagement-tree.md`, `docs/engagement-v2-progress.md`

### V2 Architecture
- V2 UI lives in `src/features/engagement-v2/` — fully isolated from V1
- V2 API routes under `/api/v2/engagement/...` and `/api/settings/methodologies/...`
- V2 server actions: `src/server/actions/methodology.ts`, `src/server/actions/engagementTree.ts`
- V1/V2 switch at `engagement/[id]/page.tsx` via `engagement.methodologyId != null`

### V2 Rules
- **Copy** V2-specific UI components (sidebar, router, layout) into `features/engagement-v2/`
- **Reuse** shared workpaper infra (`components/shared/workpaper/*`, `approvalEngine.ts`, `workpaperContent.ts`, `useWorkpaper.ts`)
- **Extend** shared actions additively (new `engagement_node` cases) — never modify existing V1 paths
- Entity type for V2 nodes: `engagement_node`
- Methodology definition stored as JSON blob on `Methodology` table
- `EngagementNode` is relational (mutable live data: content, approval, signoffs)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ddminh123)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ddminh123)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
