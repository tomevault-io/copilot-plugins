## inje-uri

> **Generated:** 2026-04-08T00:41:54Z

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-04-08T00:41:54Z
**Commit:** 0dfe671
**Branch:** main

## OVERVIEW

Mobile dating app prototype (인제우리) for university students. Next.js 16 App Router + React 19 + TypeScript 5.9 + Tailwind 4 + Prisma 6.19 (PostgreSQL). Mobile-first with 430px centered app column on desktop.

## STRUCTURE

```
Inje_uri/
├── src/
│   ├── app/              # App Router routes (see below)
│   │   ├── (main)/       # Route group: all authenticated pages share layout (BottomNav)
│   │   └── layout.tsx    # Root layout (ToastProvider, globals)
│   ├── components/       # See src/components/AGENTS.md
│   └── lib/              # See src/lib/AGENTS.md
├── prisma/               # See prisma/AGENTS.md
├── docs/
│   └── ui-policy.md      # UI/UX design constraints (MUST READ before UI work)
├── next.config.ts        # Images unoptimized, remote: picsum.photos + i.pravatar.cc
├── tsconfig.json         # strict, @/* -> ./src/*, bundler resolution
└── postcss.config.mjs    # @tailwindcss/postcss (no tailwind.config file)
```

## ROUTES

All under `src/app/(main)/` sharing BottomNav layout:

| Route | Page | Domain |
|-------|------|--------|
| `/match` | Daily recommendations | Match |
| `/match/[id]` | Profile detail | Match |
| `/self-date` | 24h story feed | Self-date |
| `/self-date/create` | Create story | Self-date |
| `/self-date/[id]` | Story detail | Self-date |
| `/self-date/mine` | My stories | Self-date |
| `/chat` | Chat list | Chat |
| `/chat/[id]` | Chat room | Chat |
| `/interest` | Received/sent interests | Interest |
| `/my` | My page | Profile |
| `/my/profile`, `/my/profile/edit` | Profile view/edit | Profile |
| `/my/settings` | Settings | Profile |
| `/my/ideal-type` | Ideal type prefs | Profile |
| `/my/posts` | My posts | Profile |

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add new page | `src/app/(main)/{route}/page.tsx` | All pages are `'use client'`; use PageContainer from layout components |
| Add UI primitive | `src/components/ui/` | Export from `ui/index.ts`; extend HTML attributes for props |
| Add domain component | `src/components/{domain}/` | Export from `{domain}/index.ts` |
| Mock data | `src/lib/data/` | Will be replaced by repository layer eventually |
| Types | `src/lib/types/` | Domain interfaces + keyword catalogs |
| Utility functions | `src/lib/utils/` | Pure functions; side-effects isolated in dedicated files |
| Navigation helpers | `src/lib/navigation/` | Custom routing context, safe-back, view state |
| Design tokens | `src/app/globals.css` | CSS variables for colors, spacing, nav-height |
| DB schema changes | `prisma/schema.prisma` | Follow prisma/MIGRATION_RULES.md strictly |
| UI/UX rules | `docs/ui-policy.md` | Spacing, text limits, action policy, mobile safety |

## CONVENTIONS

- **Imports**: Use `@/` alias for all src/ imports
- **Components**: PascalCase files, named exports only (no default), barrel index.ts per folder
- **Props**: Interface `XProps`, extend native HTMLAttributes for primitives, forwardRef where needed
- **Styling**: Tailwind utilities + CSS variables from globals.css. No CSS modules. Spacing scale: 8/12/16/20/24
- **Client/Server**: `'use client'` for hooks/state/DOM; display-only components stay server-compatible
- **Data layer**: Currently mock (src/lib/data). No API routes or repository layer yet. Prisma schema exists but isn't wired
- **Build scripts**: `npm run dev` / `npm run build` use `--webpack` flag explicitly
- **No linter config files**: ESLint installed but no .eslintrc. No Prettier config

## ANTI-PATTERNS (THIS PROJECT)

- **Selection limit mismatch**: prisma/seed.js and src/lib/types/keywords.ts disagree on maxSelections (personality: 3 vs 5, interests: 5 vs 7). Reconcile before backend integration
- **Active chat max not enforced**: README says max 5 active chats but no code constant enforces it
- **README out of sync**: References `src/data`, `src/lib/repositories`, `src/app/api` — none exist
- **Migration rules**: Never reorder shared migrations. Never edit already-shared migrations. See prisma/MIGRATION_RULES.md

## UNIQUE STYLES

- Mobile-first: `.app-container` max 430px centered. Safe area handling via `env(safe-area-inset-bottom)`
- Navigation: Custom history/viewState/safe-back system in `src/lib/navigation/` — not just Next.js router
- Stories: 24-hour expiry model with time-remaining calculations in `src/lib/utils/feed.ts`
- Chat: Expiry warning system (CHAT_EXPIRY_WARNING_MINUTES = 60) in `src/lib/data/chats.ts`

## COMMANDS

```bash
npm install              # Install deps
npm run dev              # Dev server (webpack)
npm run build            # Production build (webpack)
npm run start            # Start production server
npm run lint             # ESLint

# Prisma
npm run prisma:generate        # Generate client
npm run prisma:format          # Format schema
npm run prisma:validate        # Validate schema
npm run prisma:migrate:dev     # Dev migration
npm run prisma:migrate:deploy  # Production migration
npm run prisma:seed            # Seed baseline data
npm run prisma:studio          # DB GUI
```

## ENV VARS

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | Yes | PostgreSQL connection string |

See `.env.example` for format.

## NOTES

- No CI/CD pipeline configured. Remote branch `deploy-setup` may contain WIP
- Images unoptimized (`next.config.ts`) — Next.js image optimization disabled
- Remote image hosts whitelisted: `picsum.photos`, `i.pravatar.cc` only
- No tests exist in the repository
- TypeScript 5.9 deprecates `baseUrl` in tsconfig — warning appears in VS Code but build is unaffected

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codaengii2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/codaengii2)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
