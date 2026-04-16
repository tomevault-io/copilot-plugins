## bosternexus

> You are an expert full-stack developer working on **Boster Nexus**, an internal operations and enablement web app for Bosterbio. This is a Next.js application built on the ShipFast boilerplate foundation.

You are an expert full-stack developer working on **Boster Nexus**, an internal operations and enablement web app for Bosterbio. This is a Next.js application built on the ShipFast boilerplate foundation.

## 🎯 Project Purpose

Boster Nexus centralizes Boster's cross-system workflows across:
- **Zoho Books** (source of truth for transactional data)
- **Zoho CRM** (customer relationship management)
- **Zoho Desk** (support tickets)
- **Magento** (storefront/e-commerce)
- **Marketing Analytics** (GA4/BigQuery)

The app acts as a **cache/warehouse** and **system of record for derived/internal-only data**, not replacing source systems but enabling efficient operations and guardrails.

## 🚀 Tech Stack

- **Next.js**: 15.1.9 with React 19 and Turbopack
- **React**: 19.0.0 (latest stable)
- **Tailwind CSS**: 4.0.0 (CSS-based configuration)
- **DaisyUI**: 5.0.50 (latest with new component system)
- **Supabase**: Latest SSR package with async cookies support (Auth + Postgres)
- **Database**: Managed Postgres (Neon/Supabase) - primarily cache/warehouse
- **Deployment**: Vercel

## 📊 Core Data Principles

### Source of Truth Hierarchy
1. **Zoho Books** is the source of truth for:
   - Orders, invoices, POs, bills, inventory state
   - All transactional financial data
2. **Nexus DB** stores:
   - **Raw entity caches** (structured tables from Books/CRM/Desk)
   - **Derived/computed state** that Zoho cannot represent cleanly:
     - Bill↔PO line matching
     - Guardrail exceptions
     - Dashboard layout config
     - Loyalty ledger
     - Wiki versions
3. **BigQuery** remains the store for GA4-scale event data; Nexus pulls only what it needs on-demand

### Rate Limiting Constraints
⚠️ **CRITICAL**: Zoho API rate limit is **no more than 1 request per second**
- The Zoho class (`@/libs/zoho.js`) handles rate limiting automatically
- **Always use the Zoho class** - never make direct API calls
- For long chains, prefer async jobs (Postgres job table)
- Design must prevent parallel overload
- The Zoho class ensures all requests respect the 1 req/sec limit

## 📁 File Structure & Organization

### Route Groups (Next.js App Router)
```
app/
├── (public)/              # Public pages (no auth required)
│   ├── layout.js         # Public layout wrapper
│   ├── page.js           # Landing page
│   ├── blog/             # Blog pages
│   ├── privacy-policy/   # Legal pages
│   └── tos/
├── (private)/            # Private pages (auth required)
│   ├── layout.js         # Private layout with auth check
│   ├── admin/            # Admin functions
│   ├── freezer/          # Freezer label printer
│   └── zoho-test/        # Zoho testing interface
├── api/                  # API routes
│   ├── zoho/            # Zoho webhook ingestion and operations
│   ├── auth/            # Authentication endpoints
│   ├── stripe/          # Stripe payment endpoints
│   ├── migrations/       # Database migration endpoints
│   └── webhook/         # Webhook handlers
├── signin/              # Authentication pages
├── dashboard/           # Dashboard pages
└── layout.js            # Root layout
```

### Service Layer Architecture
```
libs/
├── zoho/                # Zoho integration
│   ├── index.js         # CRITICAL: Central Zoho class for all Zoho operations
│   ├── auth.js          # Zoho authentication service
│   ├── config.js        # Zoho configuration
│   ├── rate-limiter.js  # Rate limiting for Zoho API
│   ├── entities/        # Entity classes for data transformation
│   │   ├── books/       # Zoho Books entities
│   │   └── index.js
│   ├── orchestration/   # Orchestration services
│   │   └── ZohoOrchestrator.js
│   └── services/        # Business logic services
│       └── books/       # Zoho Books services
├── supabase/            # Supabase client utilities
│   ├── client.js        # Client-side Supabase client
│   ├── server.js        # Server-side Supabase client
│   ├── data-access-layer.js  # CRITICAL: Data Access Layer for all DB operations
│   └── middleware.js    # Supabase middleware utilities
├── utils/               # Shared utilities
│   └── logger.js        # Structured file logging utility
├── zoho.js              # Legacy export (re-exports from zoho/index.js)
├── stripe.js            # Stripe integration
├── api.js               # API client utilities
└── [other]/             # Other utility libraries
```

