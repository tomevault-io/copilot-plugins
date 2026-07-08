## nexoveranda

> > Reference document for Codex sessions on this repo. Read this first before exploring or making changes.

# AGENTS.md — Nexo Veranda

> Reference document for Codex sessions on this repo. Read this first before exploring or making changes.
> Last updated: 2026-04-17 — backend-audit branch merged to main (16 fixes); §7 Known Issues re-verified.

---

## 1. Project Overview

**Nexo Veranda** is a digital ecosystem for an international veranda / outdoor-room manufacturer (client: Mehmet @ Nexo Veranda, NL). The product is a single React SPA + a Supabase backend, but functionally it is **four integrated modules** separated by URL prefix and per-module layout:

| Module | Prefix | Purpose | Status |
|---|---|---|---|
| **A. Webshop + 3D Configurator** | `/`, `/producten/*`, `/configurator`, `/:slug` | Public marketing site, product catalog, 3D wizard, hybrid checkout (direct buy vs. quote request), regional SEO landing pages, AI chatbot | ~95% |
| **B. Admin "Het Brein"** | `/admin/*` | Quotes, orders, invoices, customers, products, suppliers, planning, messaging hub (WhatsApp + email + tickets), Exact Online sync, audit log | ~92% |
| **C. Personeel (Staff)** | `/personeel/*` | Field service: workorders 2.0 (photos, signature, checklist), planning view, time entries, leave requests, expense claims, payslips, onboarding | ~92% |
| **D. Mijn Nexo (Customer Portal)** | `/mijn-account/*` | Order timeline, quotes, invoices, documents, service tickets, Nexo Academy (videos/guides), AI chatbot, push opt-in, notification center | ~92% |

**Core workflows the system supports today:**

1. **Quote → Order → Production → Install → Invoice.** Customer requests quote (anon insert into `quotes`) → admin processes in `/admin/quotes` → on acceptance converts to order → planner schedules install (`planning_items`) → staff completes work order → admin generates invoice (synced to Exact Online).
2. **Customer self-service.** Customer logs in, watches order timeline update, downloads documents (RLS-filtered), browses Academy, opens service tickets, chats with the AI bot.
3. **Staff field workflow.** Staff opens assigned workorder on phone (PWA), uploads photos, captures signature, completes checklist → completion auto-creates a `time_entries` record.

Tech: React 18 + TS + Vite + Tailwind/shadcn + Supabase. PWA-installable. 5 languages (NL/EN/DE/FR/ES). Built originally in Lovable; now being finished in Codex + VS Code.

---

## 2. Tech Stack & Architecture

### Exact versions (from `package.json`)

- **React** 18.3.1, **React-DOM** 18.3.1, **React Router** 6.30.1
- **TypeScript** 5.8.3
- **Vite** 5.4.19, **vite-plugin-pwa** 1.2.0
- **Tailwind CSS** 3.4.17, **shadcn/ui** (Radix primitives)
- **@supabase/supabase-js** 2.94.0
- **@tanstack/react-query** 5.83.0 (used in 52 files)
- **Zustand** 5.0.11 (only `useStickyBarStore` — UI state only)
- **react-hook-form** 7.61.1, **@hookform/resolvers** 3.10.0, **zod** 3.25.76
- **i18next** 25.8.4, **react-i18next** 16.5.4, `i18next-browser-languagedetector`
- **three** 0.170.0, **@react-three/fiber** 8.18.0, **@react-three/drei** 9.117.3
- **jspdf** 4.1.0 (invoice + quote PDFs)
- **framer-motion** 12.31.0, **recharts** 2.15.4, **sonner** 1.7.4, **@dnd-kit/\***
- Package manager: **bun** (`bun.lockb` present) — npm also works

### Folder map (`src/`)

