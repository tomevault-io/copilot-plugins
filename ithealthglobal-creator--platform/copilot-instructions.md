## platform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

This is a **multi-tenant IT Modernisation Platform** (codenamed Servolu), built to be white-labelled for multiple companies. The platform operator is Servolu; tenant companies (admins, partners, customers) each get their own branded experience. The current scope spans the **Admin area**, **Customer Portal**, **Super Admin platform shell**, and the **public website**.

The platform supports a company hierarchy: `platform` (Servolu) → `admin` (resellers/MSPs) → `customer`/`partner` (end clients). Roles include `super_admin` (Servolu staff), `admin`, `partner`, and `customer`.

## Tech Stack

- **Framework**: Next.js (App Router), TypeScript
- **UI**: React, shadcn/ui, Tailwind CSS, IBM Carbon Icons (`@carbon/icons-react`), Poppins font
- **Backend**: Supabase (local Docker — Postgres, GoTrue auth, PostgREST, Studio)
- **State**: React context (auth + menu + branding + company)
- **All admin pages are client components** (`"use client"`)

## Development Commands

```bash
# Start local Supabase
npx supabase start

# Run Next.js dev server
npm run dev

# Run database migrations
npx supabase db push

# Reset database (applies migrations + seed)
npx supabase db reset
```

## Architecture

### Route Groups

- `(auth)/` — Login, reset password (unauthenticated)
- `(admin)/` — Admin shell wrapped in auth guard + sidebar + mega menu
- `(customer)/` — Customer portal (role=customer)
- `(super-admin)/` — Platform shell for super_admin role only
- `(public)/` — Public website (no auth)
- `(onboarding)/` — Get-started / assessment wizard
- `api/admin/users/` — Route Handlers for service_role operations (create user, reset password, delete user)

### Key Patterns

- **Auth guard** wraps `(admin)` layout; checks session + admin role, redirects to `/login` if missing
- **Super admin guard** wraps `(super-admin)` layout; requires `super_admin` role
- **Menu system** is database-driven (adjacency list, 4 levels): L1 = sidebar icons, L2 = mega menu tabs, L3/L4 = mega menu expanded
- **Menu context** fetches the full menu tree on login via `get_menu_tree(user_role)` security definer function
- **Supabase service_role key** is never exposed to the client — admin operations go through Next.js Route Handlers at `/api/admin/users/`
- **`profiles.email`** is a denormalized copy of `auth.users.email`, kept in sync by a database trigger
- **Branding** is per-company: `useBranding()` provides CSS vars + logo URLs; `BrandingProvider` is in `(admin)` and `(super-admin)` layouts
- **CompanyProvider** (`src/contexts/company-context.tsx`) provides `company` and `branding` for authenticated contexts
- **`resolveCompanyId()`** resolves the default company for public pages (supports `?company=slug` param)

### Database Tables

- `companies` — organizations with hierarchy (type, parent_company_id, domain, slug, tagline, support_email, contact_email)
- `profiles` — extends `auth.users` with company_id, role (`super_admin`/`admin`/`customer`/`partner`), display_name
- `company_branding` — per-company logos, colours, fonts
- `menu_items` — hierarchical menu (parent_id, level 1-4, sort_order, icon, route)
- `role_menu_access` — composite PK (role, menu_item_id), controls menu visibility per role

### RLS

- `menu_items` access goes through `get_menu_tree(user_role)` security definer function (avoids cross-table RLS)
- Admins have full read/write on `companies` and `profiles` within their hierarchy
- Non-admins read only their own company's data
- `super_admin` has cross-company access

## Conventions

- Desktop-only for this phase (no mobile/responsive)
- Light theme with dark sidebar
- Skeleton loaders for page content, spinner for auth check
- Toast notifications (shadcn Sonner) for CRUD success/error feedback
- Sort order in menu editor uses numeric input (no drag-and-drop yet)
- Use IBM Carbon icons (`@carbon/icons-react`) exclusively — never Lucide
- Use Poppins font exclusively — never Inter

## Design Spec

Full design spec: `docs/superpowers/specs/2026-04-02-ithealth-admin-platform-design.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ithealthglobal-creator)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ithealthglobal-creator)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