### Database Organization
```
Database Tables:
- zoho_books_*           # Zoho Books data (e.g., zoho_books_items, zoho_books_invoices, zoho_books_salesorders, zoho_books_purchaseorders, zoho_books_bills)
- zoho_books_*_items     # Line items (e.g., zoho_books_invoice_items, zoho_books_salesorder_items)
- auth_tokens            # OAuth tokens for Zoho and other services
- profiles               # User profiles
- derived_*               # Computed/derived state (e.g., derived_bill_po_matches)
- jobs_*                 # Async job queue (e.g., jobs_sync, jobs_reconciliation)
- config_*               # Configuration (e.g., config_dashboard_layouts, config_report_definitions)
- internal_*              # Internal-only data (e.g., internal_guardrail_exceptions, internal_loyalty_ledger)
```

## ⚠️ Critical Rules

### 1. **Zoho API Operations** (MANDATORY)
**ALL Zoho-related operations MUST use the Zoho class from `@/libs/zoho.js`.**
**NEVER code Zoho API calls from scratch or use vanilla fetch/axios directly.**

```javascript
// ❌ WRONG - Direct API call without using Zoho class
const response = await fetch('https://books.zoho.com/api/v3/items');
const items = await axios.get('https://books.zoho.com/api/v3/items');

// ❌ WRONG - Manual rate limiting (even if correct, violates class usage rule)
import { zohoRateLimiter } from '@/libs/utils/rate-limiter';
await zohoRateLimiter.acquire();
try {
  const response = await fetch('https://books.zoho.com/api/v3/items');
} finally {
  zohoRateLimiter.release();
}

// ✅ CORRECT - Always use the Zoho class (rate limiting is built-in)
import zoho from '@/libs/zoho';

// Zoho Books operations
const items = await zoho.getBooksItems();
const invoice = await zoho.getBooksInvoice('123456');
const invoices = await zoho.getBooksInvoices({ page: 1, per_page: 25 });

// Zoho CRM operations
const contacts = await zoho.getCrmRecords('Contacts');
const lead = await zoho.getCrmRecord('Leads', '123456');
await zoho.createCrmRecord('Contacts', { First_Name: 'John', Last_Name: 'Doe' });

// Zoho Desk operations
const tickets = await zoho.getDeskTickets();
const ticket = await zoho.getDeskTicket('123456');

// Generic methods (if specific methods don't exist)
const customData = await zoho.get('books', '/custom-endpoint', { param: 'value' });

// ✅ CORRECT - For long workflows, use async jobs
const dal = new DataAccessLayer({ useServiceRole: true });
await dal.insert('jobs_sync', {
  type: 'items_refresh',
  payload: { /* ... */ },
  status: 'pending'
});
// The job processor will use zoho.getBooksItems() when executing
```

**Key Points:**
- The Zoho class handles rate limiting automatically (1 req/sec)
- The Zoho class handles token refresh automatically
- The Zoho class provides type-safe methods for common operations
- All Zoho API calls must go through this class - no exceptions

### 2. **Data Access Layer (DAL) Pattern** (MANDATORY)
**ALL database operations MUST use the Data Access Layer from `@/libs/supabase/data-access-layer.js`.**
**NEVER use direct `supabase.from()` calls for database operations.**

**Exception:** Auth operations (`supabase.auth.getUser()`, `supabase.auth.admin.*`) can remain direct, but use DAL's `getCurrentUser()` or `getCurrentUserId()` methods when possible.