```
src/
├── App.tsx                    # Route map (lines 101-197) — single source of truth for URLs
├── main.tsx                   # Entry; mounts QueryClientProvider, Router, i18n
├── pages/
│   ├── admin/                 # Module B — 37 pages (AdminDashboard, AdminQuotes…)
│   ├── staff/                 # Module C — 13 pages (StaffDashboard, StaffWorkOrders…)
│   ├── customer/              # Module D — 15 pages (CustomerDashboard, CustomerAcademy…)
│   ├── Index.tsx              # Module A — homepage
│   ├── ConfiguratorPage.tsx   # Module A — 3D wizard entry
│   ├── ProductsPage.tsx, ProductDetailPage.tsx, CityLandingPage.tsx, …
│   └── (public: Contact, OverOns, Showrooms, Privacy, …)
├── components/
│   ├── admin/                 # 20 admin-specific components (MarginDashboardWidget, OrderTimeline, …)
│   ├── staff/                 # CompleteWorkOrderDialog, …
│   ├── customer/              # Customer portal pieces
│   ├── configurator/          # Module A — Three.js scene + wizard steps
│   │   ├── Scene3DEnhanced.tsx, VerandaModelEnhanced.tsx (R3F; @ts-nocheck)
│   │   ├── scene/{materials,HouseWall,RoofPanels,GlassWalls,Gutter,LamellaRoof}.tsx
│   │   ├── wizard-steps/{StepDimensions,StepFrameColor,StepRoofType,StepGlassWalls,
│   │   │                  StepLighting,StepSunscreen,StepExtras,StepInstallation}.tsx
│   │   ├── ConfiguratorWizard.tsx, PriceSummaryEnhanced.tsx, PDFExport.tsx, CameraControls.tsx
│   │   └── Configurator.tsx   # ⚠ static demo on homepage — NOT wired
│   ├── chatbot/ChatWidget.tsx # ⚠ hardcoded Supabase URL/key (see §7)
│   ├── portfolio/, ui/        # 50+ shadcn components
│   └── (Header, Footer, Hero, Products, Testimonials, ShowroomSection, FAQ, …)
├── hooks/                     # 41 hooks; data-access layer (one per domain)
│   ├── useAuth.ts, useAdminAuth.ts, useStaffAuth.ts, useCustomerAuth.ts
│   ├── useOrders.ts, useQuotes.ts, useInvoices.ts, useCustomers.ts, useSuppliers.ts
│   ├── useStaffWorkOrders.ts, useStaffTimeEntries.ts, useStaffLeaveRequests.ts, useStaffExpenseClaims.ts
│   ├── useCustomerOrders.ts, useCustomerQuotes.ts, useCustomerNotifications.ts, …
│   ├── useEmails.ts, useWhatsAppMessages.ts, usePlanning.ts, useServiceTickets.ts
│   ├── useConfiguratorState.ts, useConfiguratorPricing.ts, usePriceMatrix.ts, usePriceOptions.ts
│   ├── useSupplierProducts.ts, usePushNotifications.ts, …
├── lib/
│   ├── auth-utils.ts          # Role → portal route mapping
│   ├── rbac.ts                # ROLE_PERMISSIONS matrix + canAccessRoute()
│   ├── api/                   # Thin Supabase query helpers
│   └── export-utils.ts
├── integrations/supabase/
│   ├── client.ts              # createClient() with localStorage + autoRefresh
│   └── types.ts               # GENERATED — do not edit by hand
├── i18n/
│   ├── index.ts               # i18next init (fallback nl, langs: nl/en/de/fr/es)
│   └── locales/{nl,en,de,fr,es}.json
├── stores/useStickyBarStore.ts # only Zustand store
└── assets/
```

### Module separation

- Each module = **route prefix + subdirectory + Layout component** (`AdminLayout`, `StaffLayout`, `CustomerLayout`).
- Auth isolation: separate login pages (`/admin/login`, `/personeel/login`, `/mijn-account/login`).
- Route guard: `src/lib/rbac.ts` `ROLE_PERMISSIONS` matrix → `canAccessRoute(role, path)`.
- Roles: `admin | sales | service | content_manager | staff | customer`.

### Route inventory (from `src/App.tsx` lines 101–197)

**Module A (public):** `/`, `/producten`, `/producten/:slug`, `/configurator`, `/installeren`, `/projecten`, `/over-ons`, `/contact`, `/algemene-voorwaarden`, `/privacy-beleid`, `/showrooms`, `/inloggen`, `/:slug` (city landing)

**Module B (`/admin/*`):** `login`, `set-password`, `reset-password`, `` (dashboard), `quotes`, `quotes/new`, `quotes/:id`, `quotes/:id/edit`, `orders`, `orders/new`, `orders/:id`, `orders/:id/edit`, `customers`, `customers/new`, `customers/:id`, `customers/:id/edit`, `invoices`, `invoices/new`, `invoices/:id`, `tickets`, `tickets/:id`, `products`, `products/:id/edit`, `suppliers`, `suppliers/new`, `suppliers/:id`, `categories`, `categories/:id`, `staff`, `planning`, `documents`, `messages`, `mail`, `whatsapp`, `users`, `import`, `academy`, `settings`, `audit-log`, `notifications`, `locations`

**Module C (`/personeel/*`):** `login`, `wachtwoord-vergeten`, `set-password`, `onboarding`, `` (dashboard), `werkbonnen`, `werkbonnen/:id`, `uren`, `verlof`, `declaraties`, `loonstroken`, `profiel`

