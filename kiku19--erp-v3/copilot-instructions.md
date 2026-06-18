## erp-v3

> - Next.js 16 (App Router), TypeScript (strict), Tailwind CSS v4

# CLAUDE.md — ERP v3

## Stack
- Next.js 16 (App Router), TypeScript (strict), Tailwind CSS v4
- PostgreSQL + Prisma ORM v7
- Swagger (swagger-jsdoc + swagger-ui-react), OpenAPI 3.0
- Vitest (unit), Playwright (E2E), Storybook (components)
- Path alias: `@/*` → project root

## Hard Rules

1. **No raw colors.** Use only design tokens from `app/globals.css`. Use `bg-primary`, `text-foreground`, `var(--border)`, etc. Hardcoded hex/rgb = violation.
   - ❌ `className="bg-[#3b82f6]"` · `color: rgb(59,130,246)` · `border: 1px solid gray` · `text-white` · `bg-blue-500` · `opacity-50` for hover states
   - ✅ `bg-primary` · `text-foreground` · `border-[var(--border)]` · `bg-primary-hover` · `text-muted-foreground`

2. **Every API route must have `@swagger` JSDoc** with summary, request/response schemas, and status codes (200, 400, 401, 404, 500). Docs at `/api-doc`.
   - ❌ Bare `export async function GET()` with no JSDoc · Missing response schema · Documenting only 200 and skipping error codes · Writing OpenAPI in a separate YAML file instead of inline JSDoc
   - ✅ Full `@swagger` block above every handler with summary, `requestBody`, `responses` (200/400/401/404/500), and schema definitions

3. **TDD: Red → Green → Refactor.** Write failing test first, then implement. No feature ships without tests. **Every component/page/route MUST have unit tests, E2E tests, AND stories written BEFORE implementation. No exceptions.**
   - ❌ Writing the component first then tests after · Shipping a page with no `*.test.tsx` · Skipping stories for "simple" components · Writing only happy-path tests · Adding E2E later as a "follow-up task"
   - ✅ `*.test.tsx` + `*.stories.tsx` + `e2e/*.spec.ts` exist and fail BEFORE the implementation is written · Both happy and error paths covered

4. **Prisma for all DB access.** No raw SQL. Run `npx prisma generate` + `npx prisma migrate dev` after schema changes.
   - ❌ `prisma.$queryRaw` · `prisma.$executeRawUnsafe` · Template-literal SQL · Knex/pg/mysql2 imports · Forgetting to run `prisma generate` after schema edits
   - ✅ `prisma.user.findMany()` · `prisma.order.create()` · Prisma Client methods only · Migrations via `prisma migrate dev`

5. **Named exports** everywhere (except Next.js pages/layouts).
   - ❌ `export default function MyComponent()` in a UI component · `export default class` · `module.exports =`
   - ✅ `export function MyComponent()` · `export const myUtil =` · Only `page.tsx`/`layout.tsx` use default exports

6. **`@/` path alias** for imports outside current directory.
   - ❌ `import { Button } from '../../../components/ui/button'` · `import { cn } from '../../lib/utils'` · Any `../` reaching outside the current directory
   - ✅ `import { Button } from '@/components/ui/button'` · `import { cn } from '@/lib/utils'` · Relative `./sibling` within the same directory is fine

7. **Use the UI system. No reinventing.** All pages and features MUST use components from `@/components/ui/`. The available primitives are: `Avatar`, `Badge`, `Button`, `Card`, `Checkbox`, `DataTable`, `Divider`, `DropdownMenu`, `Input`, `Select`, `Sidebar`, `Table`, `Textarea`, `Toggle`, `Typeahead`. **Never** build ad-hoc replacements. If a UI need isn't covered, **extend the system first** by creating a new component in `components/ui/` (with tests + stories) before using it in a feature.
   - ❌ `<button className="...">` instead of `<Button>` · `<input type="text" />` instead of `<Input>` · Hand-built `<div onClick>` dropdown instead of `<DropdownMenu>` · `<table>` instead of `<DataTable>` · Importing `@radix-ui/react-select` directly in a page · Installing and using `shadcn/ui` components without wrapping in `components/ui/`
   - ✅ `<Button variant="primary">` · `<Input placeholder="..." />` · `<Select options={...}>` · New primitive → add to `components/ui/` with tests + stories first, then use

