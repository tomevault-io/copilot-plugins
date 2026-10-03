## opseron

> - `--op-*` (navy `#0F172A` + teal `#0EA5A4` accent) is the canonical system

# Agent Context

## Token System
- `--op-*` (navy `#0F172A` + teal `#0EA5A4` accent) is the canonical system
- `--c-*` aliases maintained in `global.css` for backward compat
- `check-token-usage.mjs` enforces 0 `var(--c-*)` and 0 hardcoded hex violations
- All token migration complete — 0 VAR / 0 HEX violations

## Theme
- `ThemeContext.tsx` sets `data-theme="light"` explicitly (not `removeAttribute`)
- `[data-theme="night-shift"]` overrides use `var(--op-*)` references
- `@media (prefers-color-scheme: dark)` excludes `[data-theme="light"]`
- `--brand-lt` is teal `var(--op-accent-lt)`, not green

## Responsive Grids
- Utility classes in `components.css`:
  - `.grid-2/3/4/5` — equal-column grids, collapse to 1 col on mobile
  - `.grid-weighted` — preserves inline `gridTemplateColumns` on desktop, collapses to 1 col on mobile
  - `.layout-sidebar`, `.layout-sidebar--280`, `.grid-detail`, `.version-list-*`
- `Grid.tsx` uses `op-grid--{columns}` class
- `FormGrid` uses `op-grid--{columns}` class
- Mobile breakpoint: `!important` overrides via `@media (max-width: 767px)`
- Table-like weighted grids (header + rows) skip responsive treatment — horizontal scroll is acceptable

## TypeScript
- CrmForm uses `UseFormReturn<T, any, T>` for RHF 7.71 compatibility
- `unknown` values from `Record<string, unknown>` cast via `String()` when used as ReactNode

## Key Files — Architecture Infrastructure
- `frontend/shared/contexts/ThemeContext.tsx` — theme attribute logic
- `frontend/shared/styles/global.css` — token aliases + night-shift overrides
- `frontend/shared/styles/components.css` — grid utility classes
- `frontend/shared/ui-kit/styles.css` — `op-grid--*` classes + mobile overrides
- `scripts/check-token-usage.mjs` — CI guard

### Public Website (frontend/website)
- Static Astro 7 site, 6 locales (`en` at root, `fr/de/es/zh/ar` prefixed — `prefixDefaultLocale: false`). EN pages live at `src/pages/` root, other locales under `src/pages/[locale]/`; both wrap shared `src/components/pages/*Body.astro` via `PageShell`.
- Content is typed per-page in `src/i18n/content/{en,fr,de,es,zh,ar}.ts` (en is source of truth); `contentFor(locale)` assembles ui + content (falls back to en).
- NO `tsconfig.json` in the package on purpose — `typecheck-frontend.mjs` auto-skips it; use `tsconfig.typecheck.json` locally for `src/i18n` + `src/utils` checks.
- Hex colors only in `src/styles/tokens.css` (allowlisted in `check-token-usage.mjs`); `.astro` files are not scanned by the token guard.
- Blog uses Astro Content Layer (`src/content.config.ts`, `glob` loader — `type: "content"` is removed in Astro 7); posts keyed by `lang` in frontmatter.
- Social images: `pnpm run og` → `scripts/gen-og.mjs` (satori + sharp, static TTFs in `scripts/fonts/`, outputs `public/og/{locale}/home.png` + `og/en/logo.png`).

### Bridge Files (shared/lib)
- `commandBridge.ts` — Command registration events + types
- `searchBridge.ts` — Search provider registration + result types
- `objectTypeBridge.ts` — Object type registration + definition types
- `aiBridge.ts` — AI action registration + stream chunk types
- `relationshipBridge.ts` — Relationship type registration + record types
- `eventBus.ts` — Typed EventBus class, middleware, bridge adapter, EventRegistry
- `taskOrchestrator.ts` — DAG task execution with rollback + progress events
- `queryClient.ts` — TanStack Query client config + cache helpers
- `workspaceStorage.ts` — localStorage persistence helpers

### Pattern Components (shared/ui-kit/patterns)
- `DataGrid.tsx` — Virtualized grid with sort/filter/selection/resize
- `AIActionButton.tsx` — AI action button with streaming output + input prompt
- `RelationshipPanel.tsx` — Related objects list with link/unlink
- `RelationshipGraph.tsx` — SVG relationship graph (0 deps)

