## deducto

> - **Monorepo** with npm workspaces: `client/`, `server/`, `shared/`

# Deducto PWA

## Architecture
- **Monorepo** with npm workspaces: `client/`, `server/`, `shared/`
- **Frontend**: React 19 + Vite + TypeScript + Tailwind CSS + shadcn/ui
- **Backend**: Fastify + TypeScript + Prisma ORM + PostgreSQL
- **Auth**: Clerk (works without key in dev mode with mock `dev_user`)
- **AI**: Claude API (Anthropic) for receipt scanning via vision
- **Deployment**: Vercel (client) + Railway (server + DB)

## Commands
- `npm run dev` — Start both client (port 5173) and server (port 3001)
- `npm run dev:client` — Client only
- `npm run dev:server` — Server only
- `npm run db:push` — Push Prisma schema to database
- `npm run db:migrate` — Run Prisma migrations
- `npm run db:seed` — Seed default categories and tax categories
- `npm run db:studio` — Open Prisma Studio

## Key Patterns
- **Server state**: TanStack React Query with custom hooks in `client/src/hooks/`
- **UI state**: Zustand store in `client/src/stores/uiStore.ts`
- **API client**: Axios with Clerk token interceptor in `client/src/services/api.ts`
- **Routing**: React Router v7 with Layout component using `<Outlet />`; Clerk SignIn uses `routing="hash"` to avoid URL conflicts with BrowserRouter
- **Layout**: `Header` (logo, Reports icon, Settings icon, user avatar) + content area + `MobileNav` (mobile-only bottom nav: Home, Scan, Reports, Settings). No persistent sidebar — `Sidebar.tsx` exists but is not used in Layout.
- **Components**: shadcn/ui components in `client/src/components/ui/`
- **Server routes**: Fastify plugins registered in `server/src/app.ts`
- **Auth**: Fastify plugin verifies Clerk JWT, falls back to dev user if no key
- **Analytics**: Vercel Analytics (`@vercel/analytics`) in client for page views/visitors (see Vercel project → Analytics)
- **Stats**: `GET /api/v1/stats` (auth required) returns `{ users, projects, expenses }` for tracking

## Database
- PostgreSQL via Prisma ORM
- Schema: `server/prisma/schema.prisma`
- Models: User, UserSettings, Project, Expense, Receipt, Deposit, Category, TaxCategory, Tag, ExpenseTag, RecurringRule, MileageEntry, Budget, SavedLocation
- **Expense–Receipt**: One expense can have many receipts (`Expense.receipts`, `Receipt.expenseId`); `Expense.receiptId` is optional “primary” receipt for backward compatibility
- All records scoped by `userId` for multi-tenancy
- Production DB: Railway PostgreSQL (`Postgres-hQ0m`)
- To push schema changes to production: `cd server && DATABASE_URL="<url>" npm run db:push`
- **Server build**: `npm run build` in server runs `prisma generate && tsc` so the Prisma client is always in sync
- **Startup**: DB connection runs in the background; server listens immediately. If DB is unavailable, API routes return 503; `GET /api/v1/health` returns `database: "connected" | "disconnected"`

## Deployment
- **Client**: Vercel — project `deducto` (renamed from `client`)
  - Production URL: `https://deductoapp.com` (custom domain) or `https://deducto-paul-landers-projects.vercel.app`
  - **Vercel Analytics**: Enabled via `@vercel/analytics` and `<Analytics />` in the app; view visitors and page views in Vercel project → Analytics
  - GitHub repo: `https://github.com/pjmlanders/Deducto.git` (auto-deploys on push to master)
  - Env vars: `VITE_API_URL`, `VITE_CLERK_PUBLISHABLE_KEY`
  - Root `vercel.json` handles monorepo build (`npm run build --workspace=client`, `--include=dev`)
- **Server**: Railway — `https://deducto-server-production.up.railway.app`
  - Service: `deducto-server` in project `Deducto`
  - Env vars set: `DATABASE_URL`, `NODE_ENV=production`, `ANTHROPIC_API_KEY`, `CLERK_SECRET_KEY`
  - Deploy: `cd server && railway up --service deducto-server`
- **Auth**: Clerk — add allowed domains in Clerk Dashboard → Configure → Domains
  - `https://deductoapp.com`, `https://www.deductoapp.com`, `https://deducto-paul-landers-projects.vercel.app`
- **Note**: Vercel has "Vercel Authentication" protection enabled — disable in Project Settings → Deployment Protection for public access
  - **Custom domain (e.g. deducto.com)**: If testers see "server can't be found" on the custom domain while the Vercel URL works, see [docs/CUSTOM-DOMAIN.md](docs/CUSTOM-DOMAIN.md) for step-by-step DNS and Clerk setup.

## AI Receipt Processing
- Claude Sonnet for vision-based receipt parsing
- Extracts: vendor, amount, date, line items, tax, tip, payment method
- Auto-categorizes using user's category list
- Duplicate detection via SHA-256 fingerprint

## Build Log

### Phase 1: Foundation (Feb 18, 2026)
- Monorepo structure with npm workspaces (`client/`, `server/`, `shared/`)
- Fastify server with Prisma ORM, PostgreSQL schema, migrations, seed data
- Clerk auth (client + server) with dev mode fallback (mock `dev_user`)
- Vite + React + Tailwind + shadcn/ui setup
- Layout component (sidebar + mobile bottom nav)
- Project CRUD (API + UI)
- Expense CRUD with manual entry, pagination, filtering, search
- Deposit CRUD with project association
- Receipt upload (multipart) + camera capture (`getUserMedia`)
- Claude AI vision integration for receipt processing (PDF + image)
- Receipt review/edit/accept flow with confidence scores + duplicate detection