```javascript
// ❌ WRONG - Direct Supabase database operations
const supabase = await createClient();
await supabase.from('zoho_books_items').upsert({ zoho_id: '123', name: 'Item' });
const { data } = await supabase.from('profiles').select().eq('id', userId);

// ❌ WRONG - Direct Supabase client for auth when DAL method exists
const supabase = await createClient();
const { data: { user } } = await supabase.auth.getUser();

// ✅ CORRECT - Use Data Access Layer for all database operations
import { DataAccessLayer } from '@/libs/supabase/data-access-layer';

// For system-level operations (Zoho syncs, webhooks, etc.)
const dal = new DataAccessLayer({
  useServiceRole: true,    // Bypass RLS for system operations
  requireUserId: false,     // Zoho data is not user-specific
  autoTimestamps: true      // Auto-manage created_at/updated_at
});

await dal.upsert('zoho_books_items', {
  zoho_id: '123',
  name: 'Item Name',
  // user_id automatically injected if requireUserId: true
  // created_at/updated_at automatically managed if autoTimestamps: true
}, { onConflict: 'zoho_id' });

// For user-specific operations
const dal = new DataAccessLayer({
  useServiceRole: false,    // Use authenticated client
  requireUserId: true,      // Enforce user_id injection
  autoTimestamps: true      // Auto-manage timestamps
});

const profile = await dal.getSingle('profiles', { id: userId });
await dal.insert('user_preferences', { key: 'theme', value: 'dark' });

// ✅ CORRECT - Use DAL for auth context (preferred over direct)
const dal = new DataAccessLayer({
  useServiceRole: false,
  requireUserId: false,
  autoTimestamps: false,
});

// Get user ID
const userId = await dal.getCurrentUserId();

// Get full user object
const user = await dal.getCurrentUser();

// ✅ CORRECT - Direct Supabase client ONLY for auth operations not covered by DAL
// (e.g., supabase.auth.admin.createUser, supabase.auth.exchangeCodeForSession)
import { createClient } from '@/libs/supabase/server';
const supabase = await createClient();
await supabase.auth.admin.createUser({ email: 'user@example.com' });
```

**DAL Methods:**
- `dal.insert(tableName, records, options)` - Insert records
- `dal.upsert(tableName, records, options)` - Upsert records
- `dal.update(tableName, updates, filters)` - Update records
- `dal.select(tableName, options)` - Select records with filters, ordering, pagination
- `dal.delete(tableName, filters)` - Delete records
- `dal.getSingle(tableName, filters)` - Get single record
- `dal.getCurrentUserId()` - Get current user ID from auth context
- `dal.getCurrentUser()` - Get current user object from auth context

**DAL Options:**
- `useServiceRole: boolean` - Use service role client (bypasses RLS)
- `requireUserId: boolean` - Automatically inject user_id from auth context
- `autoTimestamps: boolean` - Automatically manage created_at/updated_at

**Key Points:**
- DAL ensures RLS compliance and validates user_id matches
- DAL automatically injects user_id when requireUserId: true
- DAL automatically manages timestamps when autoTimestamps: true
- DAL provides better error messages for RLS failures
- Use service role for system-level operations (Zoho syncs, webhooks)
- Use authenticated client for user-specific operations

### 3. **Supabase Client Usage** (For Auth Operations Only)
**Direct Supabase client should ONLY be used for auth operations not covered by DAL.**

```javascript
// ❌ WRONG - Will cause async cookies warning
const supabase = createClient();

// ✅ CORRECT - Always await in server components
const supabase = await createClient();

// ✅ CORRECT - Client components use different import
import { createClient } from "@/libs/supabase/client"; // For client components
import { createClient } from "@/libs/supabase/server"; // For server components (async)

// ✅ CORRECT - Use DAL for getting user context (preferred)
const dal = new DataAccessLayer({ useServiceRole: false });
const user = await dal.getCurrentUser();
const userId = await dal.getCurrentUserId();

// ✅ CORRECT - Direct client ONLY for auth operations not in DAL
// (e.g., admin operations, code exchange)
const supabase = await createClient();
await supabase.auth.admin.createUser({ email: 'user@example.com' });
const { data } = await supabase.auth.exchangeCodeForSession(code);
```

### 4. **Next.js 15 Conventions**

