## hack-the-globe

> Construction surplus materials marketplace with AI-powered matching and carbon accounting.

# AGENTS.md — ScrapYard (Hack the Globe 2026)

Construction surplus materials marketplace with AI-powered matching and carbon accounting.
Hackathon project for Hack the Globe 2026 (Sustainable Infrastructure theme).

## Project Structure

```
hack-the-globe/
  web/                # PRIMARY APP — Next.js 16 + Convex (TypeScript)
    src/
      components/     # Shared components (Layout, Logo, AuthContext, ui/)
      pages/          # Next.js Pages Router routes
      styles/         # Global CSS (Tailwind)
      lib/            # Utilities (cn helper)
    convex/           # Convex backend (schema, queries, mutations)
    public/           # Static assets (logos, icons)
    package.json
    components.json   # shadcn/ui config (radix-nova style)
  scrapyard/          # OLD MOCKUP — React + Vite prototype (JavaScript, do NOT develop here)
  pitch/              # Pitch plan, feedback notes
  docs/               # Hackathon guides, brainstorming, research
  .agents/skills/     # Convex agent skill definitions
```

**All active development happens in `web/`.** The `scrapyard/` directory is an old mockup prototype.

## Build / Lint / Dev Commands

Package manager: **Bun**

```bash
# All commands run from web/
cd web

# Install dependencies
bun install

# Development server (Next.js + Convex)
bun run dev

# Production build
bun run build

# Lint (ESLint 9 + next core-web-vitals + typescript)
bun run lint
```

There is **no test framework** configured.

## Tech Stack

| Layer         | Technology                                          |
|---------------|-----------------------------------------------------|
| Language      | TypeScript (`.tsx` / `.ts`)                         |
| Framework     | Next.js 16 (Pages Router)                           |
| React         | 19.2                                                |
| Backend / DB  | Convex (real-time cloud database)                   |
| AI / LLM      | OpenRouter AI SDK + Gemini (`google/gemini-3.1-flash-lite-preview`) |
| Styling       | Tailwind CSS 4 (`@tailwindcss/postcss`)             |
| UI Components | shadcn/ui (radix-nova style, Radix UI primitives)   |
| Icons         | lucide-react                                        |
| Charts        | Recharts (via shadcn chart wrapper)                 |
| Maps          | Leaflet + react-leaflet (dynamic import, no SSR)    |
| Validation    | Zod 4                                               |
| Font          | Outfit (Google Fonts)                               |
| Auth          | Custom localStorage-based username (no real auth)   |
| Pkg manager   | Bun                                                 |

## Convex Data Model

Three tables in `convex/schema.ts`:

### `listings`
title, category, quantity, unit, price, originalPrice, seller, sellerRating, sellerVerified, sellerTransactions, carbonSaved, image (optional string), images (optional array of storage IDs), condition, description, location, lat, lng

Search index: `by_category` (searchField: `title`, filterFields: `["category"]`)

### `requests`
title, category, budget, urgency, requester

### `messages`
listingId (`Id<"listings">`), sender, body

Index: `by_listingId` on `["listingId"]`

## Convex Backend Functions

| File               | Functions                                |
|--------------------|------------------------------------------|
| `listings.ts`      | `list`, `getById`, `listByCategory`, `create`, `remove`, `generateUploadUrl` |
| `messages.ts`      | `list`, `send`                           |
| `requests.ts`      | `list`, `create`, `remove`               |
| `seed.ts`          | `seedAll`, `resetAndSeed`                |

## Routes (Pages Router)