**Module D (`/mijn-account/*`):** `login`, `register`, `reset-password`, `` (dashboard), `offertes`, `offertes/:id`, `orders`, `orders/:id`, `tickets`, `tickets/:id`, `facturen`, `documenten`, `academy`, `profiel`

### Data layer

- **TanStack React Query everywhere.** Query/mutation hooks live in `src/hooks/use*.ts` and wrap Supabase calls. Cache invalidation is per-hook (no global pattern yet — when in doubt, invalidate the queryKey of the hook you mutated).
- **Zustand** is intentionally minimal; only used for sticky-bar UI state. Do not introduce a global Zustand store for server state — use React Query.
- **No Context API** of consequence (Supabase client is imported directly; React Router provides its own context).
- **No Realtime subscriptions** anywhere yet (`supabase.channel(...)` is unused). Adding realtime would be a meaningful enhancement (e.g. live order timeline for customers).

### Supabase client

`src/integrations/supabase/client.ts` — single `createClient()`, persists session in localStorage, auto-refresh enabled. Anon key + project URL are embedded (acceptable for an anon/authenticated client). `types.ts` is generated from the schema and used by every hook for type-safety.

### PWA

- Configured in `vite.config.ts` via `vite-plugin-pwa` (registerType `autoUpdate`, workbox precache JS/CSS/images up to 5MB).
- Manifest is **inlined in the plugin config** (name "Nexo Veranda", theme `#2D5A3D`, display `standalone`, icons `pwa-192x192.png` + `pwa-512x512.png` maskable).
- Icons exist in `/public/`. ⚠ `index.html` references `manifest.webmanifest` as a separate file — that file is **missing**. Either remove the link tag or generate the file; vite-plugin-pwa already serves one.
- `usePushNotifications` hook exists; the `send-push` edge function is a stub. No subscription UI yet.

### i18n

- 5 locales: `nl` (primary, fallback), `en`, `de`, `fr`, `es`. JSON files at `src/i18n/locales/`.
- Browser language detection + localStorage persistence; `<html lang>` updated automatically.
- All UI strings use `const { t } = useTranslation()`; new code must follow.
- **Product content translation** is handled server-side by the `translate-product` edge function (DeepL API).
- The chatbot (`chat` edge function) accepts a `language` param and replies in the user's language.

---

## 3. Database Schema

- **40 tables**, **81 migrations** under `supabase/migrations/` (timestamps 2025–2026). Migrations are the source of truth.
- **Project ID:** `uzjzkhigkkdqtnwcqhbk`.

### Tables (grouped)

**Quotes & Orders**
`quotes`, `orders`, `order_items`, `order_documents`, `order_timeline`, `price_matrix`, `price_options`

**CRM & Communication**
`customers`, `communications`, `contact_messages`, `service_tickets`, `ticket_messages`, `service_ticket_attachments`, `showroom_appointments`

**Products & Suppliers**
`product_categories`, `supplier_products`, `suppliers` (seeded with EG Aluminium catalogue)

**Staff / HR**
`staff_profiles`, `staff_documents`, `work_orders`, `leave_requests`, `planning_items`, `time_entries`, `expense_claims`

**Accounting**
`invoices`, `invoice_lines`, `exact_tokens` (Exact Online OAuth tokens)

**Email & WhatsApp**
`emails`, `email_attachments`, `whatsapp_messages`, `whatsapp_config`

**Portal & system**
`academy_items`, `admin_settings`, `notifications`, `customer_notifications`, `customer_documents`, `push_subscriptions`, `locations`, `audit_logs`, `configurations`, `user_roles`

### Enums

- `app_role` — `admin | user` (additional roles applied via `user_roles` rows + the RBAC matrix in code)
- `quote_status` — `nieuw | in_behandeling | offerte_verstuurd | geaccepteerd | afgewezen`
- `order_status` — 9 states from `concept` to `afgerond`

### RLS

- **27/40 tables have RLS enabled.** The other 13 (mostly read-only catalog/config tables) should be audited — assume "missing RLS" until verified.
- Public anon **INSERT** is allowed on `quotes` (anon quote-request flow) and `contact_messages`.
- Public **SELECT** on `product_categories` and `supplier_products` (when `is_published = true`).
- Most other tables: admin-only reads/writes via `has_role()`. Customers see their own rows via email-join policies (e.g. `customer_documents`).

### DB functions & triggers

- 16 functions: `has_role()` (SECURITY DEFINER, used by RLS to avoid recursion), number generators (`generate_quote_number`, `generate_order_number`, `generate_invoice_number`, `generate_ticket_number` — format `NEXO-YYYY-NNN`), `update_updated_at_column`, plus notification helpers (`notify_customer_email`, `notify_push`, `trigger_email_order_status`, `trigger_push_order_status`, `trigger_push_new_quote`, …).
- 28 triggers: `updated_at` on 14 tables, number-generation on insert (quotes/orders/invoices/tickets), push notifications on order status changes and new quotes.