8. **Tenant isolation · Soft delete · UTC dates.** All DB ops scoped to `tenantId`. No hard deletes — set `isDeleted: true`. All reads filter `isDeleted: false`. Dates in UTC via `new Date()`. See `app/api/CLAUDE.md` for full reference.
   - ❌ Any query missing `tenantId` · `prisma.*.delete()` / `deleteMany()` · Query without `isDeleted: false` · Local timezone dates
   - ✅ `where: { tenantId, isDeleted: false }` on reads · `data: { tenantId }` on creates · `data: { isDeleted: true }` for deletes · `new Date()` for timestamps

9. **Worktrees must NEVER touch the database schema.** All worktrees share a single PostgreSQL database. Schema modifications (`prisma migrate dev`, `prisma db push`, `prisma migrate reset`, editing `prisma/schema.prisma`) must ONLY happen in the **main working directory** (`/home/kishore/Project/erp-v3`). Worktrees may only run `npx prisma generate` (client codegen, safe). If a feature requires schema changes, make them in the main branch first, then start the worktree.
   - ❌ Running `prisma migrate dev` in a worktree · Running `prisma db push` in a worktree · Editing `prisma/schema.prisma` in a worktree · Running `prisma migrate reset` in a worktree
   - ✅ Schema changes in main working directory only · `npx prisma generate` in worktree (safe) · Worktree code works with the existing DB schema

## Design Tokens (from `app/globals.css`)
- **Colors:** `background`, `foreground`, `primary`, `secondary`, `muted`, `accent`, `destructive`, `card`, `border`, `input`, `ring` — each has a `-foreground` variant
- **Status:** `success`, `warning`, `error`, `info` — each has `-bg` and `-foreground` variants
- **Active:** `primary-active`, `primary-active-foreground` — use for active/selected states (e.g., sidebar nav items)
- **Hover:** `primary-hover`, `secondary-hover`, `destructive-hover`, `muted-hover` — use these for interactive states, never opacity hacks or mismatched tokens
- **Animation:** `duration-fast` (150ms), `duration-normal` (200ms), `duration-slow` (300ms), `ease-default`, `scale-press` (0.98)
- **Radius:** `sm` (4px), `md` (8px), `lg` (12px), `pill` (9999px)
- **Font:** `font-primary` / `font-secondary` (Inter)
- **Type Scale** (defined as `--text-*` in globals.css `@theme inline`, generates Tailwind `text-*` utilities):
  | Class | Size | Usage |
  |---|---|---|
  | `text-micro` | 10px | Badges, tiny labels, category headers |
  | `text-caption` | 11px | Hints, secondary info, table column headers |
  | `text-detail` | 12px | Table cells, compact data, small button text |
  | `text-body-sm` | 13px | Form labels, list item names, descriptions |
  | `text-body` | 14px | Default body text, inputs, section titles |
  | `text-subhead` | 16px | Modal titles, card headings |
  | `text-heading` | 18px | Page section headings |
  | `text-title` | 23px | Page titles |
  | `text-display` | 27px | Hero/display text |
  - Use ONLY these classes for font sizing. No hardcoded pixel values or default Tailwind text sizes.
  - ❌ `text-[13px]` · `text-[11px]` · `text-sm` · `text-xs` · Any hardcoded pixel font size
  - ✅ `text-body-sm` · `text-caption` · `text-subhead` · `text-detail`

## Tests

- **Unit** (`*.test.ts` co-located): `lib/`, API routes, hooks
- **E2E** (`e2e/*.spec.ts`): every page and user flow
- **Stories** (`*.stories.tsx` co-located): every UI component
- Done = all tests pass, no regressions, happy + error paths covered

---
> Source: [kiku19/erp-v3](https://github.com/kiku19/erp-v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