#### Headers API
```javascript
// ❌ WRONG - Synchronous headers access
const signature = headers().get("stripe-signature");

// ✅ CORRECT - Always await headers in Next.js 15
const signature = (await headers()).get("stripe-signature");
```

#### Proxy File (Not Middleware)
**Next.js 15 uses `proxy.js` instead of `middleware.js`.**

```javascript
// ✅ CORRECT - Use proxy.js file in project root
// proxy.js
import { updateSession } from "@/libs/supabase/middleware";

export async function proxy(request) {
  return await updateSession(request);
}

export const config = {
  matcher: [/* ... */],
};
```

**Key Points:**
- File name: `proxy.js` (not `middleware.js`)
- Export function: `proxy` (not `middleware`)
- See `proxy.js` in project root for reference

### 5. **Service Layer Pattern**
```javascript
// ❌ WRONG - Business logic in route handler
// app/api/zoho/items/route.js
export async function GET(req) {
  const items = await fetch('https://books.zoho.com/api/v3/items');
  const processed = items.map(/* complex logic */);
  return NextResponse.json(processed);
}

// ✅ CORRECT - Thin route handler → Service → Zoho Class
// app/api/zoho/items/route.js
import { ZohoItemsService } from '@/libs/services/zoho/items';

export async function GET(req) {
  const service = new ZohoItemsService();
  const items = await service.getAllItems();
  return NextResponse.json(items);
}

// libs/services/zoho/items.js
import zoho from '@/libs/zoho';
import { DataAccessLayer } from '@/libs/supabase/data-access-layer';

export class ZohoItemsService {
  constructor() {
    this.dal = new DataAccessLayer({
      useServiceRole: true,
      requireUserId: false,
      autoTimestamps: true,
    });
  }

  async getAllItems() {
    // Business logic here
    // MUST use zoho class - never direct API calls
    const itemsData = await zoho.getBooksItems();
    
    // Use DAL for database operations
    for (const item of itemsData) {
      await this.dal.upsert('zoho_books_items', {
        zoho_id: item.id,
        name: item.name,
        // ... other fields
      }, { onConflict: 'zoho_id' });
    }
    
    return itemsData;
  }
}
```

### 6. **Data Synchronizer Storage Pattern**

The `zoho_books_*` tables have been removed (migration 027). All Zoho Books data now lives in the lean `zb_*` tables created by migration 026. Each table has:
- `external_id` — Zoho's native ID (e.g., `item_id`, `invoice_id`)
- `source_system` — `'zoho_books'` or `'zoho_crm'`
- `indexes` — JSONB with fields needed for fast lookups (defined in `libs/data-synchronizer/registry.js`)
- `raw_json` — full Zoho payload for detail views
- `remote_updated_at` — `last_modified_time` from Zoho; used for incremental sync decisions

```javascript
// ❌ WRONG - old zoho_books_* tables no longer exist
await dal.upsert('zoho_books_items', dbRecord, { onConflict: 'zoho_id' });

// ✅ CORRECT - use SyncRecordRepository + adapter
import '@/libs/data-synchronizer/index';          // registers all adapters
import { SyncRecordRepository } from '@/libs/data-synchronizer/storage/SyncRecordRepository';
import { getAdapterForTable }   from '@/libs/data-synchronizer/registry';

const repo    = new SyncRecordRepository();
const Adapter = getAdapterForTable('zb_items');

await repo.upsertOne('zb_items', {
  external_id:       Adapter.extractExternalId(zohoItem),
  source_system:     'zoho_books',
  indexes:           Adapter.extractIndexes(zohoItem),
  raw_json:          zohoItem,
  remote_updated_at: Adapter.extractRemoteUpdatedAt(zohoItem),
});

// Reading items by SKU (preserves old response shape for consumers)
const dal = new DataAccessLayer({ useServiceRole: true, requireUserId: false, autoTimestamps: false });
const { data } = await dal.select('zb_items', {
  queryBuilder: q => q.select('external_id, indexes').in('indexes->>sku', skus),
});
const items = data.map(r => ({
  zoho_id:        r.external_id,
  sku:            r.indexes?.sku,
  name:           r.indexes?.name,
  reorder_level:  r.indexes?.reorder_level,
  stock_on_hand:  r.indexes?.qty,
  available_stock: r.indexes?.available_stock,
}));
```