### Storage buckets

| Bucket | Public? | Purpose | Policy |
|---|---|---|---|
| `product-images` | yes (read) | Product photos | Auth required for write; public read |
| `order-documents` | no | Workorder photos, drawings | Admin full access |
| `customer-documents` | no | Invoices, manuals exposed to customer portal | Admin manage; customers SELECT own via email join |
| `email-attachments` | no | Inbound/outbound mail | Admin only |
| `whatsapp-media` | implicit | WhatsApp message media | Service role (edge function only) |

### Seed data

- Product categories: Veranda's, Lamellendaken, Glazen Schuifwanden, Zonwering, Screens, Kozijnen, Schuttingen.
- 14+ EG Aluminium supplier products with JSONB specifications.
- No user/customer seed data.

---

## 4. Edge Functions & Integrations

19 edge functions under `supabase/functions/`. **All have `verify_jwt = false` in `supabase/config.toml`** — auth is enforced manually inside each function. This is a security audit item (§7).

| Function | Purpose | Auth | Secrets | Status |
|---|---|---|---|---|
| `whatsapp-send` | Send WhatsApp via Meta Graph v25 | Admin token check | `SUPABASE_SERVICE_ROLE_KEY`, Meta token | Complete |
| `whatsapp-webhook` | Inbound Meta webhooks (messages, status, media → `whatsapp-media` bucket) | Webhook signature | — | Complete |
| `whatsapp-config` / `whatsapp-register` | Business-account setup | Admin | Service role | Complete |
| `send-mail` | Outbound SMTP (TransIP) | Admin | `MAIL_SMTP_HOST`, `MAIL_USER`, `MAIL_PASSWORD` | Complete |
| `fetch-mail` | IMAP polling (TransIP) | Admin | `MAIL_*` | Complete |
| `send-notification-email` | Transactional email | JWT check | `MAIL_*` | Partial — verify wiring |
| `chat` | Customer AI chatbot (streaming SSE, multi-lang) | Public | `LOVABLE_API_KEY` (Gemini 3 flash) | Complete |
| `translate-product` | Auto-translate product content NL→EN/DE/FR/ES | Admin/service-role | `DEEPL_API_KEY`, optional `DEEPL_API_BASE_URL`, optional `DEEPL_GLOSSARY_*` | Complete |
| `optimize-product` | SEO rewrite of product copy | Public | `LOVABLE_API_KEY` | Complete |
| `exact-register` | Exact Online OAuth code → token exchange | Admin | Service role | Complete |
| `exact-invoices` | Push invoices to Exact | Admin | Tokens from `exact_tokens` | Complete |
| `exact-webhook` | Exact callbacks | Webhook | — | Complete |
| `exact-config` | Connection settings | Admin | — | Complete (URL hardcoded — verify) |
| `firecrawl-scrape` / `firecrawl-map` | Web scraping for product data | Public | `FIRECRAWL_API_KEY` | Complete |
| `invite-user` | Create admin user (sends invite email) | Admin | Service role | Complete |
| `admin-users` | Admin user mgmt (disable, role change) | Admin | Service role | Complete |
| `send-push` | Web push to subscribers | Admin | — | **Stub** |

### Integration map

**SiteJob Connect bridge**: Exact Online én WhatsApp Business auth lopen via een centrale SiteJob Supabase-project (`xeshjkznwdrxjjhbpisn.supabase.co`). Die hub beheert OAuth-flows en webhook-routing voor alle SiteJob klanten. Nexo's edge functions (`exact-*`, `whatsapp-*`) halen een access-token op bij Connect en praten vervolgens **direct** met de upstream API's (`start.exactonline.nl/api/v1` resp. `graph.facebook.com/v25.0`). De hardcoded URL naar Connect in de edge functions is dus intentional, geen misconfiguratie.

