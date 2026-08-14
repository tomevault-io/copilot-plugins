## ipsos-finance-payment

> Internal tool for IPSOS Vietnam to manage field research projects and incentive payments to survey respondents. Key modules:

# IPSOS Finance Payment — Project Rules

## Project Overview

Internal tool for IPSOS Vietnam to manage field research projects and incentive payments to survey respondents. Key modules:
- **Project Management** — create/manage projects, assign users, track status
- **Transactions** — send gifts via Vinnet (telco top-up) and GotIt (e-voucher)
- **Quotation** — versioned cost quotation per project with approval workflow
- **MiniCATI** — phone survey respondent batches
- **TechcombankPanel** — panel analytics dashboard
- **TradeUnion** — recipient list email campaigns
- **Custom Voucher** — custom voucher assignment

## Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 + TypeScript, MUI v5, React Router v6, Axios, i18next |
| Backend | Laravel 11 (PHP 8.2+), Sanctum auth, Maatwebsite Excel |
| Database | MySQL (production), SQLite (local dev) |
| Infrastructure | Docker Compose (frontend + backend + db) |

## Repository Layout

```
IPSOS-FINANCE-PAYMENT/
├── backend/          # Laravel 11 API
│   ├── app/
│   │   ├── Http/Controllers/   # One controller per resource
│   │   ├── Http/Requests/      # Form request validation
│   │   ├── Http/Middleware/    # EnsureUserHasRole, CatiAuthMiddleware
│   │   ├── Models/
│   │   ├── Exports/            # Maatwebsite Excel exports
│   │   └── Constants/
│   ├── routes/api.php          # All REST routes
│   └── database/
├── frontend/
│   └── src/
│       ├── config/
│       │   ├── ApiConfig.ts        # All API endpoint URLs (single source of truth)
│       │   ├── RoleConfig.ts       # RBAC visibility config
│       │   ├── ProjectFieldsConfig.ts
│       │   └── AccountFieldsConfig.ts
│       ├── hook/               # Custom hooks: data fetching + state
│       ├── components/         # Shared UI components
│       │   ├── Dialogs/        # GenericDialog, ConfirmDialog, RejectDialog
│       │   ├── Table/          # ReusableTable
│       │   ├── Modals/
│       │   └── Sidebar/
│       ├── pages/              # Route-level pages
│       ├── Layouts/            # DefaultLayout, ProjectLayout, TechcombankLayout
│       └── routes/
└── database/                   # SQL schema, migrations, backups
```

## Development Setup

```bash
# Backend
cd backend && cp .env.example .env
php artisan key:generate
php artisan migrate
php artisan serve          # runs on :8000

# Frontend
cd frontend
npm install
npm start                  # runs on :3000

# Docker (full stack)
docker-compose up
```

## Frontend Conventions

### API Calls
- All endpoints live in `frontend/src/config/ApiConfig.ts`. Add new endpoints there; never hardcode URLs in components or hooks.
- Parameterised URLs use string `.replace('{paramName}', value.toString())` at call site.
- Auth token is stored in `localStorage` as `authToken` and sent as `Authorization: Bearer <token>`.

### Custom Hooks Pattern
Every resource has a dedicated hook in `frontend/src/hook/use<Resource>.ts`. The hook:
- Manages its own `ActionState` (`type | loading | error | message`)
- Exposes `fetchX`, `addX`, `updateX`, `deleteX` functions via `useCallback`
- Triggers `useEffect` on filter/pagination state changes

```ts
// ActionState shape — keep this consistent across all hooks
const [actionState, setActionState] = useState<ActionState>({
  type: 'idle',
  loading: false,
  error: false,
  message: ""
});
```

### Role-Based UI (RBAC)
- Visibility is configured in `frontend/src/config/RoleConfig.ts` via `BaseVisibility` + per-role `override` objects.
- Use `buildVisibility(role)` to get a merged visibility object; never check roles inline with string comparisons in JSX.
- Backend roles: `Admin`, `Finance`, `Scripter`, `Researcher`, `Field Manager`.

### Component Rules
- Use **MUI v5** components (`@mui/material`). Do not introduce a second UI library.
- Shared dialogs: `GenericDialog` for custom content, `ConfirmDialog` for yes/no, `RejectDialog` for rejection with reason.
- Table: use `ReusableTable` — pass columns config and data. Do not write one-off `<Table>` implementations.
- Icons: prefer `@mui/icons-material` or `@phosphor-icons/react`.

### State Management
- No Redux. Local state + custom hooks only.
- Server state uses plain Axios inside hooks (not React Query), following the existing `ActionState` pattern.

## Backend Conventions

### Controllers
- One controller per resource. Keep controllers thin — business logic in models or service classes.
- Return JSON with consistent shape: `{ status_code, message, data }`.

### Routes
- All routes in `backend/routes/api.php`.
- Protected routes go inside `Route::middleware(['auth:sanctum'])->group(...)`.
- Role protection: `.middleware('ensureUserHasRole:Admin,Finance')` (comma-separated, case-sensitive as defined in `EnsureUserHasRole`).

### Form Requests
- Validate all incoming data in a dedicated `FormRequest` class in `app/Http/Requests/`. Never validate in controllers.

### Naming
- Controller methods: `index`, `show`, `store`, `update`, `destroy` (Laravel resource convention).
- Non-resource actions use descriptive names: `updateStatus`, `assignUsers`, `bulkImport`.

## Security Rules

- Never commit `.env` files. Only `.env.example` is tracked.
- The `private_key.pem` file (deleted from git history) must never be re-added.
- Auth token in `localStorage` is acceptable for this internal tool — do not change storage mechanism without explicit discussion.
- All admin-only routes must have `ensureUserHasRole:Admin` middleware on the backend. Frontend visibility gates alone are not sufficient.
- Input from external providers (Vinnet, GotIt API responses) must be validated before storing.

## Working with External Payment Providers

### Vinnet (telco top-up)
- Transactions use RSA-signed payloads. Crypto logic lives in `frontend/src/utils/CryptoCustom.ts`.
- Merchant key can be rotated via the Finance panel.

### GotIt (e-voucher)
- Account balance tracked via deposit records in `AccountDeposit`.
- Each transaction debits the balance; check balance before sending.

## Git & Branching

- Branch from `main`. Branch naming: `feat/<short-description>` or `fix/<short-description>`.
- Commit messages: `feat:`, `fix:`, `refactor:` prefixes.
- Do not force-push to `main`.
- Do not commit build artifacts (`frontend/build/`).

## Things to Avoid

- Do not add new npm packages without checking if MUI or an existing dependency already covers the need.
- Do not introduce `any` types in new TypeScript code. Extend existing config types or add new ones in `*Config.ts` files.
- Do not add comments that describe *what* code does. Only add comments when the *why* is non-obvious.
- Do not create additional CSS files — use MUI's `sx` prop or extend `frontend/src/assets/css/` files that already exist.
- Do not skip Laravel form request validation for new endpoints.

---
> Source: [longlh123/IPSOS-FINANCE-PAYMENT](https://github.com/longlh123/IPSOS-FINANCE-PAYMENT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
