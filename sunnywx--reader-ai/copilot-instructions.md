## reader-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**reader-ai** is a self-hosted AI-enhanced personal book reader. Users browse their local book files (PDF, EPUB, Markdown) through a web UI, with planned AI chat/RAG capabilities.

## Monorepo Structure

```
reader-ai/
├── web/              # Next.js 14 frontend (TypeScript)
├── server-nodejs/    # Fastify backend (Node.js, ESM)
├── server-py/        # FastAPI backend (Python, early stage)
└── docker-compose.yml  # Only runs MongoDB (port 27018)
```

## Commands

### Frontend (`web/`)
```bash
pnpm dev        # Start dev server on port 4000
pnpm build      # Production build
pnpm lint       # ESLint
pnpm copy:pdfjs # Copy pdf.js dist to public/ (run after install)
pnpm tw:view    # Preview Tailwind config on port 3002
```

### Backend (`server-nodejs/`)
```bash
pnpm dev        # Start with nodemon on port 3001
```

### Database
```bash
docker compose up -d    # Start MongoDB on port 27018
```

### Python server (`server-py/`)
```bash
uv run fastapi dev web.py   # Early-stage, mostly a stub
```

## Environment Variables

**Frontend** (`web/.env.local`):
- `NEXT_PUBLIC_PROXY_URL` — URL of the Node backend (default: `http://127.0.0.1:3001`)

**Backend** (`server-nodejs/.env`):
- `PORT` — server port (default: `3001`)
- `DB_URI` — MongoDB connection string
- `LOCAL_BOOK_DIR` — path to local books directory (default: `~/books`)

## Architecture

### Frontend (Next.js Pages Router)

- **Pages** (`src/pages/`) map directly to routes: `books.tsx`, `chat.tsx`, `read-later.tsx`, `notes.tsx`, `models.tsx`, `settings.tsx`
- **Book viewer** lives at `/book/[...slug]` — handles PDF (via `react-pdf`/`pdfjs-dist`), EPUB (via `react-reader`/`epubjs`), and Markdown
- **State** is managed with Zustand stores in `src/store/`: `book-store.ts` (book list, online mode) and `layout-store.ts`
- **API calls** all go through `src/lib/utils.ts` helpers: `proxyUrl()` prefixes paths with the backend URL, `request()` and `postRequest()` wrap fetch
- **UI** uses Radix UI Themes (`@radix-ui/themes`) as the base, with shadcn/ui components in `src/components/ui/` and Tailwind for custom styling. SCSS modules are used alongside Tailwind where needed
- **Prototype screens** in `src/components/prototype/` are reference UI mockups, not production components

### Backend (Fastify, Node.js ESM)

- **Entry**: `app.js` registers plugins then starts the server
- **Plugin loading**: The `api-docs` plugin (`plugins/api-docs.js`) uses `@fastify/autoload` to automatically load all files from `routes/` — adding a new route file there is sufficient
- **Routes** follow Fastify's schema-first pattern with JSON Schema for request/response; schemas appear in the Swagger UI at `http://localhost:3001/docs`
- **MongoDB** is accessed via `fast.mongo.db` (decorated by `plugins/mongo-connector.js`)
- **Local book serving**: `routes/book.js` reads `LOCAL_BOOK_DIR` from env, serves directory listings (`/local-books`), file blobs (`/book-blob`), and manages the read-later list in MongoDB

## UI & Design System

Full spec: **`DESIGN.md`** — "The Nocturnal Scholar / Velvet Library" dark mode design system.

### Dark Mode Implementation
- Dark mode is toggled via `useLayoutStore().toggleDarkMode`, stored in `localStorage` (`x-reader-dark-mode`)
- Radix UI `<Theme appearance="dark">` adds `.dark` class to the root; Tailwind is configured with `darkMode: 'class'` to respond to it
- Design tokens are defined as CSS variables in `globals.css` under `.dark { --ds-* }` — always use these for dark-mode colors, not hardcoded values

### Key Design Tokens (dark mode)
| Token | Value | Usage |
|---|---|---|
| `--ds-surface` | `#0b1326` | Base / app background |
| `--ds-surface-container-low` | `#131b2e` | Sidebar |
| `--ds-surface-container` | `#171f33` | TopBar glass, reading cards |
| `--ds-surface-container-high` | `#222840` | Active / hover states |
| `--ds-surface-container-highest` | `#2d3449` | Modals, dropdowns |
| `--ds-on-surface` | `#dae2fd` | Headlines, primary text |
| `--ds-on-surface-variant` | `#bdc8d1` | Body / secondary text |
| `--ds-primary` | `#8ed5ff` | Accent, labels |
| `--ds-outline-variant` | `#3e484f` | Ghost borders (use at 20% opacity) |
| `--ds-ambient-shadow` | `0 16px 32px rgba(6,14,32,0.6)` | Floating element shadows |

### Design Rules (enforced in code)
- **No 1px solid borders** — boundaries defined by background color shifts only (ghost border `outline-variant` at 20% opacity if legally required)
- **No pure black** — use surface tokens; no `#000000`
- **No dividers in lists** — use vertical spacing (`mb-8` / 2rem) between items
- **Hover/active states** — use `--ds-surface-container-high`, not bright colors like `bg-blue-100`
- **TopBar** — glassmorphism: `rgba(23,31,51,0.85)` + `backdrop-blur-[12px]`
- **Tailwind opacity modifier** (`bg-[var(...)]/80`) does not work with CSS variable colors — use `rgba()` directly instead

### Data Flow

```
Browser → Next.js page → proxyUrl() → Fastify (port 3001) → MongoDB / local filesystem
```

The Next.js frontend has no own API server for business logic — all `/api/*` routes in `web/src/pages/api/` are thin pass-throughs or config endpoints (e.g., `/api/models`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sunnywx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sunnywx)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