| Integration | Status | Where |
|---|---|---|
| WhatsApp Business (Meta Graph v25) | ✅ Complete | `whatsapp-*` functions → Connect bridge → Meta Graph API |
| Exact Online (accounting) | ✅ Complete | `exact-*` functions → Connect bridge → Exact Online REST API |
| Email (TransIP SMTP + IMAP) | ✅ Complete | `send-mail`, `fetch-mail`, `send-notification-email`, `/admin/mail`, `/admin/messages` |
| Lovable AI Gateway (Gemini 3 flash) | ✅ Complete | `chat`, `optimize-product` |
| DeepL API | ✅ Complete | `translate-product` |
| Firecrawl | ✅ Complete | `firecrawl-*` (product scraping during onboarding) |
| Web push notifications | ⚠ Partial — `usePushNotifications` hook + `push_subscriptions` table + `send-push` stub; no subscription UI | |
| Google Calendar | ❌ Not started | — |
| Payments (Stripe / Mollie) | ❌ Not started | — |
| Supplier auto-PO | ❌ Not started — UI for suppliers exists, no purchase-order generation | — |
| Route optimization (Module C) | ❌ Not started | — |

---

## 5. Key Patterns & Conventions

### Naming

- Files: `PascalCase.tsx` for components/pages, `useThing.ts` for hooks, `kebab-case.tsx` only inside `components/ui/` (shadcn convention).
- Database: `snake_case` tables and columns. ID fields are `uuid` defaulting to `gen_random_uuid()`. Foreign keys named `<table>_id`.
- Routes: Dutch (`/offertes`, `/werkbonnen`, `/mijn-account/facturen`).
- Number formats: `NEXO-YYYY-NNN` for quotes/orders/invoices/tickets (auto-generated by triggers).

### Data fetching

- Always go through a hook in `src/hooks/`. Don't call `supabase` directly from a component.
- Mutations should call `queryClient.invalidateQueries({ queryKey: [...] })` for the affected list.
- Errors surface via Sonner toasts (`toast.error(...)`).

### Forms

```tsx
const form = useForm<FormValues>({
  resolver: zodResolver(schema),
  defaultValues: {...},
});
// Use shadcn <Form>/<FormField>/<FormItem>/<FormControl>/<FormMessage>
```

Schemas live next to the form component. Reuse them between create/edit forms.

### Auth & roles

1. `useAuth()` listens to Supabase `onAuthStateChange`.
2. Role is fetched from `user_roles` via `getUserRole()`.
3. `lib/rbac.ts` `canAccessRoute(role, path)` is the source of truth — update it when adding routes.
4. After login, `auth-utils.ts` `getPortalRoute(role)` redirects: admin/sales/service/content_manager → `/admin`, staff → `/personeel`, customer → `/mijn-account`.

### PDFs

- Invoices: `src/components/admin/InvoiceLocalPdf.ts` (jspdf, NL date-fns locale, called from `AdminInvoiceForm`).
- Quote screenshot from configurator: `src/components/configurator/PDFExport.tsx`.

### Error handling

- Toasts (Sonner) for user-visible errors.
- **No global ErrorBoundary** — add one at the App root before production.
- Edge functions return JSON with `{ error: string }` on failure; hooks should surface `error.message` in toasts.

---

## 6. Module-by-module Status

### Module A — Webshop + 3D Configurator (~95%)

**✅ Working**
- Product catalog (`ProductsPage`, `ProductDetailPage`) with category filters, search, multi-lang names.
- City landing pages: `/:slug` → `CityLandingPage`, content from `data/cityLandingPages.ts`, schema.org structured data.
- 3D configurator (`ConfiguratorPage` → `ConfiguratorWizard` → `Scene3DEnhanced` + `VerandaModelEnhanced`) — real R3F scene, 8 wizard steps, camera presets, lamella animation, night mode, fullscreen, screenshot capture.
- Live pricing via `useConfiguratorPricing` reading `price_matrix` + `price_options` tables.
- Hybrid checkout: `QuoteRequestModal` inserts into `quotes` (anon RLS allowed). Direct buy not yet a separate flow — quote covers both.
- 5-language UI fully wired.
- Quote PDF export via `PDFExport.tsx`.

**⚠ UI-only / stale**
- `src/components/configurator/Configurator.tsx` (used on the homepage) is a **hardcoded demo** with static types/colors/prices (lines 12–23). Either wire it to live data or remove it; do not extend it.
- After a quote is requested there is **no automatic order creation** — admin manually converts via `AdminOrderForm`.

**❌ Missing**
- Multi-currency display (EUR only).
- Inventory / stock checks.
- Regional shipping zones (price matrix has no zone logic).
- `hreflang` tags for international SEO.
- Standalone `manifest.webmanifest` file (referenced in `index.html`, served by plugin instead).

---

### Module B — Admin "Het Brein" (~85%)

