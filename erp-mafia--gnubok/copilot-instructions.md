## gnubok

> gnubok is a Swedish-focused accounting SaaS for sole traders (enskild firma) and limited companies (aktiebolag). It implements double-entry bookkeeping compliant with Swedish accounting law (Bokforingslagen), including VAT handling, tax reporting, and 7-year document retention. Multi-tenant: each user can own or be a member of multiple companies, optionally grouped into teams (for consultants).

# CLAUDE.md — gnubok

## Project Overview

gnubok is a Swedish-focused accounting SaaS for sole traders (enskild firma) and limited companies (aktiebolag). It implements double-entry bookkeeping compliant with Swedish accounting law (Bokforingslagen), including VAT handling, tax reporting, and 7-year document retention. Multi-tenant: each user can own or be a member of multiple companies, optionally grouped into teams (for consultants).

**Tech stack**: Next.js 16.1.5 (App Router), React 19.2.3, TypeScript 5 (strict), Zod 4, Supabase (PostgreSQL + RLS + email/password + TOTP MFA auth), Tailwind CSS 4 + shadcn/ui, Vercel hosting, Docker (self-hosted).

**Integrations**: Enable Banking (PSD2), TIC Identity (company lookup), Anthropic SDK, AWS Bedrock (`@aws-sdk/client-bedrock-runtime` for inbox smart-match), OpenAI (embeddings), Resend (email), Sentry (error tracking), Svix (webhooks), web-push (notifications), Upstash Redis + Ratelimit, Google Drive (cloud backup via OAuth), JSZip (archive export), sharp (image processing), Framer Motion (animations), Recharts (charts), PDF.js (`pdfjs-dist`), `@react-pdf/renderer` (invoice PDFs), xlsx, fuse.js (fuzzy search), ics (iCal feeds).

**Path alias**: `@/*` maps to the project root. **Language**: All code, comments, and commit messages in English. **License**: AGPL-3.0-or-later.

---

## Commands

```bash
npm run dev              # Start dev server (runs setup:extensions first)
npm run build            # Production build (runs setup:extensions first)
npm run lint             # ESLint
npm test                 # Run all Vitest tests
npx vitest run <dir>     # Run tests in a specific directory
npm run setup:extensions # Regenerate extension registry from extensions.config.json
```

---

## Key Architectural Relationships

- **Multi-tenant model**: `companies` table owns all business data. `company_members` links users to companies with roles (owner/admin/member/viewer). `teams` group companies for consultants. Company context resolved via cookie (`gnubok-company-id`) in middleware (`lib/supabase/middleware.ts`).
- **All journal entry creation** routes through `lib/bookkeeping/engine.ts`. Lifecycle: `createDraftEntry()` → `commitEntry()` (atomic voucher assignment via `commit_journal_entry` DB RPC). Convenience: `createJournalEntry()` does both. Reversal via `reverseEntry()`. Correction via `correctEntry()` in `lib/core/bookkeeping/storno-service.ts`.
- **API routes** that emit events must call `ensureInitialized()` (from `lib/init.ts`) at module level. This loads extensions, wires event handlers, and registers the supplier invoice handler + event log handler.
- **Event bus** (`lib/events/bus.ts`) is a module-level singleton. Handlers run via `Promise.allSettled` — failing handlers never crash the emitter. 36 event types defined in `lib/events/types.ts`. The event log handler persists actionable events to `event_log` table for external automation.
- **Supabase clients**: browser (`lib/supabase/client.ts`), server with cookies (`createClient()` from `server.ts`), service role (`createServiceClient()`), cookieless service role for API key auth (`createServiceClientNoCookies()` from `lib/auth/api-keys.ts`). Pagination helper: `fetchAllRows()` in `lib/supabase/fetch-all.ts`.
- **Extension system**: Opt-in via `extensions.config.json`. Core builds and runs with zero extensions. Currently enabled: `enable-banking`, `email`, `arcim-migration`, `tic`, `mcp-server`, `cloud-backup`.
- **Core reports** (in `lib/reports/`, not extensions): balance sheet, income statement, trial balance, general ledger, AR/supplier ledger, AR/supplier reconciliation, VAT declaration, journal register, monthly breakdown, continuity check, opening balances, KPI (+ definitions), NE-bilaga, INK2 declaration, SIE export, full archive export, salary journal, vacation liability, avgifter basis (employer contributions).
- **Types**: All shared types in `types/index.ts` (~2,570 lines, single source of truth). Import via `import type { T } from '@/types'`. Event types live in `lib/events/types.ts`. Extension types in `lib/extensions/types.ts`.
- **Error messages**: `lib/errors/get-error-message.ts` maps technical errors to Swedish user messages (Zod → Postgres → HTTP → context fallback).