### Context Providers (shared/contexts)
- `AIInteractionContext.tsx` — AI action execution with SSE streaming
- `RelationshipRegistryContext.tsx` — Relationship CRUD via API
- `WorkspacePersistenceContext.tsx` — Workspace state save/restore (localStorage)
- `QueryProvider.tsx` — TanStack QueryClientProvider wrapper

## Work State

### Completed
- **GSC "Page with redirect" fix (2026-08-09)**: trailing-slash 301s from `frontend/website/nginx.conf` emitted *absolute* Location headers built from the internal http request (`/about` → 301 `http://opseron.com/about/` → 301 `https://opseron.com/about/`) — a scheme-downgrade chain flagged by Search Console. Added `absolute_redirect off;` + `server_name_in_redirect off;` + `port_in_redirect off;` → all `return 301` now emit relative `Location: /about/`, single hop, https preserved. Deploy: `docker compose build website && docker compose up -d website`, then re-run GSC "Validate fix".
- **Names instead of UUIDs + Copilot task creation repair (2026-08-06)**: UUIDs leaked everywhere because APIs only returned raw `ownerId`. Fixes: (1) sales-svc `enrichOwnerNames()` (batched profilesTable join) adds `ownerName` to `GET /leads`, `GET /leads/:id`, `GET /leads/opportunities`, `GET /leads/opportunities/:id` — the UIs already preferred `ownerName ?? ownerId`; (2) copilot `context.ts` `resolveUserNames()` augments entity records with `ownerName`/`assigneeName`/`createdByName` via people-svc `GET /api/users/:userId` + copilot.md rule "never print raw UUIDs"; (3) copilot `checkPermission` ABAC `not_applicable` (no tenant policy rows in `auth_abac_policies`) no longer blocks admin/tenant_owner/super_admin — fixes "You don't have permission to create task records" (ABAC `decision:not_applicable` returned for every resource without policies); (4) task creation: `insertTaskSchema`/`updateTaskSchema` in operations-svc project-service schema were drizzle-zod-derived `z.uuid()` — zod v4 rejects version-0 seed ids (`tenantId Invalid UUID` on every POST); rewritten as explicit `z.object` with `laxId = () => z.string().min(1)` (same trap class as the collab fix); copilot `createRecord` now resolves `projectId` (entity context → tenant's first project) since tasks are project-scoped; (5) "yes create the follow-up task" UX: chat.ts passes the last assistant message to `dispatch` → `createRecord` merges it into extraction when the user input is terse (no own details), quoted-phrase extraction runs first, task-specific regex rejects keyword-word ("task")/UUID captures; (6) qualify 500 "AI returned an invalid response format": `parseAIResponse` extracts first balanced `{...}` from prose-wrapped JSON + one strict repair retry; agent AI call timeout 30s→60s (model exceeded 30s). Live-verified: ownerName on all endpoints, task created with correct title then deleted (204), qualify 200 full result, logs clean. Commits 1ab55fe7/6c633264/68032534/516e3348.
- **Lead edit/assign UI (2026-08-06)**: LeadOverview edit mode auto-opens from `?edit=1` (LeadDetail header Edit now lands in a working form instead of a dead state), expanded fields (status, owner select from `/api/users` — people-svc profilesTable `userId`/`firstName`/`lastName`, source, website, industry, estimated value), Save payload skips empty optionals so cleared `contactEmail` can't fail `z.email()` validation, Save/Cancel clears the `edit` param; LeadSidePanel `InlineField` gained explicit ✓/✕ save+cancel buttons (mousedown-prevented so blur can't double-save); `updateLeadSchema.ownerId` relaxed `z.string().uuid()` → `z.string().min(1)` — the seed sales reps (`00000000-...-0111` etc.) are version-0 ids and were rejected, same trap class as the collab fix. Live-verified on crm.opseron.com: full PATCH 200 (assignment + status/source/website/industry/estimatedValue persisted), reverted after. Also cleaned polluted lead data (website/industry had been seeded with the app URL `https://leads.opseron.com/leads/list`). Note: login body requires `tenantSlug` (gateway `validate.ts` `/login` schema is `.strict()`). Commits 37e4e7c8.
- **Lead detail CRUD repair (2026-08-06)**: comments 400 → 201 (zod v4 `uuid()` rejects version-0 UUIDs like the seeded `00000000-...-0001` tenant/admin — collab insert schemas now validate id fields as opaque `z.string().min(1)` instead of drizzle-zod-derived `uuid()`; keep this in mind for any new schema on id columns that receive gateway header values); file upload 500 → 201 (`permission denied for table leads` — svc_platform lacked SELECT on entity tables; migration `046_platform_entity_existence_grants.sql` + per-service-grants.sql block); AI qualify 400/500 → 200 (tenantId/userId now optional in copilot route schemas with `bodyIdentity()` falling back to `x-tenant-id`/`x-user-id` headers — headers always win over client body incl. stale "default" placeholders; `context.ts` URL join double-slash fixed — `ENTITY_PATHS` values already carry a leading `/`); pipeline-value MV refresh 500 → 200 (best-effort: nothing reads the MV; fallback CREATE now `IF NOT EXISTS` + non-concurrent REFRESH + logged, job stays green). Frontend `LeadAiTab.tsx`/`LeadOverview.tsx` no longer send fake `tenantId:"default"`. Deployed as 87bfa3c0/2c516067/e8a232b6.
- **P0**: `usePermissions` + `RequireAction`, token aliases, WorkspaceContext
- **P1a**: Command System — commandBridge, CommandRegistryContext, CommandPalette
- **P1b**: Universal Search — searchBridge, SearchProvider, GlobalSearch
- **P1c**: Object Model — objectTypeBridge, ObjectTypeRegistry, ObjectWorkspace
- **P2a**: Data Grid — useDataGrid, DataGrid, GridTable wrapper (Table-compatible)
- **P2b**: AI Interaction — aiBridge, AIInteractionContext, useAIAction, AIActionButton
- **P2c**: Relationship Engine — relationshipBridge, RelationshipRegistry, useRelationship, RelationshipPanel
- **P3a**: Relationship Graph — RelationshipGraph (SVG, no deps)
- **P3b**: Workspace Persistence — workspaceStorage, WorkspacePersistenceContext
- **System**: Event Bus — EventBus class, middleware, bridge adapter, useEvent hook
- **System**: Service Orchestration — taskOrchestrator (DAG+rollback), useTaskOrchestration
- **System**: Caching Layer — queryClient, useCachedQuery, useCachedMutation (optimistic updates)
- **Refinements**: LeadAiTab AIActionButton integration, TicketRelatedTab RelationshipPanel, 50+ list virtualized via GridTable
- **Polish**: EventBus bridge adapter bug fix, NPE safeties, try/catch boundaries, React.memo, aria-labels
- **Token + Class Migration (Leads App)**: All `var(--color-*)`→`var(--op-*)`, `var(--c-*)`→`var(--op-*)`, `var(--shadow-*)`→`var(--op-shadow-*)`, `var(--sf-brand)`→`var(--op-accent)`. Class migration: `btn`→`op-btn`/`op-btn--*`, `card`→`op-section-card`/`__*`, `badge`→`op-badge`/`--*`, `opp-kanban-*`→`op-kanban-*`/`__*` across 27 files. Fixed 3 TS errors in `LeadSidePanel.tsx` (missing `useTranslation` in `InlineField`). **0 TS errors, 0 `var(--c-*)`, 0 `var(--color-*)` in leads.**
- **P0 Architecture Freeze Remediation (2026-08-01)**: see `docs/architecture/runbooks.md` + `deployment-guide.md` + `developer-onboarding.md`.
  - **Platform now runs 9 production services** (api-gateway, auth-service-go, sales-svc, operations-svc, platform-svc, people-svc, finance-svc, launcher-service, service-service) — the legacy fleet was decommissioned (28 packages removed: 24 comment-only stubs + 4 with `src/` deleted) and its routes absorbed into `src/vendored/`. No decommissioned package boots an HTTP server or a Kafka consumer. Frontend is consolidated into a single shell build bundling all 22 modules.
  - **Route trees absorbed into `src/vendored/`**: sales-svc → catalog/commerce/marketing-service; operations-svc → project/ticket/workflow/mrp-service; finance-svc → finance/procurement-service; platform-svc → config/developer/integration/compliance/copilot/job-service (developer connectors + etl-pipelines mounted at `/api/developer/connectors` + `/api/developer/etl-pipelines`).
  - **Background consumers now start from consolidated services**: operations-svc `src/background.ts` (workflow/hr-events/orders/inventory/procurement consumers), finance-svc `src/kafka/index.ts` (payments/payroll/invoices/work-orders/inventory), platform-svc `src/scheduler.ts` (BullMQ jobs). All gate on `KAFKA_BROKERS`.
  - **Gateway defaults + launcher port fixed**: `api-gateway/src/routes/proxy.ts` defaults to consolidated ports (3003/3007/3009/3010/3015); legacy env aliases map to consolidated targets; launcher default port `3027` (`LAUNCHER_PORT ?? 3027`).
  - **docker-compose**: every `*_DATABASE_URL` now `${VAR:-default}` (dev-only placeholders, production overrides via `.env`/SOPS); grants mounted at DB boot — `00-init.sql` (init.sql) → `99-per-service-grants.sql` (per-service-grants.sql, ADR-002).
  - **Migration runner**: `scripts/apply-db-migrations.mjs` + `schema_migrations` ledger (SHA-256 checksums, UP/`-- DOWN` sections, per-file transaction); migrations `001`–`006` in `docker/postgres/migrations/`.
  - **Frontend registry now 22 modules** (`frontend/shell/src/modules/moduleRegistry.ts` — collaboration + notifications added this wave); CI typechecks all 25 frontend packages via `scripts/typecheck-frontend.mjs`.
  - **Duplicate Kafka producers/consumers eliminated**; only consolidated owners produce/consume (sales-svc `src/kafka/producer.ts`, people-svc `src/kafka/producer.ts`, finance-svc `src/kafka/index.ts`, operations-svc `src/background.ts`, platform-svc `src/scheduler.ts`).
  - **Consumer idempotency wired** via shared `services/shared/src/kafkaIdempotency.ts` (`isAlreadyProcessed`/`markProcessed`, unique `(event_id, consumer_id)` in `kafka_processed_events`) — people-svc `platformEventConsumer`, platform-svc `fieldUpdateConsumer`, operations-svc workflow triggers, finance-svc `paymentsConsumer`.
  - **`platform.event` producer + `eventVersion` envelope (ADR-003)**: consolidated owners publish via the shared producer (`services/shared/createService.ts`) with `eventVersion: "1.0"` / `schemaVersion`, `eventId`, `tenantId`, `publishedAt`, `publishedBy`.
  - **Gateway circuit breakers (ADR-018)**: `api-gateway/src/routes/proxy.ts` (search fan-out) + `src/routes/bff.ts` use `getOrCreateBreaker` from `@Opseron/shared/circuitBreaker.js`; open circuit → 503.