**✅ Working**
- Full CRUD: `AdminQuotes`, `AdminOrders`, `AdminInvoices`, `AdminCustomers`, `AdminProducts`, `AdminSuppliers`, `AdminCategories`, `AdminStaff`, `AdminLocations`.
- Margin: `MarginDashboardWidget` + `MarginIndicator` + per-line margin in `AdminInvoiceForm`/`AdminOrderForm` (cost vs sell from supplier_products).
- Quote pipeline with status badges; quote → order conversion in `AdminOrderForm`.
- Weekly planner `AdminPlanning.tsx` — week navigation, staff roster grid, planning_items with type (`installatie | inmeting | onderhoud | overig`), color coding, leave-day blocking. **Uses null coalescing — no "undefined" rendering bug found in current code.** If the demo screenshots showing "UNDEFINED" are still relevant, verify with fresh seed data; the issue is likely fixed.
- WhatsApp inbox `AdminWhatsApp` (real Meta integration via edge functions).
- Email inbox `AdminMail` (IMAP via `fetch-mail`) + `AdminMessages` for replies.
- Service tickets `AdminTickets` / `AdminTicketDetail`.
- Invoice → Exact Online sync via `exact-invoices` edge function.
- Audit log `AdminAuditLog`.
- Push trigger on order status change (DB trigger `trigger_push_order_status`); push trigger on new quote (`trigger_push_new_quote`).
- Bulk product import `AdminProductImport` (CSV).

**⚠ UI-only / partial**
- Exact Online: token refresh path needs end-to-end QA.
- No SLA / overdue alerts.
- No batch invoice generation.
- `AdminInvoiceForm` does not surface Exact API errors clearly.

**❌ Missing automations**
- 48-hour-before-install checklist (no scheduled job; would need `pg_cron` or external scheduler).
- Workorder-completed → automatic final invoice generation (currently manual).
- Supplier purchase-order auto-creation when an order is confirmed.
- Multi-location order routing (`AdminLocations` exists but no assignment logic).

---

### Module C — Personeel / Staff (~85%)

**✅ Working**
- `StaffDashboard` — KPIs (open workorders, weekly hours, pending leave, expense claims), weekly planning grid (filtered by `staff_user_id`), leave blocking.
- Workorders 2.0 (`StaffWorkOrders` / `StaffWorkOrderDetail`):
  - Status transitions `concept → in_uitvoering → afgerond → goedgekeurd`.
  - Photo upload to `order-documents` bucket.
  - Customer signature stored in `work_orders.customer_signature`.
  - Checklist + completion dialog (`CompleteWorkOrderDialog`).
  - **`completeWorkOrder` mutation auto-creates a `time_entries` row** — single source of truth for hours.
- HR: `StaffLeaveRequests` (approval workflow), `StaffTimeEntries` (regulier/overwerk/verlof/ziek), `StaffExpenseClaims` (voeding/kilometer/overig with photo of receipt), `StaffPayslips` (read-only, computed from `time_entries` + `expense_claims`), `StaffOnboarding` (legal/safety/contract acknowledgment).
- `StaffProfile` with password change.

**⚠ UI-only / partial**
- Workorder photo gallery has no preview UI in the detail view (lists filenames only).
- Signature capture: dialog exists but the canvas/library used should be verified before going live.
- `StaffWorkOrderDetail` does not null-guard `workOrder` before firing `completeWorkOrder` (line ~98).

**❌ Missing**
- Route optimization (0% — no Google Routes / TSP).
- Offline workorder editing (app is online-only).
- GPS / live location tracking.
- Fuel cost integration (expenses are manual).

---

### Module D — Mijn Nexo / Customer Portal (~85%)

**✅ Working**
- Auth: `CustomerLogin`, `CustomerRegister`, `CustomerResetPassword`, password change in `CustomerProfile`.
- `CustomerDashboard` — overview cards.
- Orders: `CustomerOrders` + `CustomerOrderDetail` with `OrderTimeline` showing progression, order_items.
- Quotes: `CustomerQuotes` + `CustomerQuoteDetail`.
- Invoices: `CustomerInvoices` (linked to orders).
- Documents: `CustomerDocuments` filtered per customer via email-join RLS on `customer_documents`.
- Service tickets: `CustomerTickets` + `CustomerTicketDetail` with admin replies and file attachments (`service_ticket_attachments`).
- **Nexo Academy** (`CustomerAcademy`): videos (iframe + local + thumbnail fallback), guides (markdown + PDF), documents (PDF). Sorted by `sort_order`, filtered by `is_published`. Icons mapped per guide.
- AI chatbot (`ChatWidget`): streaming SSE via `chat` edge function, multi-lang, hidden on `/admin`, `/personeel`, `/mijn-account`, `/inloggen` via `PublicOnly` wrapper.
- `useCustomerNotifications` hook exists but no notification-center component yet.

**⚠ UI-only / partial**
- Install-appointment view shows dates from `OrderTimeline` but **no customer-side rebook UI** (admin-controlled).
- No notification-center component for the customer.