| Path                | Page File              | Description                              |
|---------------------|------------------------|------------------------------------------|
| `/`                 | `index.tsx`            | Home — tabs: Open Requests + My Listings |
| `/browse/[category]`| `browse/[category].tsx`| Filtered listing grid with search        |
| `/listing/[id]`     | `listing/[id].tsx`     | Single listing detail + seller info      |
| `/post`             | `post.tsx`             | Batch AI scanner — multi-item detection + bulk post |
| `/scan`             | `scan.tsx`             | Redirects to `/post?scan=1`              |
| `/map`              | `map.tsx`              | Leaflet map with listing pins (not in main nav) |
| `/carbon`           | `carbon.tsx`           | Carbon impact dashboard (3 charts)       |
| `/profile`          | `profile.tsx`          | User profile + sign out                  |
| `/request`          | `request.tsx`          | Chat-based AI request posting            |
| `/chat/[id]`        | `chat/[id].tsx`        | Real-time chat per listing               |
| `/api/scan`         | `api/scan.ts`          | AI scanner — multi-item detection (Gemini) |
| `/api/parse-request`| `api/parse-request.ts` | AI request parser — natural language to structured data |

## shadcn/ui Components (Installed)

button, card, input, badge, dialog, label, chart, tabs

Config: `radix-nova` style, no RSC (`rsc: false`), neutral base color, CSS variables enabled.

Install more with: `npx shadcn@latest add <component>`

## Shared Components

| Component         | File                      | Purpose                              |
|-------------------|---------------------------|--------------------------------------|
| `Layout`          | `components/Layout.tsx`   | App shell + 5-tab bottom navigation  |
| `Logo`            | `components/Logo.tsx`     | Logo image component                 |
| `AuthContext`      | `components/AuthContext.tsx` | React Context for localStorage auth |
| `NamePrompt`      | `components/NamePrompt.tsx`  | First-visit username entry prompt   |

## Code Style Guidelines

### TypeScript
- All code is TypeScript (`.tsx` / `.ts`)
- Use proper types for Convex documents and IDs
- Zod for runtime validation (AI scanner responses)

### Imports
- Path alias: `@/*` maps to `./src/*`
- shadcn/ui imports: `@/components/ui/button`
- Convex imports: `convex/_generated/api` and `convex/_generated/dataModel`
- Single quotes for import strings

### Component Patterns
- Default exports for page components
- Named exports for shared/ui components
- Props typed inline or with interfaces
- Pages Router conventions: `useRouter()` from `next/router`

### Styling
- **Tailwind CSS 4** for all styling — use utility classes
- **CSS custom properties** in `globals.css` for theme tokens (OKLCH color space)
- **shadcn/ui** components for consistent UI primitives
- **No inline `style={{}}` objects** — use Tailwind classes instead
- Animations: `fade-up`, `fade-in`, `slide-up` with stagger classes
- Mobile-first: max-width 430px app shell

### State Management
- **Convex `useQuery`** for data fetching (reactive subscriptions)
- **Convex `useMutation`** for writes
- **`useAuth()`** custom hook for username (localStorage)
- Local `useState` for UI state
- `useRouter()` from `next/router` for navigation

### Categories (Hardcoded)
```
lumber, steel, concrete, brick, glass, pipe, electrical, fixtures
```
These are currently defined as constants in multiple page files (not centralized).

## Environment Variables

| Variable                  | Purpose                          |
|---------------------------|----------------------------------|
| `NEXT_PUBLIC_CONVEX_URL`  | Convex deployment URL            |
| `OPENROUTER_API_KEY`      | API key for AI scanner endpoint  |

## Key Reminders for Agents

1. **Work in `web/`** — `scrapyard/` is an old mockup, do not develop there
2. Use **Bun**, not npm/yarn
3. Use **TypeScript** — this is not a JavaScript project
4. Use **Tailwind classes** for styling, not inline style objects
5. Use **shadcn/ui** components (install with `npx shadcn@latest add <name>`)
6. Read `convex/_generated/ai/guidelines.md` before writing Convex code
7. Data is real (Convex), not mocked — use `useQuery`/`useMutation`
8. Pages Router (`src/pages/`), NOT App Router
9. Run `bun run lint` after changes
10. New pages go in `src/pages/`, new shared components in `src/components/`
11. AI scanner is real (Gemini via OpenRouter), not a mock
12. No SSR for Leaflet/map components — use `next/dynamic` with `ssr: false`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/not-manu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/not-manu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
