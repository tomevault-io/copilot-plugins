## saas-salessight

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

**Excel Sales Data Processor** - A full-stack web application for processing, managing, and analyzing retail inventory and sales data from Excel files. Features include file upload/parsing, database storage, real-time statistics dashboard, data management with search/pagination, and upload history tracking.

## Development Commands

- **Development**: `npm run dev` - Starts the development server with Vite HMR and Express backend (port 5000)
- **Build**: `npm run build` - Builds the client (Vite) and server (esbuild) for production
- **Production**: `npm start` - Runs the production build
- **Type Checking**: `npm run check` - Runs TypeScript compiler to check types
- **Database Push**: `npm run db:push` - Pushes Drizzle schema changes to the database (no migrations folder - uses direct push)

## Architecture Overview

This is a full-stack TypeScript application with React frontend and Express backend, using a monorepo structure.

### Tech Stack

**Frontend:**
- React 18 with TypeScript (via Vite)
- Wouter for routing (lightweight, not React Router)
- TanStack Query (React Query) for server state management
- shadcn/ui component library (48 components, ~15 actively used)
- Tailwind CSS with custom design tokens
- SheetJS (xlsx) for Excel file processing
- React Hook Form + Zod validation

**Backend:**
- Node.js with Express.js
- Drizzle ORM with Neon serverless PostgreSQL
- express-session with memorystore (dev only - not production-ready)
- Zod validation (shared with frontend)

### Project Structure

```
client/
├── src/
│   ├── components/
│   │   ├── ui/                     # 48 shadcn/ui components
│   │   ├── excel-formatter.tsx    # Main upload/processing UI
│   │   ├── file-upload-tabs.tsx   # Tabbed upload interface
│   │   ├── stats-overview.tsx     # Dashboard metrics
│   │   ├── upload-progress.tsx    # Progress tracking
│   │   └── recent-activity.tsx    # Upload history
│   ├── pages/
│   │   ├── dashboard.tsx          # Main landing page
│   │   ├── item-list.tsx          # Item management (399 lines)
│   │   └── not-found.tsx          # 404 page
│   ├── lib/
│   │   ├── excel-processor.ts     # Excel parsing & validation
│   │   ├── formatters.ts          # Complex Excel formatting (760 lines!)
│   │   ├── api.ts                 # API client functions
│   │   ├── queryClient.ts         # React Query setup
│   │   └── utils.ts               # Utility functions
│   ├── hooks/                     # Custom React hooks
│   ├── App.tsx                    # Root component with routing
│   └── main.tsx                   # Entry point

server/
├── index.ts                       # Server entry point (71 lines)
├── routes.ts                      # 10 REST API endpoints (240 lines)
├── storage.ts                     # Database operations (238 lines)
├── db.ts                          # Drizzle connection (16 lines)
└── vite.ts                        # Vite dev middleware

shared/
└── schema.ts                      # Drizzle schemas + Zod validation (109 lines)
```

### Database Architecture

Uses Drizzle ORM with Neon serverless PostgreSQL (WebSocket-based connection):

**Tables:**
- `item_list` (22 fields) - Inventory items with quantities across 8 store locations, pricing, dates, metadata
- `sales_transactions` (9 fields) - Sales records with receipt info, SKU, pricing
- `upload_history` (8 fields) - Tracks file uploads with success/failure statistics
- `users` (3 fields) - User management stub (not actively used - no auth implemented)

**Key Details:**
- Schema location: `shared/schema.ts` (shared between client/server for type consistency)
- Storage interface: `server/storage.ts` exports `storage` singleton implementing `IStorage`
- All database operations go through the storage layer, never directly via `db`
- No migrations folder - using `drizzle-kit push` directly (⚠️ risky for production)

### Import Aliases

- `@/` → `client/src/`
- `@shared/` → `shared/`
- `@assets/` → `attached_assets/`

### Development Workflow

In development mode (`npm run dev`):
1. Express server starts on port 5000 (or PORT env var)
2. Vite middleware provides HMR for React app
3. API routes under `/api/*` handled by Express
4. All other routes serve the React SPA

In production mode (`npm start`):
1. Serves pre-built static files from `dist/public`
2. Express handles API routes
3. Single server process

### Data Upload Flow

**Item List Upload:**
1. User selects Excel file → `client/src/components/excel-formatter.tsx`
2. Client parses with SheetJS → `client/src/lib/excel-processor.ts`
3. Validation with Zod schemas → `shared/schema.ts`
4. API request → `POST /api/upload/item-list` → `server/routes.ts:85`
5. Storage layer → `server/storage.ts:73` (createItemList or upsertItemList)
6. Database insertion → `item_list` table via Drizzle ORM
7. Upload history recorded → `upload_history` table with stats
8. Response → Success/failure counts + first 5 errors
9. React Query invalidates cache → UI auto-refreshes