### TypeScript Status
- `frontend/shared` — 0 errors
- `frontend/shell` — 0 errors
- `frontend/leads` — 0 errors
- `frontend/tickets` — 0 errors
- Consolidated services — 0 errors (re-verified 2026-08-01 via `tsc --noEmit`: `sales-svc`, `operations-svc`, `platform-svc`, `people-svc`, `finance-svc`, `api-gateway`; `pnpm run typecheck:libs` clean). Note: a first batch run reported transient `Type 'X' is not assignable to type 'never'` errors in operations/platform/people/finance-svc (stale shared build output); immediate re-run passes — if you see these, rebuild `@Opseron/shared` first.
- Decommissioned legacy packages — comment-only stubs / no `src/` (excluded from the active typecheck graph)
- ESLint: 1 warning (`@tanstack/react-virtual` memoization — known/acceptable)

### Route Ordering Bug Fix (Widgets Showing "No Data Available")

**Problem:** Express `/:id` catch-all routes defined before single-segment stat endpoints caused `/stats`, `/health-distribution`, `/events` to be intercepted as `:id` parameters (HTTP 500).

**Fixed (same HTTP method `/:id` before named route — actually broken):**
- `services/sales-svc/src/routes/leads/leads.ts` — Moved `GET /stats` before `/:id`
- `services/project-service/src/routes/projects.ts` — Inserted 9 widget stat endpoints + `/events` before `/:id`. Removed all duplicates and dead code.
- `services/hr-service/src/routes/employees.ts` — Moved `GET /stats` before `GET /:id`
- `services/people-svc/src/routes/hr/employees.ts` — Mirror of above
- `services/ticket-service/src/routes/tickets.ts` — Moved `GET/POST /canned-responses` + PATCH/DELETE before `GET /:id`
- `services/developer-service/src/routes/webhooks.ts` — Moved `GET /deliveries`, `GET /dlq` before `GET /:id`
- `services/platform-svc/src/routes/developer/webhooks.ts` — Mirror of above
- `services/presales-service/src/routes/presales.ts` — Moved all `/technical-requirements` routes before `GET /:id`