---

## Multi-Tenant Architecture

### Data Model

- **companies**: Business unit (name, org_number, entity_type, created_by, team_id). All business data (journal entries, invoices, transactions, etc.) has a `company_id` column.
- **company_members**: Links users to companies (company_id, user_id, role, source='direct'|'team'). Roles: `owner`, `admin`, `member`, `viewer`.
- **teams**: Consultant grouping (name, created_by). A company can belong to one team. Team members auto-sync to company_members via DB triggers.
- **team_members**: Links users to teams (team_id, user_id, role='owner'|'admin'|'member').
- **user_preferences**: Stores `active_company_id` per user.

### Company Context Resolution

Middleware (`lib/supabase/middleware.ts`) resolves the active company on every request:
1. Check `gnubok-company-id` cookie
2. Fall back to `user_preferences.active_company_id`
3. Fall back to first company membership

RLS policies use `user_company_ids()` DB helper function to filter by companies the user has access to.

### Invitations

- **company_invitations**: Email-based invites with `gnubok_inv_` prefixed tokens (SHA-256 hashed, 7-day TTL).
- **team_invitations**: Same pattern for team invites.
- Token generation: `lib/auth/invite-tokens.ts`.

---

## Authentication

Supabase Auth with **email+password** (primary) and **magic link** (fallback). MFA via TOTP is supported.

MFA is enforced **application-side** (middleware + API routes), **not** in RLS policies. Controlled by two env vars:

- `NEXT_PUBLIC_SELF_HOSTED=true` → MFA never enforced (users can enable voluntarily)
- `NEXT_PUBLIC_REQUIRE_MFA=true` (hosted/Vercel) → middleware redirects to `/mfa/enroll` or `/mfa/verify` until AAL2

**API route auth** (`lib/auth/require-auth.ts`): `requireAuth()` returns `{ user, supabase, error }` discriminated union, enforces MFA on hosted.

**API keys** (`lib/auth/api-keys.ts`): SHA-256 hashed with `gnubok_sk_` prefix. Scoped permissions (`TOOL_SCOPE_MAP`). Rate limited at 100 RPM via atomic DB RPC (`validate_and_increment_api_key`).

**Cron auth** (`lib/auth/cron.ts`): `verifyCronSecret()` with constant-time comparison.

---

## Core Bookkeeping Engine

The engine (`lib/bookkeeping/engine.ts`) is the most critical system. All accounting flows route through it.

**Lifecycle**: `createDraftEntry()` → `commitEntry()` (atomic voucher assignment via `commit_journal_entry` DB RPC). Convenience: `createJournalEntry()` does both in one call. Reversal via `reverseEntry()` (storno). Correction via `correctEntry()` in `lib/core/bookkeeping/storno-service.ts`.

**Key engine files**:
- `transaction-entries.ts` — Journal entries from bank transactions
- `invoice-entries.ts` — Journal entries from customer invoices (`generatePerRateLines()` for mixed-rate)
- `supplier-invoice-entries.ts` — Journal entries from supplier invoices
- `vat-entries.ts` — VAT-related entries
- `currency-revaluation.ts` — Multi-currency revaluation
- `mapping-engine.ts` — Account mapping rules evaluation
- `booking-templates.ts` / `counterparty-templates.ts` — Reusable templates
- `propose-payment-lines.ts` / `propose-send-lines.ts` — AI-powered matching proposals
- `handlers/supplier-invoice-handler.ts` — Event handler creating registration entries on confirmation

**BAS data** (`bookkeeping/bas-data/`): Full BAS 2026 chart organized by class (1–8) + SRU mapping.

### Key BAS Accounts

`1510` Accounts receivable | `1930` Business bank account | `2013` Private withdrawals (EF) | `2440` Accounts payable | `2611`/`2621`/`2631` Output VAT 25%/12%/6% | `2641` Input VAT | `2645` Calculated input VAT (EU) | `2893` Shareholder loan (AB) | `3001`/`3002`/`3003` Revenue 25%/12%/6% | `3305`/`3308` Export/EU service revenue

### VAT Treatments

`standard_25`, `reduced_12`, `reduced_6`, `reverse_charge`, `export`, `exempt`

Invoice items support individual `vat_rate` values (mixed-rate invoices). Use `getAvailableVatRates(customerType, vatNumberValidated)` from `lib/invoices/vat-rules.ts`. VIES validation via `lib/vat/vies-client.ts`.

### VAT Declaration Rutor (SKV 4700)

The `VatDeclarationRutor` type maps to the Swedish tax authority's momsdeklaration form:

- **Ruta 05**: Momspliktig försäljning — total domestic taxable sales (all rates combined, from 3001+3002+3003)
- **Ruta 06/07**: Unused (momspliktiga uttag / vinstmarginalbeskattning), always 0
- **Ruta 10/11/12**: Utgående moms 25%/12%/6% — output VAT per rate (from 2611/2621/2631)
- **Ruta 39/40**: EU services / Export (from 3308/3305)
- **Ruta 48**: Ingående moms — input VAT (from 2641/2645)
- **Ruta 49**: Moms att betala/återfå = (ruta 10 + 11 + 12 + 30 + 31 + 32 + 60 + 61 + 62) - ruta 48

---

## Core Services (`lib/core/`)

- `bookkeeping/period-service.ts` — Fiscal period lifecycle management (open, close, lock)
- `bookkeeping/year-end-service.ts` — Year-end closing procedures
- `bookkeeping/storno-service.ts` — Reversal/correction entry generation
- `tax/tax-code-service.ts` — Tax code definitions and rates
- `audit/audit-service.ts` — Audit trail and compliance logging
- `documents/document-service.ts` — Document attachment lifecycle (WORM storage with version chains)

---

## Accounting Guard Rails

These rules exist for legal compliance, enforced by database triggers. **Never violate them.**

1. **Committed entries are immutable.** Once `status: 'posted'`, cannot be edited or deleted (DB trigger).
2. **Never delete posted entries.** Use `reverseEntry()` (storno) to cancel.
3. **Every entry must balance.** `sum(debits) === sum(credits)`, both `> 0`.
4. **Voucher numbers are sequential.** Assigned atomically via `commit_journal_entry` DB RPC. Never set manually.
5. **Voucher gap documentation.** BFNAR 2013:2 requires documented explanations for gaps (`voucher_gap_explanations` table, `detect_voucher_gaps` RPC).
6. **Period lock enforcement.** DB trigger blocks writes to closed/locked periods. Company-wide lock date enforced via `enforce_company_lock_date()` trigger.
7. **7-year document retention.** DB triggers prevent deletion of documents linked to posted entries.
8. **Storno, never edit.** Use `correctEntry()` from `lib/core/bookkeeping/storno-service.ts`.
9. **Use `Math.round(x * 100) / 100`** for monetary calculations. Never `toFixed()`.
10. **Always use engine functions.** Never insert directly into journal tables.
11. **Account numbers are strings.** `'1930'`, never `1930`.

---

## Extension System

Extensions are opt-in plugins in `extensions/general/<name>/`, controlled by `extensions.config.json`. Core builds and runs with zero extensions. `npm run setup:extensions` generates static imports in `lib/extensions/_generated/` (runs automatically via `predev`/`prebuild`). Extensions **cannot** use dynamic imports (Next.js bundling).

### Available Extensions (12)

| Extension | Purpose | Currently Enabled |
|-----------|---------|:-:|
| `enable-banking` | PSD2 bank sync via Enable Banking | Yes |
| `email` | Email delivery via Resend | Yes |
| `arcim-migration` | Legacy ARCIM system data migration | Yes |
| `tic` | TIC Identity company lookup (org number → name, VAT, address) | Yes |
| `mcp-server` | MCP server for Claude Desktop/Code | Yes |
| `cloud-backup` | Google Drive backup of SIE + receipts + processing history | Yes |
| `inbox-smart-match` | AWS Bedrock AI matching of inbox receipts to bank transactions | No |
| `invoice-inbox` | Email-based invoice document processing | No |
| `push-notifications` | Web push notifications for events | No |
| `calendar` | Payment calendar with iCal feed | No |
| `skatteverket` | Skatteverket VAT declaration submission | No |
| `example-logger` | Reference implementation — logs events to console (not registered by default) | No |

### Extension Architecture

**Registration** (`lib/extensions/registry.ts`): Singleton registry. `register()` wires event handlers to the bus. `get(id)`, `getAll()`, `getByCapability(key)`.

**Context** (`lib/extensions/context-factory.ts`): Every handler receives `ExtensionContext` with: `userId`, `companyId`, `extensionId`, `supabase`, `emit()`, `settings` (key-value in `extension_data` table), `storage` (Supabase Storage), `log` (prefixed logger), `services` (e.g., `ingestTransactions`).

**API routes**: Dispatched via catch-all at `app/api/extensions/ext/[...path]/route.ts`. URL: `/api/extensions/ext/{extensionId}/{routePath}`. Path params extracted as `_paramName` search params.

**Service provider patterns**:
- *Interface registration* (email): Core defines noop default in `lib/email/service.ts`, extension calls `registerEmailService()`, core uses `getEmailService()`.
- *Services record* (ai-categorization): Extension exposes via `services` property, core looks up via `extensionRegistry.get('id')?.services?.method(...)`.

