## budgeting2

> Personal finance/budgeting application with a Rails API backend and React frontend. Connects to Plaid for bank account syncing and transaction imports. Supports multiple users per account, transaction categorization, merchant grouping, tagging, and financial dashboards.

# Budgeting App

Personal finance/budgeting application with a Rails API backend and React frontend. Connects to Plaid for bank account syncing and transaction imports. Supports multiple users per account, transaction categorization, merchant grouping, tagging, and financial dashboards.

## Tech Stack

### Backend
- **Ruby 3.4.2 / Rails 8.0 (API mode)**
- **PostgreSQL** (db: `budgeting2_development`, user/pass: `griffin/griffin`)
- **Plaid API** for bank integration
- **JWT authentication** via `x-budgeting-token` header
- **JBuilder** for JSON response templates (views in `app/views/`)
- Key gems: `paranoia`/`acts_as_paranoid` (soft deletes), `pagy` (pagination), `audited` (audit trails), `active_model_serializers`, `lograge`, `newrelic_rpm`

### Frontend (`ui/`)
- **React 19** with **TypeScript**
- **Vite** build tool
- **Mantine v8** component library (core, charts, dates, modals, notifications, spotlight)
- **Tailwind CSS v4**
- **Axios** for API calls
- **date-fns** and **dayjs** for date handling

### Mobile App (`apps/mobile-app/`)
- **Kotlin 2.1.0** / **Jetpack Compose** (Material 3)
- **Retrofit** + **kotlinx.serialization** for networking
- **Hilt** for dependency injection
- **Paging 3** for infinite scroll, **Vico** for charts
- See [Mobile App CLAUDE.md](apps/mobile-app/CLAUDE.md) for full details

## Project Structure

```
app/
  controllers/     # API controllers (all scoped under /api)
  models/          # ActiveRecord models
  services/        # Service objects (business logic)
  views/           # JBuilder JSON templates
config/
  routes.rb        # All routes under scope path: 'api'
db/
  schema.rb        # Current schema
  migrate/         # Migrations
spec/              # RSpec tests (factories, models, services)
lib/tasks/         # Rake tasks (sync, import, enrich)
ui/
  src/
    api/           # API client modules (axios-based)
    components/    # Reusable React components
    hooks/         # Custom React hooks
    pages/         # Page components
    utils/         # Utility functions
apps/
  mobile-app/      # Android app (Kotlin/Compose)
documentation/
  API.md           # REST API reference
```

## Key Models

- **Account** — top-level entity, has many users
- **User** — belongs to account, authenticated via JWT
- **PlaidAccount** / **PlaidAccessToken** — Plaid bank connections
- **PlaidTransaction** — imported transactions from Plaid
- **Merchant** — transaction merchants, can belong to a MerchantGroup
- **MerchantGroup** — groups related merchants together
- **MerchantTag** — hierarchical categories for merchants (has `is_leaf`, `target_budget`)
- **Tag** / **TagPlaidTransaction** — user-defined tags on transactions (many-to-many)
- **AccountBalance** — historical balance snapshots
- **Color** — color assignments for UI display

## Key Services

- `AuthService` — JWT token encoding/decoding, user lookup
- `PlaidService` — Plaid API integration (link tokens, syncing)
- `TransactionSearchService` — transaction querying/filtering
- `MerchantGroupingService` / `MerchantGroupSpendService` — merchant group logic
- `MerchantSpendService` / `MerchantTagService` — spend stats by merchant/category
- `ProfitAndLossService` / `MonthlySpendService` — financial reporting
- `SignupService` — account/user creation

## Running the App

```bash
# Start PostgreSQL
docker compose up -d db

# Start Rails API + UI dev server
foreman start          # or: bin/dev
# API on port 3000, UI via Vite dev server
```

## API Conventions

- **Base path:** `/api`
- **Auth:** `x-budgeting-token` header (JWT)
- **Pagination:** `currentPage` and `perPage` query params; responses include `page` object with `currentPage`, `totalPages`, `totalCount`
- **Errors:** 401 `{ "error": "Unauthorized" }`, 422 `{ "errors": ["..."] }`
- All controllers inherit from `ApplicationController` which runs `before_action :require_authenticated_user!`

## Testing

```bash
bundle exec rspec            # Run all specs
bundle exec rspec spec/models/    # Model specs only
bundle exec rspec spec/services/  # Service specs only
```

Uses RSpec with FactoryBot and Faker.

## Documentation

- [API Endpoints](documentation/API.md) — Complete REST API reference

When API endpoints are added, modified, or removed, update `documentation/API.md` to keep it in sync.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/griffinm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/griffinm)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