**❌ Missing**
- Warranty coverage display / claim submission.
- Self-service service-request form (tickets can only be created by admin or directly in DB).
- Realtime order tracking (no Supabase Realtime configured anywhere).
- Push-subscription opt-in UI (table + hook + edge stub exist).

---

## 7. Known Issues & Tech Debt

**Security — open**
- 🔴 **3.1 HMAC webhook verification** still pending — `whatsapp-webhook` can verify Meta's `X-Hub-Signature-256` directly, but `exact-webhook` and `exact-config`/`whatsapp-config` rely on a plaintext `X-Webhook-Secret` from SiteJob Connect until Connect can sign payloads. Status: blocked on Connect-eigenaar.
- 🟡 Backend-audit branch added additive `user_id`-RLS on `customers` (commit `b759923`); legacy email-join policies are still active for safe rollout. Drop them in a follow-up migration once the customer portal is verified end-to-end on production.

**Security — closed (verify if you see contradicting info)**
- ✅ Hardcoded Supabase URL/key in components — `ChatWidget.tsx` and `CustomerAcademy.tsx` both use the centralized `client.ts` (URL/key live there as named exports `SUPABASE_URL` + `SUPABASE_PUBLISHABLE_KEY`).
- ✅ Edge function `verify_jwt` audit done — `config.toml` documents the pattern (admin-only `true`, webhooks/triggers `false` with explicit `requireServiceRole`/`X-Webhook-Secret` checks).
- ✅ Edge function error sanitization — `_shared/errors.ts` `safeErrorResponse` helper used everywhere (commit `708d5d4`).
- ✅ `requireAdmin` deduplicated — all admin functions use `_shared/auth.ts` (commit `c3863ce`).
- ✅ All 41 tables have RLS enabled (AGENTS.md historisch zei 27/40 — onjuist).
- ✅ Notification deduplication via `notification_log` + `try_claim_notification` helper (commit `abf78dd`).
- ✅ Rate limiting on `chat` (60/min/IP) and `firecrawl-*` (10/min/user) via `rate_limit_check` RPC + `_shared/rate-limit.ts` (commit `997be2b`).

**Code quality — known living issues**
- `@ts-nocheck` in `Scene3DEnhanced.tsx` and `VerandaModelEnhanced.tsx` — intentional for R3F typing limits; leave alone unless replacing the lib.
- `undefined as any` casts in `src/components/configurator/PriceSummaryEnhanced.tsx` — type the pricing model properly.
- ✅ Homepage Configurator demo gone — replaced by `ConfiguratorCTA.tsx` (used in `src/pages/Index.tsx:91`).
- ✅ Global `ErrorBoundary` wraps `<App />` in `src/main.tsx` (component at `src/components/ErrorBoundary.tsx`).
- ✅ Stale `<link rel="manifest">` removed from `index.html`; vite-plugin-pwa injects manifest at build time.

**Realtime / UX**
- No `supabase.channel(...)` subscriptions anywhere — order timelines, planning, chat are poll-only. Adding realtime would meaningfully improve the customer-portal "order timeline" experience.
- No optimistic updates in mutations (low priority).

**Missing automations** (carried from Module B)
- 48h-before-install reminder
- Workorder-done → auto-invoice
- Supplier auto-PO

**Audit follow-ups still open** (van branch `feat/backend-audit-fixes`, gemerged in main)
- 4.2 anon quote → customer reconciliation trigger
- 4.5 WhatsApp/email orphan relinking UI

**AGENTS.md inaccuracies discovered during audit (corrigeer ze als je ze tegenkomt)**
- Tabelnaam `service_ticket_attachments` heet werkelijk `ticket_attachments`, FK is `ticket_id` (niet `service_ticket_id`).
- `audit_logs.details` heet werkelijk `metadata`.
- `time_entries` kolom heet `type` (niet `entry_type`); enum `time_entry_status` is `(ingediend, goedgekeurd, afgewezen)` zonder `geannuleerd`.
- `expense_claims` kolom heet `category` (niet `type`).

---

## 8. Development Setup

### Prerequisites
- Node 18+ (or Bun) — `bun.lockb` is committed; `npm` also works.
- Supabase CLI (only needed if editing migrations or deploying functions).

### Run locally
```bash
bun install        # or: npm install
bun run dev        # or: npm run dev   → http://localhost:8080
bun run build      # or: npm run build → ./dist
```

### Environment