**Creating extensions**: `npx tsx scripts/create-extension.ts --name my-ext --sector general --category operations --description "..."`, then add to `extensions.config.json`.

---

## MCP Server & API Keys

gnubok exposes its bookkeeping engine as an MCP (Model Context Protocol) server, letting users do bookkeeping through Claude Desktop, Claude Code, or any MCP-compatible client.

**MCP extension** (`extensions/general/mcp-server/`): 35 tools — transactions, categorization, customers, suppliers, invoices, supplier invoices, accounts, fiscal periods, trial balance, general ledger, balance sheet, income statement, AR/supplier ledger, reconciliation, VAT report, KPI report, receipt matching, invoice payments/sending, inbox items, employees + salary runs (list/get/create/calculate), salary journal, AGI generation, document upload. JSON-RPC 2.0 protocol implemented directly (no SDK dependency). Endpoint: `/api/extensions/ext/mcp-server/mcp`.

**API key infrastructure** (`lib/auth/api-keys.ts`, `api_keys` table): SHA-256 hashed keys with `gnubok_sk_` prefix. Scoped permissions mapped via `TOOL_SCOPE_MAP`. Rate limited at 100 RPM via atomic DB RPC (`validate_and_increment_api_key`). `createServiceClientNoCookies()` creates a Supabase service client without cookies for API key auth — all queries filter by `company_id` (defense in depth).

**OAuth 2.1** for Claude Desktop connectors:
- `.well-known/oauth-protected-resource` and `.well-known/oauth-authorization-server` — discovery endpoints (excluded from auth middleware)
- `/api/mcp-oauth/authorize` — consent page + auth code generation
- `/api/mcp-oauth/token` — PKCE verification + API key creation
- `/api/mcp-oauth/register` — dynamic client registration
- Stateless encrypted auth codes (AES-256-GCM via `lib/auth/oauth-codes.ts`)
- Single-use enforcement via `oauth_used_codes` table
- Redirect URI allowlist: `claude.ai/api/*`, `claude.com/api/*`, `localhost`

**npm package** (`packages/gnubok-mcp`): Published as `gnubok-mcp` on npm. Stdio-to-HTTP bridge for Claude Desktop. Users configure `npx gnubok-mcp` with their API key.

---

## API Route Pattern

```typescript
import { createClient } from '@/lib/supabase/server'
import { NextResponse } from 'next/server'
import { ensureInitialized } from '@/lib/init'
import { validateBody } from '@/lib/api/validate'
import { MySchema } from '@/lib/api/schemas'

ensureInitialized()  // Module-level — loads extensions for event emission

export async function POST(request: Request) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })

  const result = await validateBody(request, MySchema)
  if (!result.success) return result.response

  // Business logic... always filter by company_id (defense in depth alongside RLS)
  return NextResponse.json({ data: result })
}
```

- Dynamic route params: `{ params }: { params: Promise<{ id: string }> }` (Next.js 16)
- Response shapes: `{ data }` for success, `{ error }` for failures
- Zod schemas in `lib/api/schemas.ts` — 30+ schemas with shared primitives (uuid, isoDate, accountNumber, nonNegativeAmount)

---

## Key lib/ Directories

