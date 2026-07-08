## pengi-med-saas

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Full stack (Docker)
```bash
just dev                          # Entire stack via Docker Compose (recommended)
```

### Backend (`apps/api` — Go)
```bash
cd apps/api
go run cmd/main.go                # Run API (port 8080)
go build ./...                    # Build
go vet ./...                      # Static analysis
```

### Frontend (`apps/web` — React)
```bash
cd apps/web
pnpm run dev                      # Dev server (port 5173)
pnpm run build                    # Production build (tsc + vite)
pnpm run lint                     # ESLint
```

### Code quality (root — applies to all TS/JS)
```bash
just check                         # Check formatting + lint (primary command)
just lint                          # Auto-format all TS/JS
```

### Setup & utilities
```bash
just setup                         # Configure git hooks (run once after cloning)
```

### Infrastructure dependencies (for local backend dev without Docker)
```bash
docker compose -f docker-compose.dev.yaml up pengi-db-dev pengi-rabbitmq-dev gotenberg sri-xml-signer -d
```

---

## Documentation & Skills

**Complete implementation guides** are in `docs/skills/`:
- [`api-backend-complete-guide.md`](docs/skills/api-backend-complete-guide.md) — Backend architecture + patterns + how-to
- [`web-frontend-complete-guide.md`](docs/skills/web-frontend-complete-guide.md) — Frontend architecture + patterns + how-to
- [`form-creation-standard.md`](docs/skills/form-creation-standard.md) — Standard for creating forms (Zod + Form components)

When implementing a feature:
1. Read the complete guide for your stack (API or Web)
2. Follow the "Paso a paso" (step-by-step) section
3. Use the checklist at the end

---

## Architecture

### Monorepo structure
```
apps/
  api/              # Go backend
  web/              # React SaaS frontend
  backoffice/       # React admin panel
  landing/          # Astro landing page
  sri-xml-signer/   # Node.js SRI XML signing microservice
biome.json          # Formatter + linter for all TS/JS apps
Justfile            # Dev shortcuts
```

---

## Backend (`apps/api`)

**Module name:** `pengi-med-saas`
**Stack:** Gin + GORM + Zap + RabbitMQ + PostgreSQL

### Request/Response — the envelope pattern

Every handler **must** return `envelope.Response`, never write directly to `gin.Context`. Routes are wrapped with `envelope.Handle()`, which translates the i18n key in `Message` and serializes to JSON.

```go
// Handler definition
func (h *InvoiceHandler) GetAllInvoices(c *gin.Context) envelope.Response {
    tenantScope := tenant_middleware.TenantScope(c)
    var invoices []billing_models.Invoice
    if err := h.db.Scopes(tenantScope).Find(&invoices).Error; err != nil {
        h.logger.Error("failed to fetch invoices", zap.Error(err))
        return envelope.ErrorResponse(http.StatusInternalServerError, "...", core_errors.ErrInternal)
    }
    return envelope.SuccessResponse(invoices, "billing.invoices.fetch.success")
}

// Route registration
billingGroup.GET("/invoices", envelope.Handle(invoiceHandler.GetAllInvoices))
```

The second argument to `SuccessResponse`/`ErrorResponse` is always an **i18n key** (never a hardcoded string). Keys live in `apps/api/i18n/messages/messages_es.json` and `messages_en.json`.

### Handler struct pattern

```go
type InvoiceHandler struct {
    db     *gorm.DB
    logger *zap.Logger
}

func NewInvoiceHandler(db *gorm.DB, logger *zap.Logger) *InvoiceHandler {
    return &InvoiceHandler{db: db, logger: logger}
}
```

Handlers are instantiated in `apps/api/routes/` and injected with `db` + `logger.Log`.

### Multi-tenancy — CRITICAL

Every DB query that touches tenant data **must** apply the GORM scope:

```go
tenantScope := tenant_middleware.TenantScope(c)
h.db.Scopes(tenantScope).Find(&records)
```

**Never query without the scope.** The scope is populated by `TenantMiddleware(db)`, which reads the `X-Tenant-Slug` header from the request context.

Every model has `TenantID uint`. When creating records, always set it:

```go
tenantID, _ := c.Get("tenant_id")
item := &models.Item{TenantID: tenantID.(uint), ...}
```

### Error codes

Centralized in `apps/api/core/errors/codes.go`. Format: `E-{DOMAIN}-{N}`. Always use existing codes or add new ones there — never pass raw strings as error codes.

### Feature structure

```
features/[domain]/
  handlers/     # One file per resource (e.g. invoice-handler.go)
  models/       # GORM models
  dto/          # Request/response DTOs
  workers/      # Background consumers (billing only)
  middleware/   # Feature-specific middleware (tenants, users)
```

### i18n Messages

All user-facing strings are stored in `apps/api/i18n/messages/`:
- `messages_es.json` — Spanish
- `messages_en.json` — English

When creating endpoints or features:
1. Use i18n keys in `SuccessResponse`/`ErrorResponse`
2. Add new keys to **both** JSON files
3. Keys follow pattern: `{domain}.{resource}.{action}` (e.g., `billing.invoice.create.success`)

### Migrations

