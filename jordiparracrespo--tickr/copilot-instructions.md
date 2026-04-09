## tickr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
bun run dev      # Start dev server at localhost:3000
bun run build    # Production build
bun run lint     # ESLint
```

**Always use `bun`, never `npm` or `npx`.**

After any schema change:
```bash
bunx prisma db push          # Sync schema to DB (dev only — use --accept-data-loss for column type changes)
bunx prisma generate         # Regenerate Prisma client (always run after db push)
```

## Architecture

**Stack:** Next.js 16 App Router · Prisma 7 (Neon/PostgreSQL) · NextAuth.js v4 (JWT) · next-intl v4 · shadcn/ui · Tailwind CSS 4 · Resend · Cloudflare R2

### Data hierarchy

```
User → UserSettings (1:1)
User → Organization → Customer → Project → Session
                   └──────────────────────→ Invoice
```

Every entity is scoped to a `userId`. API routes always filter by `userId` extracted from the JWT session.

### Route structure

All app routes live under `src/app/[locale]/` — the locale segment is injected by next-intl middleware.

- `[locale]/(auth)/` — public pages (login, forgot-password, reset-password)
- `[locale]/(dashboard)/` — protected pages; layout server-checks session and renders sidebar + mobile-nav
- `src/middleware.ts` — next-intl middleware handles locale detection and URL prefixing

### Internationalization (next-intl v4)

- Supported locales: `en-US`, `es-ES`, `en-GB`, `de-DE`, `fr-FR`, `pt-BR` (default: `en-US`)
- Config: `src/i18n/routing.ts` (`defineRouting`) · `src/i18n/request.ts` · `src/i18n/navigation.ts` (`createNavigation`)
- Message files: `messages/<locale>.json`
- **Always import `Link`, `usePathname`, `useRouter` from `@/i18n/navigation`** in client components — these are locale-aware wrappers
- **Server-side `redirect()`** (layout.tsx, page.tsx server components) uses `next/navigation` — the middleware automatically adds the locale prefix
- `useLocale()` from `next-intl` returns the current locale string
- `useUserPrefs()` from `src/lib/locale-context.tsx` returns `{ timezone, currency, loading, refresh }` — fetched from `/api/user/settings`

### Context providers (`src/components/providers.tsx`)

```
SessionProvider (NextAuth)
  └── LocaleProvider (UserPrefs: timezone + currency)
        └── OrganizationProvider   ← persists currentOrgId to localStorage
              └── CustomerProvider ← persists currentCustomerId to localStorage; resets when org changes
```

### User settings

- `UserSettings` model — separate table, 1:1 with `User` via `@unique userId`
- Fields: `locale`, `timezone`, `currency` (defaults: `en-US`, `Europe/Madrid`, `EUR`)
- API: `GET /api/user/settings` (upserts on first access) · `PUT /api/user/settings` (profile name, locale/tz/currency, password change)
- Settings page at `/settings` — accessible via user avatar dropdown at bottom of sidebar

## Key conventions

**Params must be awaited** (Next.js 15+):
```ts
export async function GET(req, { params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
}
```

**Prisma client** is generated to `src/generated/prisma` — import from there or via the `@/generated/prisma` alias.

**User ID type**: NextAuth stores `session.user.id` as a string; always `parseInt()` before Prisma queries.

**API route pattern**:
```ts
const session = await getServerSession(authOptions);
if (!session?.user?.id) return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
const userId = parseInt(session.user.id);
```

**Dynamic `where` filters**: build a `where: any = { userId }` object and spread conditions onto it. For cross-relation filters use nested Prisma relation syntax (`project: { customer: { organizationId: ... } }`).

**Parallel Prisma queries**: use `Promise.all([findMany, count, aggregate])` for paginated list endpoints.

**Prisma error P2002** = unique constraint violation — handle explicitly and return a 400.

**Never define a React component inside another component's render function** — it causes remounting on every render. Inline JSX as a variable (`const formBody = <div>…</div>`) and render it with `{formBody}`.

**Session duration**: `totalDuration` is stored in seconds. Active pauses without a `resumedAt` are calculated at read time. Use `formatDuration` / `formatHours` from `src/lib/utils/duration.ts`.

**Monetary values**: stored as `Decimal @db.Decimal(10,2)` in Prisma. Cast with `Number()` before arithmetic. Format with `new Intl.NumberFormat(locale, { style: "currency", currency })` using locale from `useLocale()` (next-intl) and currency from `useUserPrefs()`.

**Invoice numbers**: auto-incremented integers per user, generated server-side via `MAX(invoiceNumber) + 1`.

**Passwords**: bcryptjs, rounds = 10 (register) / 12 (reset). Google OAuth users have `password = null`.

**Email**: Resend via `src/app/api/auth/forgot-password/route.ts`. Reset tokens expire in 1 hour and are single-use (`usedAt` timestamp).

**File uploads**: Cloudflare R2 via `src/app/api/upload/route.ts`. Public URLs use `R2_PUBLIC_URL`.

**Timezone-aware date filtering**: sessions and customer pages use DST-aware utilities (`tzOffsetHours`, `tzStartISO`, `tzEndISO`, `getTzToday`) — always pass the user's timezone string from `useUserPrefs()`. Never use plain `new Date()` for day boundaries.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JordiParraCrespo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JordiParraCrespo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