| Directory | Purpose |
|-----------|---------|
| `bookkeeping/` | Engine, entry generators, mapping, templates, BAS data, template library + embeddings |
| `core/` | Period service, year-end, storno, tax codes, audit, documents |
| `events/` | Event bus singleton, 36 event types, event log handler |
| `auth/` | API keys, require-auth, require-write, MFA, OAuth codes, invite tokens, cron auth, BankID |
| `supabase/` | Browser/server/service clients, middleware, fetch-all pagination |
| `api/` | Zod validation (`validateBody`/`validateQuery`), schemas |
| `reports/` | 20 report generators (financial statements, ledgers, tax, exports, salary journal, vacation liability, avgifter basis) |
| `invoices/` | Invoice/supplier matching, payment match log, reminders, VAT rules, PDF template |
| `transactions/` | Multi-source ingestion (`ingest.ts`), AI category suggestions |
| `import/` | SIE parser/import, bank file import, opening balance, account mapper |
| `documents/` | Document matcher, receipt matcher, batch matching |
| `extensions/` | Registry, loader, context factory, types, generated files |
| `email/` | Service interface (noop default), Resend provider, templates (invite, invoice, reminder, consent) |
| `company/` | Company context resolution, CRUD actions, fiscal period computation |
| `company-lookup/` | Shared types for org-number → company info lookups |
| `providers/` | Third-party accounting provider adapters (Fortnox, Bokio, Briox, BL/Björn Lundén, Visma) with OAuth, rate limiting, retry, consent resolution |
| `salary/` | Payroll calculation engine, tax tables, absence/benefits/traktamente, AGI, KU, PDF payslips, löneväxling, personnummer, payment, salary entries, transaction matcher |
| `processing-history/` | Processing-history append helper for audit/inbox timelines |
| `reconciliation/` | Bank statement reconciliation |
| `tax/` | Tax calculator, deadline config/generator, expense warnings, Swedish holidays |
| `vat/` | VIES client, EU countries, MOMS box mapping |
| `deadlines/` | Deadline status engine |
| `currency/` | Riksbanken exchange rates |
| `skatteverket/` | Tax authority data formatting |
| `bankgiro/` | Luhn checksum validation |
| `calendar/` | ICS generator, calendar utilities |
| `errors/` | Swedish error message mapping (Zod → Postgres → HTTP → fallback) |
| `hooks/` | React hooks (e.g., `use-unsaved-changes`, `use-can-write`) |
| `logger.ts` | Structured logger with module prefixes, env-aware filtering |
| `support.ts` | Server-side support recipient email (used by `/api/support/contact`) |
| `utils.ts` | `cn()`, `formatCurrency()`, `formatDate()`, `formatOrgNumber()` |

---

## App Routes

### Pages

| Route | Purpose |
|-------|---------|
| `/login`, `/register`, `/reset-password` | Auth pages |
| `/mfa/enroll`, `/mfa/verify` | MFA flow |
| `/onboarding` | Multi-step company setup wizard |
| `/companies/new` | Create new company |
| `/invite/[token]` | Accept team/company invite |
| `/` | Dashboard home |
| `/transactions` | Bank transaction list & categorization |
| `/invoices`, `/invoices/new`, `/invoices/[id]`, `/invoices/[id]/credit` | Customer invoicing |
| `/supplier-invoices`, `/supplier-invoices/new`, `/supplier-invoices/[id]` | Supplier invoices |
| `/customers`, `/customers/[id]` | Customer management |
| `/suppliers`, `/suppliers/[id]` | Supplier management |
| `/expenses`, `/expenses/new`, `/expenses/[id]` | Expense tracking |
| `/receipts`, `/receipts/scan` | Receipt management |
| `/bookkeeping`, `/bookkeeping/[id]`, `/bookkeeping/year-end` | Journal entries, chart of accounts, year-end |
| `/salary`, `/salary/employees`, `/salary/runs` | Payroll: employees, salary runs, AGI, KU |
| `/reports` | Financial reports |
| `/import` | SIE and bank file import |
| `/kpi` | KPI metrics + monthly trend chart |
| `/deadlines` | Tax & business deadlines |
| `/pending` | Pending operations queue |
| `/help` | In-app help/support page |
| `/extensions`, `/extensions/[sector]/[extension]` | Extension marketplace |
| `/e/[sector]/[slug]` | Extension workspace |
| `/settings/*` | Company, invoicing, bookkeeping, tax, team, banking, templates, salary, backup, account, API settings |
| `/dpa`, `/privacy` | Legal pages |
| `/invoice-action/[token]` | Public invoice payment link |
| `/sandbox` | Test environment |

### API Endpoints (key groups)

- `/api/bookkeeping/*` — Accounts, fiscal periods (close/lock/year-end/opening-balances/currency-revaluation), journal entries (CRUD/reverse/correct/chain), mapping rules, voucher gaps
- `/api/invoices/*` — CRUD, send, mark-sent/paid, convert, PDF, reminders cron
- `/api/supplier-invoices/*` — CRUD, approve, mark-paid, credit
- `/api/transactions/*` — Categorize, uncategorize, describe, book, match-invoice, match-supplier-invoice, batch operations, AI suggestions
- `/api/customers/*`, `/api/suppliers/*` — CRUD
- `/api/documents/*` — CRUD, versions, link, verify, match-sweep, verify cron
- `/api/reports/*` — 19 report endpoints (general-ledger, trial-balance, balance-sheet, income-statement, journal-register, ar-ledger, supplier-ledger, vat-declaration, sie-export, ink2, ne-bilaga, kpi, audit-trail, continuity-check, monthly-breakdown, full-archive, salary-journal, vacation-liability, avgifter-basis)
- `/api/salary/*` — Employees, payroll-config, tax-tables, KU, salary runs (CRUD + calculate)
- `/api/import/*` — Bank file (parse/execute), SIE (parse/execute/mappings/create-accounts)
- `/api/reconciliation/bank/*` — Link, unlink, run, status, unmatched-entries
- `/api/settings/*` — Company settings, API keys, logo upload, counterparty templates, booking templates
- `/api/company/*` — Current, members (list/CRUD/invite), `[id]`
- `/api/team/*` — Accept, invite, members
- `/api/deadlines/*`, `/api/tax-deadlines/*` — Deadline CRUD and crons
- `/api/pending-operations/*` — Queue, commit, reject
- `/api/events/*` — Event log and cleanup cron
- `/api/documents/*` — CRUD, counts, match-sweep, verify cron
- `/api/calendar/feed/[token]` — iCal subscription feed
- `/api/mcp-oauth/*` — Register, authorize, token
- `/api/support/contact` — Support contact form submission
- `/api/account/delete` — User account deletion
- `/api/audit-trail/*` — Audit trail queries
- `/api/log` — Client-side log ingestion
- `/api/health` — Health check
- `/api/vat/validate` — VIES VAT validation
- `/api/currency/rate` — Riksbanken exchange rate lookup
- `/api/sandbox/*` — Seed, cleanup cron
- `/api/extensions/ext/[...path]` — Dynamic extension API routes (plus top-level `/api/extensions/enable-banking/*`, `/api/extensions/cloud-backup/*`, `/api/extensions/push-notifications/*`)