`apps/api/migrations/migrate.go` runs on startup:
1. GORM `AutoMigrate` for all models
2. Code migrations in `migrations/code-migrations/` (keyed by date)
3. Seeds i18n messages from JSON files into DB

To add a migration: create a file in `migrations/code-migrations/{year}/`, register it in `GlobalDBMap`.

### Async invoice processing

`POST /billing/invoices/:id/sri/process` publishes a message to RabbitMQ queue `invoice_tasks`. The worker in `features/billing/workers/invoice-worker.go` consumes it, calls the `sri-xml-signer` microservice, and updates the invoice status through these stages: `pending → processing → signed → validated → authorized`.

---

## Frontend (`apps/web`)

**Stack:** React 19 + Vite + TypeScript + TailwindCSS v4 + shadcn/ui + Zustand

### API service layer

Never use axios directly in components. Always go through a service file:

```typescript
// Pick the right axios instance:
//   api            → auth routes (no tenant header)
//   apiWithTenant  → tenant-scoped routes
//   noAuthApi      → public routes (no auth)

const billingService = createHttpService(apiWithTenant);

export const createInvoice = async (payload: CreateInvoicePayload) =>
    billingService.post<Invoice>("/billing/invoices", payload, {
        notifySuccess: true,   // service shows toast on success
        notifyError: true,     // service shows toast on error
    });
```

### Toast pattern

Toasts are owned by the **service layer** via `notifySuccess`/`notifyError` flags. Components only handle navigation and state updates after checking `res.success`:

```typescript
const res = await createInvoice(payload);
if (res.success) {
    navigate("/billing");
}
// No manual successToast/errorToast here — service handles it
```

Exception: pure UI validation toasts (e.g. "no patient selected") that are not the result of an API call stay in the component.

### Types from DB

All interfaces that mirror backend models extend `BaseModel` and use `snake_case` keys:

```typescript
export interface Invoice extends BaseModel {   // BaseModel has ID, CreatedAt, UpdatedAt, DeletedAt
    tenant_id: number;
    patient_id: number;
    sequential: string;
    status: string;
    total: number;
    items: InvoiceItem[];
}
```

### i18n

```typescript
const { textGet } = useText();   // hook from @/hooks/use-text (never use `t` or `useTranslation`)
const label = textGet("billing.invoice.title");
// Missing keys render as *billing.invoice.title* — no fallback needed
```

Never hardcode user-visible strings. Every label, placeholder, and message must be an i18n key. Add new keys to **both** `apps/api/i18n/messages/messages_es.json` and `messages_en.json`.

Keys are sourced from backend JSON and seeded into the database on startup.

### State management

**Zustand only** — no Redux, Context, or Recoil. Stores live in `src/store/`.

Create a store only if state is **shared across multiple components**. For single-component state, use `useState`.

```typescript
interface ItemStore {
  items: Item[];
  selectedItem?: Item;
  setItems: (items: Item[]) => void;
  setSelectedItem: (item: Item | undefined) => void;
}

export const useItemStore = create<ItemStore>((set) => ({
  items: [],
  selectedItem: undefined,
  setItems: (items) => set({ items }),
  setSelectedItem: (item) => set({ selectedItem: item }),
}));
```

### Routing & permissions

Routes are defined in `src/routes/routes.tsx` and wrapped with `<CheckPermission permissions={[...]} />`. Permission constants are in `src/lib/constants.ts` under `PERMISSIONS`.

Navigation items are managed in `src/config/nav-config.ts`. Each item can be filtered by:
- `feature` — Feature flag from backend (disabled items hide automatically)
- `permission` — RBAC permission requirement

---

## Environment variables

Backend template: `apps/api/.env.example`. Key vars:
- `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`
- `AUTH_KEY` — JWT signing secret
- `SRI_SIGNER_SERVICE_URL` — Points to `sri-xml-signer` service
- `SRI_ENV` — `test` or `prod` (Ecuador SRI environment)
- `RABBITMQ_USER`, `RABBITMQ_PASSWORD`
- `GIN_MODE` — `development` or `release`

Frontend env: Vite reads `VITE_API_URL` (defaults to `http://localhost:8000/api/v1`).

---

## Important Patterns to Remember

### 1. Never Query Without TenantScope (Backend)
```go
// ❌ WRONG
h.db.Find(&items)

// ✅ CORRECT
h.db.Scopes(tenant_middleware.TenantScope(c)).Find(&items)
```

### 2. Never Call API Directly (Frontend)
```typescript
// ❌ WRONG
const res = await apiWithTenant.get("/items");

// ✅ CORRECT
const res = await itemService.getItems();  // Service layer handles it
```

### 3. All User-Facing Strings Must Be i18n Keys
```typescript
// ❌ WRONG
<h1>Items</h1>

// ✅ CORRECT
<h1>{textGet("item.title")}</h1>
```

### 4. Toasts Are Service Responsibility (Frontend)
```typescript
// ❌ WRONG
const res = await createItem(data);
if (res.success) showSuccessToast("Item created");

// ✅ CORRECT
const res = await createItem(data);  // Service shows toast automatically
if (res.success) navigate("/items");
```

---
> Source: [pengistudio/pengi-med-saas](https://github.com/pengistudio/pengi-med-saas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