- **Frontend**: Supabase URL + anon key are embedded in `src/integrations/supabase/client.ts`. No `.env` is required to boot the dev server.
- **Edge function secrets** (set via Supabase dashboard → Edge Functions → Secrets):
  - `SUPABASE_SERVICE_ROLE_KEY`
  - `MAIL_SMTP_HOST`, `MAIL_SMTP_PORT`, `MAIL_USER`, `MAIL_PASSWORD` (TransIP)
  - `MAIL_IMAP_HOST`, `MAIL_IMAP_PORT` (fetch-mail, default smtp/imap.transip.email)
  - `LOVABLE_API_KEY` (Lovable AI gateway / Gemini — chat, optimize-product)
  - `DEEPL_API_KEY` (DeepL product translations via `translate-product`)
  - `DEEPL_API_BASE_URL` (optioneel; defaults naar `https://api-free.deepl.com`, gebruik `https://api.deepl.com` voor Pro)
  - `DEEPL_GLOSSARY_EN`, `DEEPL_GLOSSARY_DE`, `DEEPL_GLOSSARY_FR`, `DEEPL_GLOSSARY_ES` (optioneel; DeepL glossary IDs per doeltaal)
  - `FIRECRAWL_API_KEY` (firecrawl-scrape, firecrawl-map)
  - `RESEND_API_KEY` (send-notification-email — transactionele klant-mails)
  - `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY` (send-push — web push)
  - `CONNECT_API_KEY` (SiteJob Connect bridge)
  - `PORTAL_BASE_URL` (optioneel; defaults naar `https://nexoveranda.com/mijn-account` voor e-mail links)
  - Meta WhatsApp: `META_WHATSAPP_TOKEN`, `META_WHATSAPP_PHONE_NUMBER_ID`, `META_WHATSAPP_VERIFY_TOKEN`
  - Exact Online: client id/secret/redirect URI used in `exact-register`

### Deploy
- Frontend → **Netlify** (demo: https://nexo-veranda.netlify.app/)
- Edge functions → `supabase functions deploy <name>` (per function)
- Migrations → `supabase db push` (against project `uzjzkhigkkdqtnwcqhbk`)

### Supabase
- Project ID: **`uzjzkhigkkdqtnwcqhbk`**
- Use the Supabase MCP server in Codex (`mcp__claude_ai_Supabase__*`) for read-only schema inspection and log queries.

---

## 9. Team & Contact

- **Developer:** Kas — kas@sitejob.nl (SiteJob)
- **Client:** Mehmet, Nexo Veranda (NL, internationally active)
- **Repo:** https://github.com/sitejob-nl/nexoveranda
- **Demo:** https://nexo-veranda.netlify.app/
- **Supabase project:** `uzjzkhigkkdqtnwcqhbk`

**Commercial:** setup fee €3.950 (50/50 betaling); maandelijks €299 (hosting + AI + updates + doorontwikkeling). Eerste maand inbegrepen.

---

## 10. Critical Files (read these first)

| Concern | File |
|---|---|
| Routing | `src/App.tsx` |
| Permissions | `src/lib/rbac.ts`, `src/lib/auth-utils.ts` |
| Supabase client + types | `src/integrations/supabase/client.ts`, `types.ts` |
| i18n | `src/i18n/index.ts`, `src/i18n/locales/*.json` |
| 3D configurator | `src/components/configurator/*` |
| Data hooks (41) | `src/hooks/use*.ts` |
| Schema | `supabase/migrations/` (81 files) |
| Edge functions | `supabase/functions/*/index.ts` |
| Function settings | `supabase/config.toml` (admin-only functies verify_jwt=true; webhooks + triggers false) |
| Edge function auth helper | `supabase/functions/_shared/auth.ts` (`requireAdmin`, `requireServiceRole`) |
| Vite + PWA + path aliases | `vite.config.ts` |
| PWA assets | `public/pwa-*.png` |

---

## Working with this repo: do's and don'ts

**Do**
- Keep new data access in `src/hooks/use*.ts`. Use TanStack Query.
- Use shadcn + Tailwind for new UI; mirror the existing per-module folder pattern.
- Wrap forms with react-hook-form + zod + shadcn `<Form>`.
- Use `t('...')` for every visible string.
- Add new routes to `src/lib/rbac.ts` so route guards know about them.
- Regenerate `supabase/types.ts` (`supabase gen types typescript`) after schema changes.

**Don't**
- Don't import `@supabase/supabase-js` and create a new client — use the shared `client.ts`.
- Don't put Supabase URL/key into components (see §7).
- Don't introduce a global Zustand/Redux store for server state.
- Don't add `verify_jwt = false` to new edge functions without explicit reason — gebruik `_shared/auth.ts` helpers en documenteer de auth-vorm in `config.toml`.

---
> Source: [sitejob-nl/nexoveranda](https://github.com/sitejob-nl/nexoveranda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