**Not actually broken (different HTTP methods or GET /:id correctly at end):**
- `finance analyticAccounts.ts` — Only `PATCH /:id`/`DELETE /:id`, no `GET /:id`
- `finance fixedAssets.ts` — `GET /:id` correctly at end (line 743), after all named GET routes
- `CRM accounts.ts` — `GET /:id` only, `POST /bulk-delete` different method
- `CRM territories.ts` — `PATCH /:id`/`DELETE /:id`, `POST /assign` different method
- `config approvalWorkflows.ts` — `PATCH /:id`/`DELETE /:id`, POST/GET named routes different methods
- `MRP quality-inspections.ts` — `GET /:id` on separate router instance from `/rate`
- `procurement inventory.ts` — `GET /:id` only, `POST /transfer` different method

### P2.11 Enterprise Readiness (Phase 2 close — all 14 gates Yes)
See `docs/architecture/master-data-governance.md`. Summary:
- **Security**: knowledge cross-tenant reads/writes closed (articles.ts all 6 handlers); file uploads validated (MIME allowlist, 100MB cap, size verification, filename sanitization); download/delete ownership checks; documents DELETE route added; gateway tenant middleware fails closed (503); aggregator forwards x-user-team-id/x-user-territory-id.
- **Search permission-aware**: `hasModuleAccess` helper (services/shared/src/middleware/requirePermission.ts:116, parses `x-module-roles` JSON `{moduleId: role}`, null→fail-open, admin/* bypass) gating the /search aggregator + all 17 providers.
- **Transactions**: payroll, employee+comp, lead merge/create/bulk-stage/CSV, PO, orders, work-order+operations; Repository gains `transaction()` + `upsert(conflictTarget)` (onConflictDoUpdate); RecycleBinService restore atomic; ImportEngine `maxErrors` enforced; import sync failures write `failed` job rows; custom-object import idempotent (natural-key upsert); scheduled imports wired (BullMQ repeatable, `IMPORT_SCHEDULER_ENABLED`); formula route `POST /api/config/custom-fields/evaluate`; fieldUpdateConsumer starts with `KAFKA_BROKERS`; customFieldValues enforces entityType match (422).
- **Lifecycle/history**: recycle-bin snapshots on lead/opportunity/ticket/project/quotation deletes (best-effort); audits added for order confirm/cancel, fixed-asset status, expense approvals, dunning runs; `hr_lifecycle_events` written on employee status/termination.
- **Validation sweep**: zod added — project tasks/milestones/resources/time-entries, copilot 12 handlers, ticket 11 handlers (incl. portal + bulk), sales-svc convert/activities/bulk/merge/territories/scoring/crm-tasks, config approvals/workflows; mass-assignment closed.
- **DB (init.sql `---- P2.11:` sections :6522+)**: 46 FK indexes, unique keys (leads contact_email, accounts company_name, contacts email, employees employee_number), CHECKs (lead score/value, opportunity probability), 28 missing tables ported from lib/db.
- **Dynamic objects (P2.3)**: 10 field types (url/email/relation/formula added), defaults, schema `version`/`parentObjectName` inheritance, createdById; config-service CRUD mirrors; frontend `FieldType` + `BuiltInRenderers` extended (relation renderer added); OpenAPI enum 10 types + duplicate operationIds fixed; api clients regenerated (codegen drift check in CI).
- **CI**: `.github/workflows/enterprise-checks.yml` — token guard, shared build, codegen drift, frontend tsc (shared+config), init.sql index-name uniqueness (468 unique names, 0 dups).
- **Fixed pre-existing**: dangling `services/compliance-service/src/schema/index.ts` import of deleted `./compliance.js` → `complianceAuditLogTriggerSQL` added to lib/db schema + local re-export created; full platform-svc bundle now BUILD_OK.
- **Final verification round (P2.11 close)**: 6 read-only auditors + 7 fix agents. All bugs found were fixed: init.sql dedupe (54 dup tables removed, 468 unique index names), read-only branch bypass closed (branchAccess.ts:164), ticket+project search branch-scoped, milestone/team zod+tenant, operations-svc raw SQL aligned, custom-fields full validation (10 types) + `requirePermission("config","admin")` + both consumers load ALL values + evaluate route, relationship restrict/cascade/self-link/bidirectional-409 + **one_to_many target-side cardinality**, documents version access control + sanitized keys, lead merge email-collision in-tx, vendor/quotations zod+tenant, accounts/presales recycle-bin snapshots, payroll duplicate-run guard + GL failure → 502, MRP reserve failure propagates, importPersistence per-row errors for missing natural keys, merkle roots anchored (`AppendRootChain`, rootChain.go:17) + verification scans `changes` column, per-module `admin`/`*` in hasModuleAccess.
- **Second audit round (8 auditors, every finding fixed incl. LOWs)**: +20 FK leading-column indexes (init.sql), RLS for file tables in bootstrap, hr_payslips unique key, audit_merkle_roots + notification_email_templates tables in init.sql, project_milestones status/updated_at aligned (lib/db + init.sql + service), marketing_scoring_rules 3 columns, deduped branch_id ALTERs; order-confirm atomic status claim (`WHERE status='draft'` in tx) + sales-mirror compensation; payroll GL idempotency (reference_id) + conditional transition; **all single/bulk deletes snapshot-in-tx** (RecycleBinService client param); Repository.restore single-tx; custom-object **formula fields computed** on create/update + records/values gated `hasModuleAccess` (not admin) + parent validation on create+PATCH + `_deleted` count filter; milestone branch isolation (parent-project subquery, all 5 routes); vendors products tenant scope; **exports permission-gated on all 4 routes**; **`DELETE /import/scheduled/:id` cancel endpoint**; presigned PUT ContentLength condition + `POST /documents/:id/confirm` size verification; **`canMutate()`** separates destructive ops from read visibility; documents version keys slugified; distinct Kafka consumer group ids; people-svc hr_lifecycle_events mirrored; merkle inline retry; notification/people runtime CREATE TABLE removed; ImportWorker + platform-svc scheduler BullMQ `defaultJobOptions` type fixes; **one_to_one target-side cardinality** + PG 23505 → 409.
- **Exit criteria CLOSED — shared data platform enforced**: project-service + marketing-service migrated to `@workspace/db/schema` re-exports (0 local pgTable); platform-svc 61 dead duplicate defs deleted (4 shim files, 1 infra keeper kafka_processed_events); **10 legacy shells (catalog/launcher/job/tenant/service/notification/integration/collab/workflow/crm) migrated** — tables added to lib/db (incl. new `kafka.ts` with kafka_processed_events + kafka_dlq superset; `jobs.ts`), crm-service DECOMMISSIONED placeholder, workflow dead file deleted; 0 `pgTable(` in any migrated service; lib/db is the single schema source of truth.
- **Build-system fix**: `services/shared/package.json` now declares `zod`, `bullmq`, `csv-parse`, `xlsx`, `ioredis` (used by `src/import/*` + DynamicObjectService) — platform-svc + config-service previously failed the canonical `pnpm build` with "Could not resolve" errors. All 15 services + auth-service-go (`go build`) + frontend shell/shared tsc are green via the canonical build. Final sweep: **25/25 services esbuild green**, services/shared + lib/db tsc 0 errors, init.sql 491 unique indexes / 387 tables / 0 dups.

### Branch Isolation (Gate 6 closure — production enforcement)
Gateway is the enforcement point; services filter by row `branch_id`:
- **Gateway** `services/api-gateway/src/middleware/branchAccess.ts` (mounted after tenantMiddleware in app.ts:263): validates `x-branch-id` against the user's effective branches (POST `/grants/cross-branch/effective`, cached 60s/user + 5min/tenant, fail-closed 503); read-only cross-branch grants reject mutating methods; absent header → home branch injected for regular users (single-branch default), all-branches merged mode only for hierarchy roles (`BRANCH_OVERRIDE_ROLES` env, default admin/tenant_owner/super_admin/branch-manager/manager); also mounted on /api/bff + bff forwardHeaders forwards x-branch-id; BFF cache key includes branch (`cache.ts:14`).
- **auth-service-go**: new `GET /me/branches` (branches + homeBranchId + canViewAllBranches; hierarchy roles get all tenant branches); hardening — AssignUser verifies user belongs to tenant, Delete/RemoveUser now 404 on RowsAffected 0.
- **Service pattern** (19 tables): reads filter `branch_id = X OR branch_id IS NULL` (NULL = tenant-global, visible in any branch view); creates populate `branch_id` from header (`?? null`). Applied in sales-svc (leads/opportunities/quotations/presales/accounts/campaigns/orders/returnOrders/contracts + internalStats), commerce-service, marketing-service, project-service, ticket-service, hr-service + people-svc employees, finance-service (journalEntries/customerInvoices/vendorInvoices), procurement-service (rfq/purchaseRequests/purchaseOrders). user_profiles has no branch_id → membership join stays.
- **Frontend**: BranchContext fetches `/api/auth/me/branches`; default = localStorage (validated) ?? (canViewAllBranches ? null : homeBranchId); TopBar BranchSelector lists only the user's branches ("All Branches" option only when canViewAllBranches), Home marker on isDefault; WorkspacePersistenceContext no longer owns activeBranchId (single source of truth = BranchContext localStorage).
- **Gate 9 exports**: `GET /tickets/export` (tickets.ts:744, requirePermission), `GET /projects/export` (projects.ts:749), `GET /orders/export` (commerce-service + sales-svc mirror, orders.ts:75) — CSV, tenant+branch scoped, registered before `/:id`.

### Phase 3.4 MRP — Enterprise Manufacturing Module (8 workstreams complete)
MRP scope below. All backend/frontend builds + typechecks green.
- **Schema (lib/db)**: catalog product master (`catalog_product_type` enum, family, barcode/qr, revision; SKU leadTimeDays/orderMultiple/safetyStock/reorderPoint/defaultWarehouseLocation; `catalog_uom_conversions`), MRP expansion (`mrpBomTypeEnum`, `pending_approval` status, BOM type/effective dates/lock/template/baseBom/approval fields, items scrap/yield/optional/substituteGroup/phantom, `mrp_bom_substitutes` + `mrp_bom_approvals`, full WO lifecycle enum draft/approved/paused/blocked/closed + batch/priority/pause/block fields + `mrp_work_order_status_history`, `mrp_ncrs`/`mrp_capa`/`mrp_inspection_plans`, quality stage + inspection_plan_id, routing version/lock + queue/move time + machine/labor cost + work_instructions, branch_id across 8 tables). Migration `028_mrp_enterprise_expansion.sql` + `init.sql` parity.
- **Catalog integration fixed**: MRP no longer calls dead port 3022 — `SALES_SERVICE_URL ?? http://localhost:3003` + `/api/catalog/...`. sales-svc gained `GET /skus/:id` (with productName join) + `POST /internal/sku-metadata` (bulk). Fixed boms.ts/capacity.ts/mrpRuns.ts call sites.
- **Router wiring**: `routingsRouter` + `defectsRouter` mounted (previously never registered → 404s); `GET /capacity-planning` implemented (per-week utilization per work center, calendar-aware availability).
- **Real stats (no stubs)**: all 9 decision-engine stat endpoints (`/stats/efficiency|wip|on-time-delivery|inventory-turns|capacity/utilization|production-schedule|demand-forecast|materials/shortages|alerts/supply-chain`) computed from DB via `vendored/mrp-service/stats.ts`; `app.ts` dead code marked DEPRECATED (still imported by orphaned `vendored/mrp-service/index.ts`).
- **BOM endpoints**: type/status filters, full create with enterprise fields, PATCH with one-way lock (409 BOM_LOCKED), `by-product/:skuId` (effective-date aware), `:id/clone` (baseBomId), items CRUD w/ scrap/yield, substitutes CRUD, approvals workflow (draft→pending_approval→active + approvedById/At), `:id/full` with names.
- **WO lifecycle**: full state machine (approve/pause/resume/block/unblock/close/cancel + release/start/complete) via central `applyTransition` writing `mrp_work_order_status_history` + audit events; draft create + locked PATCH; materials requirement/issue/backflush (inventory decrement + shortage flags, raw `mrp_wo_material_consumption`); outputs (finished-goods receipt, inventory credit, `mrp_wo_outputs`); status-history read.
- **Quality**: NCR + CAPA CRUD + status machines (`ncr-capa.ts`), auto-numbering `NCR-YYYYMMDD-XXXX`/`CAPA-...` with advisory-lock serialization + 23505 retry, inspection-plans CRUD (referential delete guard), `quality-metrics` + `defects-pareto` in stats.ts.
- **Costing/analytics**: `costing.ts` — per-WO cost breakdown (material via finance `/internal/material-costs`, labor via time logs × work-center rate, overhead via routing machine cost), analytics (by work center/product), estimated-vs-actual variance, cost-trend, production, scrap — all computed live (no stored cost columns on WOs), nulls not fabrications.
- **Pagination**: `GET /runs/:id/planned-orders` returns `{data,total,page,limit,totalPages}` (count + tenant/branch scoped) — matches MrpRunDetail.
- **Frontend (frontend/mrp)**: WorkCenters unwraps `{items,nextCursor}` envelope; capacityPerHour field contract; CapacityPlanning consumes capacity-planning weeks (0-demand weeks render 0%); MrpRun/MrpRunDetail paginated; Quality surface + metrics strip; WorkOrderDetail full rewrite (lifecycle buttons + block-reason prompt + complete-qty modal, materials issue/backflush, outputs, status-history timeline); WorkOrderBoard 10-column; 0 `var(--c-*)`/hex violations.
- **Verification**: `pnpm exec tsc --noEmit` 0 errors in sales-svc + operations-svc; canonical `pnpm run build` green (operations-svc); all 25 frontend packages pass `typecheck-frontend.mjs`; `check-token-usage.mjs` clean (1108 files).
- **Gaps closed (2026-08-03)**: NCR/CAPA/inspection-plans frontend pages (`NcrsList`/`NcrDetail`/`CapaList`/`InspectionPlansList` in `frontend/mrp/src/pages`) + routes/nav/commands; WO create form in `WorkOrdersList.tsx` (SKU→BOM by-product → routing select, posts `/api/mrp/work-orders`); CapacityCalendar exceptions wired to real backend (new `GET /capacity-calendars/:id/exceptions` in `capacityCalendars.ts` + frontend query/mutations); `lib/api-spec/openapi.yaml` extended with 143 MRP operations + reusable schemas + split shared request-body schemas (`TransitionNcrRequest`/`TransitionCapaRequest`/`UpdateCapacityCalendarRequest`/`UpdateEmailSignatureRequest`) to fix an orval shared-body emission bug → `pnpm run codegen` + both `@workspace/api-client-react` + `@workspace/api-zod` typecheck clean (0 errors), frontend/mrp tsc clean, token guard clean (1113 files).

### Next Steps
1. **ARB acceptance of ADR-006..020** — all remain PROPOSED except ADR-001..005 + ADR-018 (circuit breaker); tracker keeps these as ⚠️
2. **RLS freeze contradiction** — `enterprise-architecture-freeze.md` §5.1 says "No RLS — manual filtering" while ADR-011 (PROPOSED) + migrations `001_rls_tenant_isolation`/`003_rls_enforcement` introduce RLS; needs ARB decision to reconcile
3. **auth-service-go Kafka audit consumer stubbed** — `internal/kafka/audit_consumer.go` runs hash verify/DB write/DLQ routing but `realKafkaReader`/`realKafkaWriter` return nil (kafka-go client not implemented); wire a real client
4. **Event schema registry enforcement** — only `schemaVersion: "1.0"` envelope headers exist; no registry validates payload shape at produce/consume
5. **DLQ replay generalization** — `GET /internal/events/dlq` + `POST /:id/replay` (sets `replayedAt`) exist per module and finance-svc has `dlqReplayHandler`, but replay coverage is not uniform across all consumer topics
6. **135 pre-existing frontend vitest failures** — baseline failures remain unaddressed (outside freeze scope); being fixed in progress by a separate engineer
7. Build and deploy affected services to test widget data; fix dashboard CSS layout overlap (react-grid-layout `measureBeforeMount` or height fix)
8. Branch follow-ups: wire auth-service-go `RequireBranchAccess` at route level if auth-service is ever exposed directly; gate `GET /tenants/branches` for non-hierarchy roles; consider `branch_id` NOT NULL + branch picker for privileged creates
9. Phase 3 tracked items: R09–R12 relationship cardinality (one_to_one/one_to_many/many_to_one done; full polymorphism remaining), R13–R15 search relevance, R16–R17 distributed tx, R23 virus scanning, R29/R30 catalog automation, gateway body-validation middleware, PAR storage restriction, merkle anchor on internal audit path

### Remaining (Leads App)
- 100+ inline `style={{}}` instances (most are dynamic/one-off positioning, progress widths, conditional colors — acceptable to keep inline; candidates for future CSS migration if refactoring component structure)
- 1 custom class `filter-search-icon` remains (icon wrapper, no `op-*` equivalent)

---
> Source: [Opseron/Opseron](https://github.com/Opseron/Opseron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
