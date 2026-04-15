## full-circle

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Full Circle is a multi-tenant web-based ERP system built with Elixir/Phoenix, covering accounting, billing, payroll (Malaysia-standard), inventory, and agricultural operations. It uses Phoenix LiveView exclusively for the UI (no REST/SPA pattern).

**Runtime versions:** Elixir 1.19.5, Erlang/OTP 28.3.1, Phoenix 1.8.3, Phoenix LiveView 1.1.x

## Git

Remote origin uses SSH: `git@github.com:tankwanghow/full_circle.git`

## Common Commands

```bash
# Setup
mix setup                        # deps.get, ecto.setup, assets.setup, assets.build

# Development
mix phx.server                   # Start dev server (HTTP :4000, HTTPS :4001)
iex -S mix phx.server            # Start with interactive shell

# Database
mix ecto.migrate                 # Run migrations
mix ecto.reset                   # Drop, create, migrate, seed

# Testing
mix test                         # Run all tests (auto-creates/migrates test DB)
mix test test/full_circle/accounting_test.exs          # Single test file
mix test test/full_circle/accounting_test.exs:42       # Single test at line

# Code quality
mix credo                        # Static analysis

# Assets
mix assets.build                 # Build CSS (Tailwind) and JS (esbuild)
mix assets.deploy                # Minified build + phx.digest for production
```

## Architecture

### Multi-Tenancy

Every entity belongs to a `Company`. Routes are scoped as `/companies/:company_id/*`. The `set_active_company` plug verifies user access via `CompanyUser` junction table and stores the active company in session. All queries are scoped through `Sys.user_company(company, user)` subquery joins to enforce data isolation.

### Two Ecto Repos

- **`FullCircle.Repo`** — Primary read/write repo
- **`FullCircle.QueryRepo`** — Read-only repo (separate DB user `full_circle_query`) for complex reporting queries

### Custom Schema Base

`FullCircle.Schema` (`lib/schema.ex`) sets `binary_id` as primary key type for all schemas. Use `use FullCircle.Schema` instead of `use Ecto.Schema`.

### Domain Contexts (`lib/full_circle/`)

| Context | Purpose |
|---------|---------|
| `Accounting` | GL accounts, transactions, tax codes, fixed assets, contacts |
| `Billing` | Sales invoices (`Invoice`) |
| `Billing` | Purchase invoices (`PurInvoice`) — same context |
| `ReceiveFund` | Cash receipts, received cheques |
| `BillPay` | Payments |
| `Cheque` | Deposits, returns, post-dated cheques |
| `DebCre` | Debit/credit notes |
| `HR` | Employees, salary types, pay slips, time attendance, holidays |
| `Product` | Goods and packaging |
| `Layer` | Agricultural: houses, flocks, harvests, weighing, movements |
| `EInvMetas` | E-invoice metadata (Malaysia LHDN integration) |
| `Reporting` | Report queries |
| `Sys` | Companies, users, logging |
| `UserAccounts` | Authentication (bcrypt, session tokens) |
| `Authorization` | Role-based access via `can?(user, :action, company)` |

### StdInterface Pattern (`lib/full_circle/std_interface.ex`)

Reusable CRUD operations used across most contexts: `get!`, `filter`, `create`, `save`, `delete`. Accepts schema class, company, and user — automatically handles authorization checks and audit logging. When adding a new entity, implement the `query/2` function in the schema's context.

### Authorization Roles

Defined in `lib/full_circle/authorization.ex`. Roles: `admin`, `manager`, `supervisor`, `cashier`, `clerk`, `auditor`, `punch_camera`, `guest`, `disable`. Authorization uses pattern-matched `can?/3` functions.

### LiveView Structure (`lib/full_circle_web/live/`)

Each feature follows a consistent folder pattern:
- `index.ex` — List view with search/pagination
- `form.ex` — Create/edit form
- `index_component.ex` — Table row component for index
- `detail_component.ex` — Nested detail line component (e.g., invoice lines)
- `print.ex` — Print-optimized view (uses `print_root` layout, HTML print stylesheets)

### Layouts

Five layouts in `lib/full_circle_web/components/layouts/`:
- `root.html.heex` — Main app shell
- `app.html.heex` — Authenticated app content
- `print_root.html.heex` — Print-optimized (no nav, print CSS)
- `punch.html.heex` — Time punch kiosk mode
- `recon.html.heex` — Face recognition attendance

### Frontend Assets

- **CSS**: Tailwind CSS 3.4, configured in `assets/tailwind.config.js`
- **JS**: esbuild with ESM format and code splitting. Entry points in `assets/js/`:
  - `app.js` — Main app (LiveView hooks, IndexedDB caching via `indexdb.js`)
  - `tri_autocomplete.js` — Custom autocomplete component
  - `take_photo_human.js` / `face_id.js` — Face recognition (uses Human.js library in `assets/vendor/human-main/`)
  - `qr_attend.js` — QR code attendance scanning

### I18n

Supports English (`en`) and Chinese (`zh`) via Gettext. Locale stored in session, set by `set_locale` plug.

### Key Conventions

- Document numbers are gapless per company (see `create_gapless_doc_number` migration)
- Transactions use a double-entry pattern with `Transaction` and `TransactionMatcher` tables
- Database triggers handle transaction posting automatically (see `create_transaction_trigger` migration)
- Print views support `pre_print` parameter (true = data only for pre-printed forms, false = full letterhead)
- Company deletion cascades via database triggers (see `create_triggers_when_delete_company` migration)
- PostgreSQL `pg_trgm` extension used for fuzzy search (see `create_fuzzy_search` migration)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tankwanghow)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tankwanghow)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