**Upload Modes:**
- `initial` - Inserts new records (fails on duplicates)
- `weekly_update` - Upserts by item_number (updates existing records)

**Sales Transaction Upload:**
- Similar flow but goes to `POST /api/upload/sales-transactions`
- Stores in `sales_transactions` table
- Supports complex hierarchical Excel structure parsing (transaction headers + line items)

### API Endpoints (10 total)

- `GET /api/health` - Health check
- `GET /api/stats/item-list` - Item statistics (COUNT, SUM, DISTINCT aggregations)
- `GET /api/stats/sales` - Sales statistics
- `GET /api/item-list?limit=50&offset=0&search=...` - Paginated item list with search
- `DELETE /api/item-list/:id` - Delete single item
- `DELETE /api/item-list` - Clear all items (⚠️ destructive)
- `POST /api/upload/item-list` - Upload item data (batch processing)
- `POST /api/upload/sales-transactions` - Upload sales data
- `GET /api/upload-history?limit=10` - Recent upload history

### Excel Processing (Most Complex Module)

**File: `client/src/lib/formatters.ts` (760 lines)**

Key functions:
- `formatItemList()` - Deletes top 5 rows + specific columns from "Item Detail" sheet
- `formatSalesFile()` - Processes multiple "Sales Detail" sheets, deletes rows/columns
- `flattenSalesData()` - Parses hierarchical transaction structure (header rows + line items)
- `normalizeHeaders()` - Maps Excel headers to DB field names (handles variations)
- `coerceTypes()` - Type conversion for dates, numbers, strings
- Supports downloading formatted Excel/CSV files
- Business statistics calculation (top products, revenue, etc.)

**Hardcoded Values:**
- "Item Detail" sheet name (line 424)
- "Sales Detail" sheet name pattern (line 533)
- Specific column indices to delete (lines 445, 556)
- Row deletion counts (5 rows from top)

### Key Technical Details

- **Session handling**: `express-session` with `memorystore` (⚠️ in-memory, not production-ready)
- **Database connection**: WebSocket-based Neon serverless driver (`ws` package required)
- **Build outputs**: `dist/public/` (client) + `dist/index.js` (server)
- **File processing**: Client-side Excel parsing (may have memory issues with large files)
- **Batch processing**: 100 records per batch (hardcoded in `client/src/lib/api.ts:32`)
- **Pagination**: 50 items per page (hardcoded in `client/src/pages/item-list.tsx:56`)
- **Error tracking**: First 100 errors stored in upload_history, first 5 returned to client
- **TypeScript**: Strict mode with bundler module resolution
- **Replit-specific**: Vite plugins for dev banner, error overlay, cartographer

## Known Issues & Technical Debt

### Critical (Fix Before Production)
- ❌ **No test coverage** - Zero test files (no Jest, Vitest, etc.)
- ❌ **No database migrations** - Using `drizzle-kit push` directly (dangerous)
- ❌ **No authentication** - User schema exists but not implemented
- ❌ **No API security** - No rate limiting, CORS, or input sanitization beyond Zod
- ❌ **Memory session store** - Not suitable for production (use Redis/PostgreSQL)
- ⚠️ **Large files in repo** - 6.3MB Excel file committed twice in `attached_assets/`

### Important
- ⚠️ **No error boundaries** - React app has no error boundary components
- ⚠️ **Client-side processing** - Large Excel files may cause browser memory issues
- ⚠️ **Hardcoded values** - Sheet names, column indices in formatters.ts
- ⚠️ **Complex formatter** - 760-line file needs refactoring
- ⚠️ **No structured logging** - Using console.log only
- ⚠️ **Static dashboard stats** - "+12.5% from last week" is hardcoded (not real data)

### Nice to Have
- 🔹 **Unused UI components** - 48 components loaded, only ~15 used (bundle size impact)
- 🔹 **Unused dependencies** - Passport packages installed but not used
- 🔹 **Duplicate components** - `excel-formatter.tsx` and `file-upload-tabs.tsx` overlap
- 🔹 **No monitoring** - No error tracking (Sentry, etc.)
- 🔹 **No API versioning** - Should use `/api/v1/...`
- 🔹 **No Docker** - No containerization setup
- 🔹 **Type safety gaps** - Some `any` types in Excel processing code

## Important Patterns & Conventions

- Always use the storage layer (`server/storage.ts`) for database operations, never query directly
- Excel processing happens client-side before upload (reduces server load but limits file size)
- React Query handles all server state - invalidate queries after mutations
- Zod schemas in `shared/schema.ts` are the single source of truth for validation
- Component composition: feature components use shadcn/ui primitives
- Upload modes: "initial" for fresh imports, "weekly_update" for incremental updates
- Error handling: Collect errors during batch processing, return summary to user

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Willabor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