**Current Table Names (zb_* = Zoho Books, zcrm_* = Zoho CRM):**
- `zb_items` — items/products (replaces `zoho_books_items`)
- `zb_invoices` — invoices (replaces `zoho_books_invoices`)
- `zb_salesorders` — sales orders (replaces `zoho_books_salesorders`)
- `zb_purchaseorders` — purchase orders (replaces `zoho_books_purchaseorders`)
- `zb_bills` — bills (replaces `zoho_books_bills`)
- `zb_contacts` — Books contacts
- `zb_lineitems` — all line items (sales orders, invoices, POs, bills); has `parent_type`, `parent_external_id`
- `zcrm_contacts` — Zoho CRM contacts

Index fields per table are defined in `libs/data-synchronizer/registry.js`.

### 7. **File Upload Chunking** (For Large File Ingestion)

**ALL file-based data ingestion features MUST use the shared chunking utilities** from `@/libs/upload-chunking.js` to avoid Next.js request body size limits (proxyClientMaxBodySize default ~10MB, configurable in `next.config.js`).

- **`chunkCsv(csvText, options)`** — For CSV with header row. Splits by byte size; each chunk includes the header. Use for Antigen Designer ingestion, PIM CSV, etc.
- **`chunkNdjson(text, options)`** — For NDJSON (one JSON per line). Splits by line count. Use for Data Synchronizer NDJSON uploads.

```javascript
// ✅ CORRECT - Use shared chunking
import { chunkCsv, chunkNdjson } from '@/libs/upload-chunking';

// CSV with header
const csvChunks = chunkCsv(await file.text(), { chunkBytes: 15 * 1024 * 1024 });
for (let i = 0; i < csvChunks.length; i++) {
  const fd = new FormData();
  fd.append('file', new Blob([csvChunks[i]], { type: 'text/csv' }), file.name);
  const result = await apiClient.post(endpoint, fd);
  // aggregate result...
}

// NDJSON (no header)
const ndjsonChunks = chunkNdjson(await file.text(), { linesPerChunk: 500 });
for (const chunk of ndjsonChunks) {
  const fd = new FormData();
  fd.append('file', new Blob([chunk], { type: 'text/plain' }), 'chunk.ndjson');
  // ...
}
```

```javascript
// ❌ WRONG - Defining chunking logic inline per feature
function myChunkCsv(text) { /* duplicate implementation */ }
```

**Key points:**
- Do NOT define chunking logic inline in individual pages/components
- Use `chunkCsv` for header-preserving CSV splits; use `chunkNdjson` for line-based NDJSON
- Upload chunks sequentially; aggregate results client-side
- `next.config.js` sets `experimental.proxyClientMaxBodySize` for overall limit (e.g. 500mb)

## 🔐 Authentication & Authorization

### Route Protection
- **Public routes**: Use `app/(public)/layout.js` - no auth check
- **Private routes**: Use `app/(private)/layout.js` - checks Supabase auth, redirects to `/signin` if not logged in
- **Admin routes**: Use `app/(private)/admin/layout.js` - checks auth, future: add role/permission checks

### Login Flow
1. User visits protected route
2. `(private)/layout.js` checks authentication using DAL
3. If not authenticated → redirect to `/signin`
4. After successful login → redirect to `config.auth.callbackUrl` (default: `/admin`)

### Auth Implementation Pattern
```javascript
// ✅ CORRECT - Use DAL for auth checks in layouts
import { DataAccessLayer } from '@/libs/supabase/data-access-layer';

export default async function PrivateLayout({ children }) {
  const dal = new DataAccessLayer({
    useServiceRole: false,
    requireUserId: false,
    autoTimestamps: false,
  });
  
  const user = await dal.getCurrentUser();
  
  if (!user) {
    redirect(config.auth.loginUrl);
  }
  
  // ... rest of layout
}
```

## 📦 Key Modules (Implementation Order)

### Phase 1: Foundation (Current)
1. ✅ Authentication & login
2. ✅ Folder structure (`(public)` and `(private)/admin`)
3. ✅ Layout files with auth checks
4. ✅ Logging system (Logger utility + log viewer scripts)
5. ✅ Data Access Layer (DAL) implementation
6. ✅ Zoho Books sync infrastructure

