## galing-pup

> **Domain:** Academic Research Repository (CCIS Department).

# GEMINI.md

## Project Context

**Name:** Galing-PUP
**Domain:** Academic Research Repository (CCIS Department).
**Mission:** Transition research storage from manual distribution to a centralized web platform.

**Core Functional Requirements:**

- **Search Engine:** Filter by Keyword, Topic, Author, Year, and Research Type.
- **Access Control (RBAC):**
  - **Viewer (Unregistered):** Search and view abstracts only. No downloads.
  - **Registered User:** Verified account. Can download PDFs, bookmark studies, and generate citations.
  - **Staff (College Admin):** Can upload, edit, and delete documents for their specific college only. Restricted from User Management and Approval workflows.
  - **Super Admin:** Full system control. Manages users, roles, global content, analytics, and settings.
  - **Owner:** Highest privilege level. Functionally equivalent to Super Admin but reserved for the system owner.
- **Data Structure:** Categorized by Subject and Publication Year.
- **File Storage:** Local storage in `public/uploads` (Development/Testing).
- **Security:** Role-Based Access Control (RBAC) via Supabase Auth and Custom Admin Layout Shell.
  - **Note:** `username` field is used as Full Name for Admin/Staff users.
- **AI Integration:** Use Google Gemini to generate summaries of research materials automatically.

## Tech Stack

- **Framework:** Next.js 16 (App Router)
- **Runtime:** Bun (v1.x+)
- **Language:** TypeScript
- **Authentication:** Supabase Auth
- **Database:** Supabase (PostgreSQL)
- **ORM:** Prisma
- **AI Model:** Google Gemini (via `google-generative-ai` SDK)
- **Styling:** Tailwind CSS v4, shadcn/ui
- **Icons:** Lucide React
- **Testing:** Buntest (Unit), Cypress (E2E)
- **Deployment:** Vercel

## Key Commands

- **Install:** `bun install`
- **Dev Server:** `bun run dev`
- **Database:**
  - Generate: `bunx prisma generate`
  - Migrate: `bunx prisma migrate dev`
  - Seed: `bunx prisma db seed`
- **Test:** `bun test`

## Directory Structure

- `app/` (Next.js App Router)
  - `(public)/`: Public facing pages (browse, paper, pricing)
  - `(auth)/`: Authentication routes (signin, verify-otp)
  - `admin/`: Protected admin dashboard
  - `api/`: Route Handlers
- `components/`: Reusable UI components (shadcn)
- `lib/`: Utility functions (Supabase client, Gemini wrapper)
- `prisma/`: Schema and seed data
- `supabase/`: Local config

## Coding Standards (Strict)

- **Routing:** Use kebab-case for folders (`app/request-access`).
- **Data Fetching:** Prefer **Server Components** over `useEffect` for initial data load.
- **Comments:** Add JSDoc to every function. Explain inputs, outputs, and what it does.
- **Imports:** Use absolute imports (`@/components/...`).
- **Type Safety:** No `any`. Define interfaces in `types/` or co-located if specific.
- **Naming:**
  - `camelCase` for variables/functions.
  - `PascalCase` for Components/Interfaces.
  - `SCREAMING_SNAKE_CASE` for environment variables.
  - `UPPER_SNAKE_CASE` for constants.
  - `camelCase` for prisma schema.
  - `snake_case` for mapping database columns.
- **Color Scheme:** Use official PUP colors from `globals.css` instead of custom hex codes:
  - **Maroon:** `pup-maroon` - Primary brand color
  - **Gold Light:** `pup-gold-light` - Light accent color
  - **Gold Dark:** `pup-gold-dark` - Dark accent color
  - Apply via Tailwind: `bg-pup-maroon`, `text-pup-gold-light`, `border-pup-gold-dark`, etc.

## Negative Constraints (DO NOT DO)

- **NO NPM/YARN:** Always use `bun` for package management.
- **NO RAW SQL:** Always use Prisma Client for database operations.
- **NO SECRETS:** Never hardcode API keys; use `process.env`.
- **NO CLASS COMPONENTS:** Use React Functional Components only.
- **NO REDUNDANT CODE:** Refactor and remove redundant code.
- **NO REDUNDANT COMMENTS:** Remove redundant comments.
- **NO MAGIC NUMBERS:** Use constants for magic numbers.
- **NO MAGIC STRINGS:** Use constants for magic strings.

## Git & Version Control

**Commit Message Convention:**

- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation
- `style:` Formatting
- `refactor:` Code restructuring
- `test:` Testing
- `chore:` Config/Maintenance

---
> Source: [Galing-PUP/Galing-PUP](https://github.com/Galing-PUP/Galing-PUP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
