## lumen

> **What**: A B2B SaaS billing system to support complex pricing models. Startup of 2 people focussed on shipping fast.

# Lumen Development Guide

## 1. Project Overview

**What**: A B2B SaaS billing system to support complex pricing models. Startup of 2 people focussed on shipping fast.

**Architecture**: Monorepo (`api`, `ui`, `common`) with Hono/TypeScript API, Next.js UI, Drizzle ORM, PostgreSQL

**Commands**: `bun run dev`, `dev:api`, `dev:ui`, `docker:up`, `docker:down`, `lint`, `format`

## 2. Development Guidelines

### 2.1 Code Style

- TypeScript with strict typing and explicit return types
- camelCase for variables/functions, PascalCase for components
- Double quotes for strings, required semicolons
- Group imports (external, internal, types) with empty line between groups
- Use path aliases: `@/common/...` not `@lumen/common/...`

### 2.2 Security Patterns

- **Critical**: Always use authenticated merchantId from context for access control
- Pattern: `const merchantId = c.get("merchant")?.id; if (!merchantId) return c.json({ error: "Unauthorized" }, 401);`
- Never allow client-provided merchantIds to override auth context
- Merchants must only access their own data; all multi-tenant queries must include merchantId filter

## 3. Core Subsystems

### 3.1 API Security

- Authentication: Session-based (UI), API keys (external), Trigger.dev tokens (jobs)
- API Key Types: Secret (`sk_{env}_{random}`), Publishable (`pk_{env}_{random}`)
- Secret keys are hashed (SHA-256); publishable keys stored as plaintext
- Redis-based rate limiting with sliding window

### 3.2 Plan Versioning

- Append-only: New versions created instead of modifying existing ones
- Existing customers remain on current plan version until explicitly migrated
- Version Intent: Clean audit trail, not multiple active plan variations

### 3.2.1 Plan Creation API

- **Comprehensive Creation**: Plans can be created with metrics, features, and prices in a single transaction
- **New Features**: Can create usage-based features with automatic metric creation and linkage
- **Existing Features**: Can attach existing features to new plans via featureIds
- **Pricing**: Supports monthly and yearly pricing creation during plan creation
- **Transaction Safety**: All operations (metrics, features, plan, prices) happen in a single DB transaction

### 3.3 Feature System

- Feature types: boolean (toggles), numeric (limits), string (settings)
- Many-to-many relationships: features-plans, features-subscriptions
- Denormalized fields for performance (merchantId, customerId, featureSlug)
- Priority: Subscription override → Plan feature → Default value

### 3.4 Subscription System

- Version-based with `stableSubscriptionId` across versions
- Previous versions soft-deleted with `deletedAt` timestamps
- Features inherited from plans with override capability

### 3.5 Billing System

- Price DSL with fixed, per-unit, and usage components
- Recurrence Rules: RFC5545 RRULE format (e.g., `RRULE:FREQ=MONTHLY;BYMONTHDAY=1`)
- Supports fixed-day and anniversary billing with DST awareness
- Smart handling for month-end dates (29-31) and leap years
- PDF generation via React-PDF, stored in S3 with presigned URLs
- Jobs pipeline via Trigger.dev: check billing due → generate invoices → create PDFs
- Multiple Price Components: Support for multiple components in a price with different billing cycles
  - Types: fixed, per_unit, and usage
  - Each component can have its own recurrence rule (monthly, weekly, daily)
  - Components are billed independently based on their own billing cycles
  - Each component generates its own subscription details and invoices

#### 3.5.0 Dunning & Grace Period System

- **Grace Period Management**: Subscriptions enter `past_due` status when payment fails or payment link is created
- **Automatic Flow (Pricing Table)**: Payment failures trigger immediate dunning via webhook → subscription marked `past_due` with grace period
- **Manual Flow (Admin-Created)**: Payment link generation automatically marks subscription `past_due` with grace period
- **Grace Period Timeline**:
  - Day 0: Payment fails or link sent (invoice email already delivered)
  - Day 6 (T-24h): "Grace ending soon" reminder email sent automatically
  - Day 7: Grace expires, configured action enforced (pause/cancel/mark_as_unpaid)
- **Dunning Configuration**: Hierarchical (plan-specific → merchant default → system default)
- **Recovery**: Successful payment automatically resumes subscription and resets billing cycle
- **Jobs Pipeline**:
  - `retryDunningPayments` (daily at 10 AM UTC): Retry failed payments
  - `notifyGraceEndingSoon` (hourly): Send T-24h reminder emails
  - `applyScheduledChanges` (every minute): Enforce expired grace periods

#### 3.5.1 Taxation Scope (B2B vs B2C)