### Phase 2: Core Integration
1. **Zoho Books sync + cache + reconciliation**
   - Webhook ingestion endpoints
   - Rate-limited sync jobs
   - Daily reconciliation
   - Historical sync service

2. **Procurement & fulfillment operational signals**
   - PO quantity calculations
   - Backorder/on-the-way signals
   - SO guardrails

### Phase 3: Extended Features
3. Zoho CRM ingestion + tools
4. Quote lifecycle automation
5. Customer portal widgets
6. Zoho Desk sync
7. Marketing attribution dashboards
8. Sales analytics + AI curation
9. PIM (Product Information Management)
10. Loyalty system
11. Notifications
12. Monitoring/alerts framework

## 🎨 Styling Rules

### Tailwind CSS v4 Syntax
```css
@import "tailwindcss";
@plugin "daisyui";

@theme {
  --color-primary: #570df8; /* Boster brand color - adjust as needed */
}
```

### DaisyUI v5 Component Usage
```html
<div class="card card-border"> <!-- v5: card-border instead of card-bordered -->
<button class="btn btn-primary btn-sm">
```

## 🚫 Common Mistakes to Avoid

1. **DON'T** make Zoho API calls without using the Zoho class from `@/libs/zoho.js`
2. **DON'T** code Zoho API operations from scratch - always use zoho class methods
3. **DON'T** use direct fetch/axios calls to Zoho APIs - use zoho.get(), zoho.post(), etc.
4. **DON'T** use direct `supabase.from()` calls for database operations - use DAL
5. **DON'T** store Zoho data as source of truth (it's a cache)
6. **DON'T** use synchronous `headers()` or `cookies()` in Next.js 15
7. **DON'T** put business logic directly in route handlers
8. **DON'T** import unused dependencies
9. **DON'T** use `createClient()` without await in server components
10. **DON'T** replicate GA4-scale data into Nexus DB (pull on-demand from BigQuery)
11. **DON'T** use `console.log/error/warn` for production logging - use `Logger` from `@/libs/utils/logger`
12. **DON'T** use `middleware.js` - use `proxy.js` for Next.js 15
13. **DON'T** use table names like `zoho_cache_*` - actual tables are `zoho_books_*`

## 🔧 Development Best Practices

### Async Jobs Pattern
```javascript
// For long-running or rate-limited operations:
// 1. Insert job into queue using DAL
import { DataAccessLayer } from '@/libs/supabase/data-access-layer';

const dal = new DataAccessLayer({ useServiceRole: true });
await dal.insert('jobs_sync', {
  type: 'items_full_refresh',
  status: 'pending',
  payload: { /* ... */ }
});

// 2. Background worker processes jobs (separate process/cron)
// Uses rate limiter, handles retries, updates status
```

### Error Handling
```javascript
// Always handle Zoho API errors gracefully
import zoho from '@/libs/zoho';
import { Logger } from '@/libs/utils/logger';
import { DataAccessLayer } from '@/libs/supabase/data-access-layer';

try {
  const data = await zoho.getBooksItems();
} catch (error) {
  if (error.message.includes('429')) {
    // Rate limited - should queue as job instead
    const dal = new DataAccessLayer({ useServiceRole: true });
    await dal.insert('jobs_sync', {
      type: 'items_refresh',
      status: 'pending',
      payload: { retryAfter: error.retryAfter }
    });
  }
  // Log errors using Logger (not console.error)
  Logger.error('Zoho API error', error, { 
    operation: 'getBooksItems',
    service: 'books'
  });
}
```

### Logging System
**ALWAYS use the Logger utility instead of console.log/error/warn for production code.**

The project has a structured logging system that writes to files in the `logs/` directory. Logs are organized by level and are accessible to AI agents for debugging.