### Phase 2–5: Features (Feb 18–19, 2026)
- **Dashboard Charts**: Recharts pie chart (category breakdown), bar chart (daily spending), line chart (12-month trend)
- **CSV Export**: Server endpoint with csv-stringify, client download button
- **PDF Export**: Server endpoint with pdfkit, formatted A4 reports
- **Tag System**: Tag CRUD in settings, tag selection on expenses, tag filter on expense list
- **Reimbursement Workflow**: Interactive status dropdown, amount/date fields, status filter
- **Bulk Operations**: Multi-select checkboxes, bulk delete, bulk categorize toolbar
- **Mileage Tracking**: Full CRUD routes, year-based IRS rates (2024: $0.67, 2025: $0.70, 2026: $0.725), summary cards, trip form/list
- **Budget Tracking**: CRUD routes, budget status (actual vs budget), progress bars on dashboard (green/yellow/red)
- **PDF Viewer**: Clean iframe-based PDF preview on receipt review (no toolbar/sidebar)
- **Receipt Management**: Dedicated receipt review page, batch drag-and-drop upload, receipt deletion
- **Default Categories**: 10 seeded categories (Renovations, Furnishings, Maintenance, etc.)
- **Database Inspector**: Prisma Studio available via `cd server && npx prisma studio`

### Phase 6: Project-Centric Workflow (Feb 22, 2026)
- **Rebrand**: Renamed from "Expense Tracker" to "Deducto" (app name, PWA manifest, package names, UI)
- **Dashboard project tiles**: Responsive grid above analytics — colored border, expense/deposit/net totals per project
- **ProjectDetail entry cards**: 3-card grid (Scan Receipt, Add Expense, Log Mileage) replacing old header buttons
- **Mileage route calculator**: "Calculate Route" toggle uses Nominatim geocoding + OSRM routing (no API key required) to auto-fill distance from start/end addresses
- **Mileage project field**: Project selector (required) on mileage form; project badge shown on trip list entries
- **Saved Locations**: `SavedLocation` model + `/api/v1/saved-locations` CRUD; manage in Settings; quick-fill chips in mileage form
- **Capital Expense flag**: `isCapitalExpense` boolean on Expense model — checkbox in expense form, badge in expense detail
- **Navigation**: Sidebar removed from Layout. Desktop nav is Header icons (Reports, Settings) only; content-level links drive deeper navigation. Mobile uses bottom `MobileNav` (Home, Scan, Reports, Settings). `Sidebar.tsx` is dead code.
- **Logo**: Deducto logo (emerald “D” + minus) as favicon, sidebar icon, and dashboard hero; PWA icons in `client/public/icons/`
- **Deployment**: Vercel (client) + Railway (server + PostgreSQL) — both live in production

### Phase 7: Receipts, Resilience, Analytics (Feb 2026)
- **Multiple receipts per expense**: One expense can have many receipts (`Receipt.expenseId`, `Expense.receipts`). Batch upload offers “One expense per receipt” (review list) or “One expense for all” (`POST /api/v1/receipts/accept-batch`). Attach receipt to existing expense: `POST /api/v1/expenses/:id/receipts` with `{ receiptId }`.
- **Server startup**: DB connection deferred so the server listens even if PostgreSQL is slow or down; non-health API routes wait for DB (with timeout) or return 503. `GET /api/v1/health` returns `database: "connected" | "disconnected"`.
- **Build**: Server build runs `prisma generate && tsc` so the Prisma client is always generated at deploy time.
- **Stats API**: `GET /api/v1/stats` (auth required) returns `{ users, projects, expenses }` for dashboards or monitoring.
- **Vercel Analytics**: `@vercel/analytics` and `<Analytics />` in the client; view in Vercel project → Analytics.
- **X-Frame-Options**: Set to `DENY` for the app; skipped for receipt file and preview URLs so PDFs can load in iframes on the receipt review page.

### Phase 8: UI Polish, Mileage Enhancements, Receipt Workflow (Mar 2026)
- **Receipt workflow overhaul**: Replaced dedicated receipt review page with inline flag + modal workflow. Scanned receipts auto-create draft expenses and flag incomplete ones for user review.
- **Mileage flags**: `taxDeductible` (default true) and `reimbursable` (default false) boolean fields added to `MileageEntry`; shown in mileage form and trip list.
- **Address autocomplete**: `AddressAutocomplete` component (`client/src/components/ui/address-autocomplete.tsx`) used in mileage form start/end fields and saved locations — powered by Nominatim, no API key required.
- **Route map**: After calculating a mileage route, an interactive map (`RouteMap.tsx`) is displayed inline using Leaflet.
- **Clerk hash routing**: `SignIn` component uses `routing="hash"` in `main.tsx` to prevent sign-up navigation from conflicting with `BrowserRouter`.
- **CORS**: `www.deductoapp.com` added to allowed origins on the server alongside `deductoapp.com`.
- **Legal pages**: `/privacy`, `/terms`, `/security` routes added (`client/src/components/legal/`).
- **Layout additions**: `ErrorBoundary`, `ApiStatusBanner` (shows when server is unreachable), and `AppFooter` added to the Layout wrapper.
- **Security hardening**: Various server-side security improvements (input validation, rate limiting, header hardening).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pjmlanders)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pjmlanders)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