---

## Testing

**Framework**: Vitest 4, `globals: true`, `environment: 'node'`. Tests colocated in `__tests__/` directories. Scope: business logic in `lib/` and API routes in `app/api/`. No component or E2E tests.

**Test helpers** (`tests/helpers.ts`): `createMockSupabase()` (chainable proxy), `createQueuedMockSupabase()` (sequential calls), `createMockRequest()`, `parseJsonResponse()`, `createMockRouteParams()`, and fixture factories: `makeTransaction()`, `makeJournalEntry()`, `makeJournalEntryLine()`, `makeInvoice()`, `makeInvoicePayment()`, `makeCustomer()`, `makeSupplier()`, `makeSupplierInvoice()`, `makeFiscalPeriod()`, `makeReceipt()`, `makeDocumentAttachment()`, `makeCompanySettings()`, `makeCompany()`, `makeCompanyMember()`, `makeInvoiceInboxItem()`, `makeTaxCode()`, `makeCategorizationTemplate()`, `makeSIEVoucher()`, `makeBankConnection()`.

**Patterns**: Always mock `@/lib/supabase/server`. Use `vi.clearAllMocks()` and `eventBus.clear()` in `beforeEach`. API route tests: mock `@/lib/init` and lib functions, test auth (401), validation (400), not found (404), errors (500), happy path.

### Testing database-level logic (pg-real)

Mocked Supabase clients cannot exercise Postgres triggers, RPCs, or RLS policies. A parallel Vitest project `pg-real` runs against a real Postgres instance in CI (GitHub Actions `supabase/postgres:15` service container, migrations replayed from `supabase/migrations/` before the suite runs). Locally: `npm run test:pg` against a DATABASE_URL pointing at any Postgres with the Supabase `auth` schema and migrations applied.

**File convention**: `*.pg.test.ts`. The `unit` project excludes this suffix; only `pg-real` picks it up.

**Helpers**: `tests/pg/setup.ts` exposes `getPool()` and `withUserContext(userId, fn)` (sets `ROLE authenticated` + `request.jwt.claims` for RLS tests). `tests/pg/fixtures.ts` has `seedCompany()`, `insertDraftJournalEntry()`, `insertBalancedLines()`, etc.

**When to add a pg-real test**: any PR that creates or modifies a trigger, RPC, RLS policy, or DEFERRABLE constraint must include or extend a `*.pg.test.ts` test covering the new behavior. Mock coverage is not sufficient — it will pass on a broken migration. The suite is intentionally small (compliance gate, not a rewrite of the mock suite); add only smoke-level tests for new DB-layer constructs.

---

## Database & Migrations

**Location**: `supabase/migrations/` — 118 files. Early migrations use sequential numbering (`20240101000001`–`20240101000038`), later ones use real timestamps.

### Key Tables (~60)

**Multi-tenant**: `companies`, `company_members`, `company_invitations`, `teams`, `team_members`, `team_invitations`, `user_preferences`, `profiles`

**Bookkeeping**: `chart_of_accounts`, `fiscal_periods`, `journal_entries`, `journal_entry_lines`, `account_balances`, `voucher_sequences`, `voucher_gap_explanations`

**Invoicing**: `customers`, `invoices`, `invoice_items`, `invoice_payments`, `invoice_inbox_items`

**Suppliers**: `suppliers`, `supplier_invoices`, `supplier_invoice_items`