**Logger Usage:**
```javascript
// ✅ CORRECT - Use Logger utility
import { Logger } from '@/libs/utils/logger';

// Log errors (with error object and context)
Logger.error('Operation failed', error, { 
  userId: '123',
  operation: 'syncItems',
  service: 'books'
});

// Log warnings
Logger.warn('Rate limit approaching', { 
  remaining: 10,
  service: 'books'
});

// Log info messages
Logger.info('Items synced successfully', { 
  count: 5,
  service: 'books'
});

// Log debug (only in development)
Logger.debug('Processing batch', { 
  batchNumber: 1,
  size: 50
});

// ❌ WRONG - Don't use console methods for production logging
console.error('Error:', error);
console.log('Info:', data);
console.warn('Warning:', message);
```

**Log Files:**
- Logs are written to `logs/` directory in the project root
- Level-specific files: `error.log`, `warn.log`, `info.log`, `debug.log`
- Combined file: `combined.log` (all levels)
- Log files are JSON-formatted, one entry per line
- Log files are git-ignored (see `.gitignore`)

**Checking Logs:**
```bash
# View recent error logs (default: last 50)
npm run logs:error

# View recent info logs
npm run logs:info

# View all recent logs (combined)
npm run logs:combined

# Or use the script directly with custom parameters
node scripts/check-logs.js error 20   # Last 20 error logs
node scripts/check-logs.js info 100   # Last 100 info logs
node scripts/check-logs.js warn 30    # Last 30 warning logs
```

**When to Use Logging:**
- **Error**: Always log errors with full context (error object, operation, relevant IDs)
- **Warn**: Use for recoverable issues, rate limit warnings, validation failures
- **Info**: Use for important operations (syncs, API calls, state changes)
- **Debug**: Use for detailed debugging info (only in development)

**Best Practices:**
- Always include context data (service, operation, IDs, counts)
- Use structured data objects, not string concatenation
- Log at appropriate levels (don't log everything as error)
- Include error objects when logging errors
- Log important state changes and sync operations

## 🎯 Commit Message Format
Use Conventional Commits:
- `feat:` for new features
- `fix:` for bug fixes  
- `refactor:` for code refactoring
- `style:` for formatting changes
- `docs:` for documentation
- `perf:` for performance improvements
- `chore:` for maintenance tasks

## 🔍 Before Committing Checklist
- [ ] No unused imports
- [ ] All async functions properly awaited
- [ ] **All Zoho API calls use zoho class from `@/libs/zoho.js`** (MANDATORY)
- [ ] No direct fetch/axios calls to Zoho APIs
- [ ] **All database operations use DAL from `@/libs/supabase/data-access-layer.js`** (MANDATORY)
- [ ] No direct `supabase.from()` calls for database operations
- [ ] **Use Logger utility instead of console methods for production logging**
- [ ] No ESLint warnings
- [ ] Build passes (`npm run build`)
- [ ] All environment variables checked
- [ ] Proper error handling in API routes
- [ ] Server/client components used correctly
- [ ] Business logic in service layer, not route handlers
- [ ] Using `proxy.js` (not `middleware.js`) for Next.js 15

## 🛠️ Development Commands
```bash
npm run dev          # Development with Turbopack
npm run build        # Production build
npm run lint         # ESLint check
npm run lint:fix     # Auto-fix ESLint issues

# Log viewing commands
npm run logs         # View recent error logs (default)
npm run logs:error   # View recent error logs
npm run logs:info    # View recent info logs
npm run logs:combined # View all recent logs
```

## 📝 Environment Variables

Required environment variables (see `.env.local`):
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`
- `ZOHO_BOOKS_CLIENT_ID`
- `ZOHO_BOOKS_CLIENT_SECRET`
- `ZOHO_BOOKS_ACCESS_TOKEN` (or refresh token flow)
- `ZOHO_CRM_CLIENT_ID` (similar pattern)
- `ZOHO_CRM_CLIENT_SECRET`
- `ZOHO_DESK_CLIENT_ID` (similar pattern)
- `ZOHO_DESK_CLIENT_SECRET`
- `DATABASE_URL` or `SUPABASE_DB_URL` or `SUPABASE_DB_PASSWORD` (for migrations)

Remember: This is an **internal operations tool**. Focus on reliability, data accuracy, and efficient cross-system workflows. Always respect rate limits and maintain clean separation between cache and source of truth.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/boster00) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
