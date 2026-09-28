## threadforge

> This file is the single source of truth for any Claude agent (Claude Code, Claude design tools, or AI assistants) working on this repo. Read it fully before writing or editing anything.

# ThreadForge — Claude Context

This file is the single source of truth for any Claude agent (Claude Code, Claude design tools, or AI assistants) working on this repo. Read it fully before writing or editing anything.

---

## What this project is

**ThreadForge** is a niche SaaS platform for clothing brands. It lets them:
- Place designs on photorealistic garment templates using a canvas editor
- Manage their brand kit (logos, colors, fonts) per brand
- Organize mockups into collections / seasonal drops
- Export print-ready or digital assets (PNG, WebP, PDF)
- Subscribe to Pro or Studio plans via Stripe

Target users: small-to-mid clothing brands, streetwear labels, print-on-demand sellers.

---

## Tech stack

| Layer | Technology | Version |
|---|---|---|
| Framework | Next.js (App Router) | 14 |
| Language | TypeScript | 5 |
| Styling | Tailwind CSS + shadcn/ui | 3.x |
| Canvas editor | Fabric.js | 5 |
| ORM | Prisma | 5 |
| Database | PostgreSQL | any |
| Auth | Clerk | 5 |
| Storage + CDN | Cloudinary | 2 |
| Payments | Stripe | latest |
| Email | Resend | 3 |
| State (client) | Zustand | 4 |
| Validation | Zod | 3 |
| Deployment | Vercel | — |

---

## Directory structure

```
threadforge/
├── CLAUDE.md                        ← you are here
├── .env.example                     ← all required env vars with descriptions
├── .github/
│   ├── workflows/ci.yml             ← lint + typecheck + build on push/PR
│   └── ISSUE_TEMPLATE/
└── apps/
    └── web/                         ← the Next.js application (primary workspace)
        ├── prisma/
        │   ├── schema.prisma        ← canonical data model, edit this first
        │   └── seed.ts              ← seeds template rows (clothing garments)
        ├── public/
        │   ├── templates/           ← placeholder slot for local garment PNGs
        │   └── og/                  ← OpenGraph images
        └── src/
            ├── app/
            │   ├── (auth)/          ← Clerk sign-in / sign-up pages
            │   ├── (dashboard)/     ← authenticated app shell + all pages
            │   │   ├── layout.tsx   ← sidebar nav, UserButton
            │   │   ├── mockups/     ← main grid of user's mockups
            │   │   ├── brands/      ← brand kit management
            │   │   ├── collections/ ← lookbooks / drops
            │   │   ├── editor/      ← Fabric.js canvas editor per mockup
            │   │   └── settings/    ← account, billing, plan
            │   ├── (marketing)/     ← public landing page, pricing, about
            │   └── api/
            │       ├── mockups/     ← GET list, POST create
            │       ├── brands/      ← GET list, POST create
            │       ├── templates/   ← GET list (no auth required)
            │       ├── upload/      ← POST base64 → Cloudinary
            │       └── webhooks/stripe/ ← Stripe event handler
            ├── components/
            │   ├── ui/              ← shadcn/ui primitives only (Button, Dialog, etc.)
            │   ├── editor/          ← Fabric.js canvas, toolbar, color picker, layers panel
            │   ├── mockup/          ← MockupCard, MockupGrid, MockupViewer
            │   ├── brand/           ← BrandCard, BrandKit, ColorPalette
            │   ├── template/        ← TemplateGallery (filterable), TemplateCard
            │   └── shared/          ← UploadZone, Navbar, Sidebar
            ├── lib/
            │   ├── db.ts            ← singleton Prisma client
            │   ├── cloudinary.ts    ← upload, delete, URL helpers
            │   ├── stripe.ts        ← Stripe client + PLANS constant
            │   ├── validations.ts   ← all Zod schemas (source of truth for shapes)
            │   └── utils.ts         ← cn(), slugify(), formatBytes(), absoluteUrl()
            ├── hooks/               ← useEditor(), useMockup(), useBrand()
            └── types/
                └── index.ts         ← re-exports Prisma types + composite types
```

---

## Data model (Prisma)

Read `apps/web/prisma/schema.prisma` for the full schema. Key relationships:

```
User (Clerk identity)
  ├── Brand[]           — one user → many brands (streetwear labels, etc.)
  │     └── Mockup[]   — one brand → many mockups
  ├── Mockup[]          — mockups can also exist without a brand
  │     ├── Template    — every mockup references one garment template
  │     ├── Export[]    — download history (PNG/WebP/PDF)
  │     └── CollectionMockup[] — join table (many-to-many with Collection)
  ├── Collection[]      — lookbooks / seasonal drops
  └── Subscription      — Stripe subscription (one-to-one)

Template                — seeded rows, not user-created
  ├── category: TemplateCategory enum (TSHIRT, HOODIE, HAT, …)
  ├── imageUrl          — Cloudinary URL of the blank garment photo
  ├── maskUrl           — PNG mask for compositing the design layer
  └── designArea        — JSON { x, y, width, height, rotation } in canvas coords
```

When adding a new model or field: edit `schema.prisma` → run `npx prisma migrate dev --name <slug>` → update `src/types/index.ts` if a composite type is needed.

---

## Auth pattern

All authenticated API routes and server components follow this pattern:

```ts
import { auth } from "@clerk/nextjs/server";

const { userId } = await auth();           // Clerk user ID (string)
if (!userId) return redirect("/sign-in");  // or 401 in route handlers

const user = await db.user.findUnique({ where: { clerkId: userId } });
```

The `User.clerkId` column is the bridge between Clerk and the database. Never use Clerk's `userId` as a primary key in relations — always resolve to `User.id` (cuid) first.

Webhook to sync Clerk → DB is not yet implemented. Add it at `src/app/api/webhooks/clerk/route.ts` using Clerk's `user.created` event.

---

## API conventions

- All API routes live under `src/app/api/`
- Routes return `{ success: true, data: T }` or `{ success: false, error: string | ZodFlatten }`
- Validate request bodies with Zod schemas from `src/lib/validations.ts` — never validate inline
- Always authenticate first, then validate, then query
- Use `NextResponse.json()` with explicit status codes (201 for creates, 422 for validation errors, 401 for auth)

---

## Canvas editor

The mockup editor lives in `src/components/editor/` and `src/app/(dashboard)/editor/`.

- **Library:** Fabric.js 5.x (not v6 — API is different)
- **Canvas state** is serialized via `canvas.toJSON()` and stored in `Mockup.canvasJson`
- **Restore** via `canvas.loadFromJSON(mockup.canvasJson)`
- The garment base image is always the bottom-most object (`sendObjectToBack`)
- Design placement is constrained by `Template.designArea` — clip or snap to those bounds
- Export: `canvas.toDataURL({ format: 'png', multiplier: exportScale })` → POST to `/api/upload`

When adding editor features, keep side effects (upload, DB save) outside the canvas components — pass callbacks down from the page.

---

## Cloudinary conventions

- Upload preset: `threadforge_uploads` (configure in Cloudinary dashboard — unsigned)
- Folder structure: `threadforge/designs/`, `threadforge/logos/`, `threadforge/templates/`
- Always use `getMockupUrl(publicId, width)` from `src/lib/cloudinary.ts` to build URLs — never construct them manually
- Template images are stored in Cloudinary but seeded with hardcoded URLs in `prisma/seed.ts`

---

## Stripe / billing

Plans are defined in `src/lib/stripe.ts` → `PLANS` constant. Three tiers:

| Plan | Price | Mockups/mo | Brands | Exports/mo |
|---|---|---|---|---|
| FREE | $0 | 10 | 1 | 5 |
| PRO | $19 | 200 | 5 | 100 |
| STUDIO | $49 | unlimited | unlimited | unlimited |

Stripe webhooks are handled at `/api/webhooks/stripe`. The `Subscription` table is the source of truth for plan status — not Clerk metadata. Check `user.subscription.status === "ACTIVE"` for gating.

---

## Styling rules

- **shadcn/ui primitives** (`src/components/ui/`) are the base — never re-implement Button, Dialog, Select, etc.
- **Tailwind only** — no CSS modules, no inline styles, no styled-components
- Use `cn()` from `src/lib/utils.ts` for conditional class merging (clsx + tailwind-merge)
- Color system uses CSS variables (defined in `globals.css`) — always use semantic tokens (`bg-background`, `text-muted-foreground`) not raw colors
- Dark mode is class-based (`dark:` variants) — the `<html>` class is controlled by a future ThemeProvider
- Spacing scale: use Tailwind defaults — no magic numbers
- Border radius: use `rounded-lg` / `rounded-xl` for cards, `rounded-md` for inputs, `rounded-full` for pills/badges

---

## Design system (for Claude design tools)

### Brand identity

ThreadForge targets clothing brands — the aesthetic should feel **minimal, editorial, and tactile**. Think: clean white space, strong typography, subtle shadows, not flashy gradients.