**Banking**: `bank_connections`, `transactions`, `bank_file_imports`, `payment_match_log`

**Documents**: `document_attachments` (WORM), `receipts`, `receipt_line_items`

**Settings & Config**: `company_settings`, `mapping_rules`, `categorization_templates`, `booking_template_library`, `extension_data`

**Dimensions**: `cost_centers`, `projects`

**Tax & Deadlines**: `tax_rates`, `tax_table_rates`, `deadlines`, `calendar_feeds`, `skatteverket_tokens`

**API & Auth**: `api_keys` (with scopes), `oauth_used_codes`, `bankid_identities`

**Audit & Ops**: `audit_log` (immutable), `event_log` (30-day TTL), `pending_operations`, `processing_history` (+ `processing_event_types`), `ai_usage_tracking`, `voucher_gap_explanations`, `automation_webhooks`

**Inbox & Migration**: `invoice_inbox_items`, `company_inboxes`, `email_connections`

**Salary**: `employees`, `salary_runs`, `salary_run_employees`, `salary_line_items`, `salary_payroll_config`, `agi_declarations`

**Third-party providers**: `provider_consents`, `provider_consent_tokens`, `provider_otc`

**Other**: `sandbox_users`

### Key RPC Functions

- `create_company_with_owner()` — Atomic company + owner creation
- `commit_journal_entry()` — Atomic draft→posted with voucher number
- `next_voucher_number()` — Concurrent-safe voucher generation
- `detect_voucher_gaps()` — BFNAR 2013:2 gap detection
- `generate_invoice_number()`, `get_next_arrival_number()`, `generate_delivery_note_number()` — Sequence generators
- `seed_chart_of_accounts()` — BAS chart seeding per entity type
- `validate_and_increment_api_key()` — Atomic rate limiting
- `user_company_ids()` — RLS helper returning user's company IDs
- `get_unlinked_1930_lines()` — Bank reconciliation helper
- `cleanup_sandbox_user()`, `cleanup_expired_sandbox_users()` — Sandbox lifecycle

### Key Triggers

- `check_journal_entry_balance()` — Debit must equal credit
- `enforce_journal_entry_immutability()` — Posted entries cannot be modified
- `enforce_period_lock()` — No entries in closed/locked periods
- `enforce_company_lock_date()` — Company-wide bookkeeping lock date
- `block_document_deletion()` — WORM compliance
- `enforce_retention_journal_entries()` — 7-year retention
- `audit_log_immutable()` — Audit log cannot be modified
- `write_audit_log()` — Auto-audit on DML operations
- `sync_team_member_to_companies()` — Auto-sync team→company membership

### Migration Rules

1. **Always enable RLS** and create policies using `user_company_ids()` for company-scoped data
2. **Always add `updated_at` trigger** using `update_updated_at_column()`
3. **UUID primary keys**: `DEFAULT uuid_generate_v4()`
4. **Company ownership**: `company_id UUID REFERENCES companies NOT NULL` + `user_id UUID REFERENCES auth.users ON DELETE CASCADE NOT NULL`
5. **Never modify existing migrations** — create new ones
6. **Never modify enforcement triggers** (migration 017) — legally required
7. **Apply via Supabase MCP tool**: `mcp__plugin_supabase_supabase__apply_migration`
8. **Always include `NOTIFY pgrst, 'reload schema'`** at the end of migrations that alter table structure (ADD/DROP COLUMN, CREATE TABLE, ALTER TYPE). Without this, PostgREST serves stale schema until next reload.

---

## Skills, Git & CI

**Skills**: Always use `/frontend-design` for new UI. Use `vercel:deploy` for deployment. Use `/supabase-migration` for new migrations. Use `/erp-api-route` for new API routes. Use `/create-extension` for new extensions. Use the Swedish domain skills (`swedish-sie-import-export`, `swedish-accounting-compliance`, `swedish-vat`, `swedish-invoice-compliance`, `swedish-payroll`, `swedish-year-end-closing`, `swedish-financial-reporting`, `swedish-sru-filing`, `swedish-asset-accounting`, `swedish-project-accounting`, `swedish-tax-planning`) for accounting domain questions.

**Git**: Conventional commits (`feat:`, `fix:`, `refactor:`, `test:`, `docs:`). Atomic commits, branch from `main`.

**CI**:
- `.github/workflows/core-build.yml` — resets extensions to empty, runs build + test, verifies no core code imports from `@/extensions/` directly.
- `.github/workflows/swedish-compliance-review.yml` — Swedish accounting compliance review on PRs touching bookkeeping/reports/tax logic.
- `.github/workflows/docker-publish.yml` — pushes images to GHCR on main.