- Current scope: B2B only. We require business identifiers to compute tax (e.g. VAT ID or business name).
- If a customer lacks business identifiers (no `taxId` and no `businessName`), we treat the transaction as B2C and return zero tax.
- Invoices will include a zero-tax line (when applicable) with reason code "B2C Not Supported" for auditability.
- Reason code added: `b2c_tax_not_supported` (code 21) to clearly flag B2C cases.
- First-time and trial-to-paid flows store tax metadata in payment/invoice; renewals re-calculate taxes using the engine.

#### 3.5.1.1 Tax Setup UI (Onboarding Wizard)

- The `/tax` page shows a guided wizard until tax is configured.
- Configuration is considered complete when either:
  - `merchant_tax_config.auto_tax_calculation = true`, or
  - `merchant_tax_config.default_tax_rate` is set (including `0` for 0%).
- Completion signal comes from `GET /merchants/checkonboarding` (`hasTax`).
- Wizard flow:
  - Step 1: Audience selection (B2B vs B2B+B2C)
  - Step 2: Method selection
    - Automatic (B2B only; consumers 0% by default; explicit acknowledgement required)
    - Single default rate (enter %)
    - Advanced manual rates (marks default rate as 0% to complete onboarding; rates can be created and assigned later)
  - Step 3: Review & confirm → saves via `POST /tax/config/update`.
- For US merchants choosing Automatic, the page recommends configuring state nexus after saving.

### 3.5.2 Seat-Based Pricing & Mid-Cycle Billing

- **Seat Management**: API endpoints for adding/removing seats with event tracking
- **Mid-Cycle Charging**: Prorated billing for seat additions with immediate charging support
- **Overage Billing**: Automatic invoice generation when seat usage exceeds plan limits
- **Usage Tracking**: Real-time seat usage calculation via QuestDB event aggregation
- **Proration Logic**: Time-based proration for partial billing periods using `prorateAmountCents()`
- **Event Verification**: Validation of active seats before removal using event history

### 3.6 Events and Metrics System

- **Dual Storage**: QuestDB (primary) with PostgreSQL backup for high-volume events
- **Event Ingestion**: RESTful API with idempotency via Redis
- **Event Structure**: Flexible schema supporting numeric and string values with custom properties
- **Metrics**: Three types supported:
  - `simple`: Basic aggregations (count, sum, avg, min, max) with filtering
  - `indexed`: SQL-based metrics with optimized access patterns
  - `complex`: Advanced SQL metrics with full flexibility
- **QuestDB**: Time-series optimized with TIMESTAMP(event_timestamp) PARTITION BY DAY
- **Batch Processing**: Events batched for PostgreSQL persistence via BullMQ
- **Redis Caching**: Metric definitions cached for fast lookup and execution
- **SQL Validation**: Security measures for custom SQL metrics (AST validation)
- **Merchant Isolation**: All metrics/events scoped to merchantId for security

### 3.7 Credit System

- **Credit Types**: Usage credits (for entitlements) and monetary credits (for billing discounts)
- **Application Scopes**: Component-level, price-level, plan-level, and merchant-level with priority hierarchy
- **Credit Definitions**: Template system for creating reusable credit configurations
- **Credit Grants**: Individual allocations to customers with denormalized data for performance
- **Usage Tracking**: Comprehensive audit trail via `creditUsageTransactions` table
- **Renewal System**: Supports both billing-triggered and RRULE-based scheduled renewals
- **Background Jobs**: Automated expiration (hourly) and renewal (hourly) processing via Trigger.dev
- **Billing Integration**: Multi-stage credit application during invoice generation with line item visibility
- **Entitlement Integration**: Usage credits offset usage against subscription limits for access control

## 4. Known Issues

- **drizzle-zod**: JSONB fields require explicit typing: `jsonb('field').$type<{ key: string }>()`
- **Billing Period Display**: Awaiting upstream change to provide period dates during invoice creation
- **Metrics Testing Gap**: Missing integration tests for SQL-based metrics (indexed and complex types). Current tests only cover simple metric types.

### 3.8 Pricing Tables System

- **Public Access**: Publishable API key-based access for frontend integration
- **Slug-based Routing**: SEO-friendly URLs for pricing table access
- **Plan Association**: Many-to-many relationship between pricing tables and plans
- **Display Configuration**: Configurable ordering, popular badges, button styling
- **Plan Transformation**: Automatic formatting for frontend PricingTable components
- **Validation**: Plan count validation and merchant isolation enforcement

### 3.9 Payment Service Providers