### Color palette

```
Background:     #FFFFFF / hsl(0 0% 100%)
Foreground:     #0A0A0F / hsl(240 10% 3.9%)
Primary:        #0A0A0F (dark mode: #FAFAFA)
Muted:          #F4F4F5 / hsl(240 4.8% 95.9%)
Muted fg:       #71717A / hsl(240 3.8% 46.1%)
Border:         #E4E4E7 / hsl(240 5.9% 90%)
Destructive:    #EF4444 / hsl(0 84.2% 60.2%)
Accent (amber): #F59E0B — used only for PRO badge, premium indicators
```

Dark mode mirrors the above — backgrounds go deep charcoal, not pure black.

### Typography

- **Font:** Geist Sans (variable) for UI, Geist Mono for code/IDs
- **Scale:** text-xs (10–12px) for metadata/badges, text-sm (14px) for body, text-base (16px) for labels, text-2xl–text-4xl for headings
- **Weight:** font-medium for labels, font-semibold for headings, font-bold for hero/display
- **Tracking:** `tracking-tight` on large headings only

### Component patterns

```
Cards:          rounded-xl border bg-card shadow-sm hover:shadow-md
Buttons:        Default = black fill. Ghost = transparent. Outline = border only.
Badges:         rounded-full px-2 py-0.5 text-xs font-medium
Inputs:         border bg-background rounded-md px-3 py-2 text-sm
Empty states:   border-2 border-dashed rounded-xl flex-col items-center py-20
Upload zones:   border-2 border-dashed, border-primary on drag-active
```

### Layout

- Sidebar width: `w-56` (224px), fixed, not collapsible yet
- Main content: `max-w-7xl mx-auto p-6`
- Grid for mockup cards: `grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5`
- Grid gap: `gap-4`
- Section spacing: `space-y-6`

### Icon library

Lucide React — always use `h-4 w-4` for inline icons, `h-5 w-5` for nav, `h-8 w-8` for feature icons. Never mix icon libraries.

---

## Environment variables

All vars are documented in `.env.example` at the repo root. For a new clone:

```bash
cp .env.example apps/web/.env.local
# Fill in: DATABASE_URL, Clerk keys, Cloudinary keys, Stripe keys, Resend key
```

Variables prefixed `NEXT_PUBLIC_` are exposed to the browser. Never put secrets in `NEXT_PUBLIC_` vars.

---

## Running locally

```bash
cd apps/web
npm install
npx prisma migrate dev       # applies migrations, generates client
npx prisma db seed           # seeds template rows
npm run dev                  # http://localhost:3000
```

Other useful commands:

```bash
npm run typecheck            # tsc --noEmit (run before committing)
npm run lint                 # eslint
npx prisma studio            # GUI database browser at localhost:5555
npx prisma migrate reset     # wipe + re-apply all migrations (dev only)
```

---

## What is NOT built yet (next priorities)

These are known gaps — if asked to implement them, check this list first:

1. **Clerk webhook** (`/api/webhooks/clerk`) — syncs new Clerk users to the `User` table on `user.created` event. Without this, users who sign up won't have a DB row yet.
2. **Editor page** (`/dashboard/editor/[id]`) — the actual page wrapping `MockupCanvas` + `EditorToolbar`. The components exist but the page route doesn't.
3. **Brands page** (`/dashboard/brands`) — list + create brand form. Schema and API route exist.
4. **Collections page** (`/dashboard/collections`) — list + create. Schema exists.
5. **Settings / billing page** — upgrade flow using Stripe Checkout.
6. **ThemeProvider** — dark mode toggle.
7. **`/api/mockups/[id]`** — PATCH (save canvas), DELETE, GET single.
8. **`/api/templates`** — GET public list for the template gallery.
9. **Zustand editor store** — `src/hooks/useEditor.ts` is a stub.
10. **Export pipeline** — canvas → dataURL → Cloudinary → `Export` row.

---

## Commit conventions

Format: `type(scope): description`

Types: `feat`, `fix`, `chore`, `refactor`, `docs`, `style`, `test`
Scope: `editor`, `brands`, `mockups`, `auth`, `billing`, `db`, `api`, `ui`

Examples:
```
feat(editor): add text color picker to toolbar
fix(billing): handle past_due subscription status in gating check
chore(db): add index on Mockup.userId
```

Never reference issue numbers in the subject line — put them in the body.
Never add AI tool attribution in commit messages.

---
> Source: [zyadhajaji/threadforge](https://github.com/zyadhajaji/threadforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