**Docker** (`.github/workflows/docker-publish.yml`): Pushes to GHCR (`erp-mafia/erp-base`) on main push. 4-stage Dockerfile (base → deps → builder → runner) with Node 22 Alpine. Runtime env placeholder replacement via `docker-entrypoint.sh`. Docker Compose with app + supercronic cron service.

---

## Deployment

### Vercel (Hosted)

Cron jobs defined in `vercel.json`:

| Schedule | Endpoint | Purpose |
|----------|----------|---------|
| `0 6 * * *` | `/api/deadlines/status/cron` | Update deadline statuses |
| `0 8 * * *` | `/api/invoices/reminders/cron` | Send invoice reminders |
| `0 0 2 1 *` | `/api/tax-deadlines/cron` | Generate tax deadlines |
| `0 5 * * *` | `/api/extensions/enable-banking/sync/cron` | Bank transaction sync |
| `0 3 * * *` | `/api/documents/verify/cron` | Document integrity verification (daily) |
| `0 4 * * *` | `/api/sandbox/cleanup/cron` | Sandbox user cleanup |
| `0 2 * * *` | `/api/events/cleanup/cron` | Event log cleanup (30-day TTL) |
| `0 * * * *` | `/api/extensions/cloud-backup/auto-sync/cron` | Hourly cloud backup auto-sync |

### Docker (Self-Hosted)

- `Dockerfile`: 4-stage Node 22 Alpine build with standalone output
- `docker-compose.yml`: App service + supercronic cron scheduler
- `docker-entrypoint.sh`: Validates required env vars, replaces build-time placeholders in `.next/static/` JS
- Extension presets: `docker/extensions.self-hosted.json`, `docker/extensions.hosted.json`

### Environment Variables

**Required**: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `NEXT_PUBLIC_APP_URL`, `CRON_SECRET`

**Auth**: `NEXT_PUBLIC_REQUIRE_MFA` (set `true` on hosted), `NEXT_PUBLIC_SELF_HOSTED` (set `true` for Docker)

**Extension-specific** (only when extension is enabled): `ENABLE_BANKING_APP_ID`/`ENABLE_BANKING_APP_KEY`, `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `RESEND_API_KEY`, `VAPID_PUBLIC_KEY`/`VAPID_PRIVATE_KEY`

**Optional**: `SENTRY_DSN`, `SENTRY_AUTH_TOKEN`

## Other
Never create a NUL/nul file: \gnubok\NUL

---

## Design Context

### Users

Swedish sole traders (enskild firma) and small business owners (aktiebolag) who need to manage their own bookkeeping. They are not accountants — they are professionals (consultants, freelancers, shop owners) who want to stay compliant without hiring one. They use gnubok in short, focused sessions: sending an invoice, categorizing bank transactions, filing a VAT declaration. Speed and clarity matter — every second spent in the app is a second away from their real work.

### Brand & Aesthetic

**Minimal. Sharp. Efficient.** The interface should feel like a well-made instrument: considered, quiet, and confident. Reference: Mercury (banking). Anti-reference: enterprise software (SAP/Oracle density).

- **Palette**: Grayscale foundation with restrained semantic colors — sage green (success/balance), terracotta (errors/overdue), ochre (warnings/attention). No loud brand color.
- **Typography**: Fraunces (serif) for display headings, Geist (sans) for body. Tabular numbers everywhere financial data appears.
- **Surfaces**: White/near-white cards on light gray backgrounds. Subtle borders (60% opacity). Soft shadows. Dark mode follows the same restraint.
- **Spacing**: Generous whitespace. Dense data (tables, ledgers) uses tighter spacing but never feels cramped.
- **Motion**: Subtle and purposeful. Stagger animations for list entry, spring easing for feedback. Never decorative.
- **Icons**: Lucide — 15px in navigation, slightly larger in empty states.

### Design Principles

1. **Clarity over cleverness.** Every element immediately understandable. Clear labels (in Swedish), obvious hierarchy.
2. **Earned minimalism.** Remove what doesn't serve the task, but don't strip context that prevents compliance errors.
3. **Numbers are first-class.** Tabular-nums, proper alignment, adequate contrast, clear positive/negative distinction.
4. **Trust through consistency.** Same patterns, spacing, and behavior everywhere.
5. **Speed is a feature.** Optimize for the 90-second session.

### Accessibility

- **WCAG AA**: 4.5:1 text contrast, 3:1 UI components
- Keyboard-navigable with visible focus rings
- Respect `prefers-reduced-motion`
- Color never sole indicator of state — always pair with icons, text, or shape

---
> Source: [erp-mafia/gnubok](https://github.com/erp-mafia/gnubok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