- **Multi-PSP Support**: Stripe (primary) and Dodo Payments integration
- **Dodo Payments**: Full PSP integration with encrypted credential storage
  - Environment detection (test/live) via API validation
  - Webhook secret management for secure event processing
  - Configuration management with masked key display
  - Validation endpoints for API key verification
  - Automatic webhook provisioning: When saving a valid Dodo API key, the system attempts to auto-create a webhook endpoint in Dodo using the authenticated API client and stores the returned signing secret securely. The webhook URL used is `${API_URL}/v1/public/webhook-dodo/{merchantId}`. If auto-creation fails, the save still succeeds and the webhook can be configured later.
  - Stripe webhook auto-healing (production): On webhook signature verification failure for an active merchant, the system acquires a short Redis lock and automatically deletes any existing webhook endpoint for the merchant URL, recreates a fresh endpoint, and rotates/persists the new signing secret. Skips in development (Stripe CLI flow) and rate-limits via lock cooldown.

### 3.9.1 Payment Method Selection & Failover (Subscriptions)

- For subscription renewals and subscription-related charges, the system now prefers the payment method configured on the subscription (`subscription_details.payment_method_id`).
- If that is not set, we fall back to the customer's most recently used active payment method (`payment_methods.last_used_at` DESC).
- On payment failure webhooks for recurring charges, we immediately attempt charging the next available customer payment method (excluding ones already tried for that payment). We track attempted payment method IDs on the `payments.metadata.attempted_payment_method_ids` field.
- We continue retrying one alternative method per failure webhook until all methods are exhausted; only then do we enter the dunning flow. This preserves existing dunning behavior and grace-period handling once failover options are depleted.

### 3.10 Trial & Free Plan System

- **Trial Plans**: Dedicated trial plan creation with parent plan relationships
- **Trial Metadata**: `isTrialPlan`, `trialParentPlanId`, and `trialDays` fields in plan schema
- **Free Plan Detection**: Automatic identification of $0 plans with no usage costs
- **Free Subscription Creation**: Streamlined signup flow for free plans without payment methods
- **Trial-to-Paid Conversion**: Seamless upgrade path from trial to parent plan

### 3.11 Teams & Invitations

- Policy: One user belongs to exactly one organization; organization has exactly one merchant.
- Invite flow:
  - Logged-out users are redirected to `/login` with invite context preserved.
  - Logged-in users see a confirmation screen on `/accept-invite` outlining impact before joining.
  - On confirmation, backend endpoint `POST /v1/team/accept-invitation` accepts the invite and enforces single-org policy:
    - Soft-deletes the previous merchant (`merchants.deletedAt`),
    - Deletes the previous organization if it has no other members; otherwise removes the current user’s membership.
    - Session refresh required client-side to reflect new org/merchant.

## 5. Current Focus

- Multi-PSP payment processing (Stripe & Dodo Payments)
- Invoice viewing and management UI
- Subscription lifecycle management (activate, pause, resume)
- Credit system UI and merchant dashboard integration
- Seat-based billing optimization and real-time usage tracking

## 6. Security Requirements

- Secret keys can access all endpoints; publishable keys only access public endpoints
- Multi-tenant isolation enforced for all operations
- Key rotation capability for publishable keys (auto-revokes old keys)
- Only return key prefixes (first 4 chars) in UI/API responses, never full values

### 6.1 Admin-Only Internal Key Values Management

- Admins (Better Auth `role = "admin"`) can manage global internal key/value pairs used by backend logic.
- API: Protected endpoints mounted at `/v1/internal/internal-key-values`.
  - `GET /v1/internal/internal-key-values` → list active rows
  - `GET /v1/internal/internal-key-values/:key` → fetch single row
  - `PUT /v1/internal/internal-key-values/:key` → upsert JSON `value`
- AuthZ: Requires session user with admin role OR secret API key with `admin` permission.
- UI: Admin-only page at `/admin/internal` (in the dashboard) to view and edit entries.
- Storage: `common/src/db/schema/internal_key_values.ts` (JSONB `value`, soft-delete via `deletedAt`).

## 7. Development Phase Notes

When instructed to use "FULL CONTEXT" with MCP tools, provide:

1. Project objectives
2. Relevant filenames and contents (with relative paths)
3. Approaches already tried and results

## 8. Coding Best Practices

- DO NOT TRY TO RUN TESTS YOURSELF WITHOUT ASKING FOR PERMISSION

## 9. Test Runner Guidelines

- TO RUN TESTS, YOU MUST: cd api && bun test <test_file_path>
- TO RUN INDIVIDUAL TESTS: cd /Users/prasoon/work/lumen/api && bun test tests/integration/<file> -t "test case"

## 10. ORM and Database

### 10.1 Drizzle ORM

- We use Drizzle ORM for DB migrations
- The TS file schemas are the source of truth which Drizzle uses to create SQL migration files

## 11. Type Checking

- If you want to run a type check, you need to run `tsc --noEmit` in either the `api`, the `ui` or the `common` folder on the file you want

---
> Source: [pretzelai/lumen](https://github.com/pretzelai/lumen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
