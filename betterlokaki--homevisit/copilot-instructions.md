## homevisit

> **Strictly enforced in this codebase:**

# HomeVisit Development Rules for Cursor AI

## SOLID Principles & Code Structure

**Strictly enforced in this codebase:**

- **One type/class per file** - Never define multiple types or classes in a single file
- **Max 100 lines per file** - Split larger files into focused modules
- **Single responsibility** - Each class handles one logic only
- **Helper functions** - When a function needs multiple operations, split into helpers and orchestrate
- **Dependency injection** - Never instantiate dependencies inside classes, inject from outside
- **Interface contracts** - Use interfaces between classes/services, never couple directly
- **Throw first, catch later** - Services throw errors, only catch at the highest layer (controllers/schedulers)
- **Separation of concerns** - Never mix different layers (no business logic in controllers, no data access in services)

## Error Handling: Throw First, Catch Later

**Services and low-level modules must throw errors, never catch them:**

```typescript
// ❌ WRONG - Service catches its own errors
async fetchData(): Promise<Data[]> {
  try {
    const response = await axios.get(url);
    return response.data;
  } catch (error) {
    logger.error("Failed", error);
    return []; // Swallowing the error
  }
}

// ✅ CORRECT - Service throws, let caller decide
async fetchData(): Promise<Data[]> {
  const response = await axios.get(url);
  return response.data;
}
```

**Only catch at the highest layer:**

- **Controllers** - Catch and send HTTP error response
- **Schedulers/Orchestrators** - Catch to allow other operations to continue
- **Server startup** - Catch for graceful degradation or exit

```typescript
// Controller (highest layer) - catches and responds
async getSites(req: Request, res: Response): Promise<void> {
  try {
    const sites = await this.siteService.getAll();
    sendSuccess(res, sites);
  } catch (error) {
    sendError(res, "Failed to fetch sites", 500, error);
  }
}

// Scheduler orchestration - catches per-item for resilience
for (const group of groups) {
  try {
    await this.refreshGroup(group.name);
  } catch (error) {
    logger.error(`Failed for group: ${group.name}`, error);
    // Continue with next group
  }
}
```

## Backend Service Directory Structure

Each service lives in its own directory with co-located interfaces:

```
services/
  serviceName/
    interfaces/
      IServiceName.ts    # Interface definition
      index.ts           # Interface exports
    serviceName.ts       # Implementation
    helperFunction.ts    # Helper functions (if needed)
    index.ts             # Service exports
```

Example: `services/site/interfaces/ISiteService.ts`

## Shared Types Directory Structure

Types in `packages/common` are organized by domain:

```
packages/common/src/models/
  site/       # Site, EnrichedSite, SeenStatus, CoverStatus (formerly UpdatedStatus)
  user/       # User, AuthPayload
  filter/     # SiteFilters, FilterRequest
  enrichment/ # EnrichmentRequestBody, EnrichmentResponseBody
  history/    # SiteHistory, CoverUpdateEntry, MergedHistoryEntry, MergedStatus
  group/      # Group
  api/        # ApiResponse types
  overlay/    # ElasticProviderOverlay, FilterSchema
```

**Rule:** One type per file per subdirectory.

## Backend Dependency Injection

All dependencies injected via constructor. Factory pattern in `controllers/controllerFactory.ts`:

```typescript
const postgrestClient = new PostgRESTClient();
const siteService = new SiteService(postgrestClient);
const groupService = new GroupService(postgrestClient);
export const sitesController = new SitesController(siteService, groupService, ...);
```

## Shared Types

Import from `@homevisit/common` - never duplicate types between apps:

```typescript
import type { EnrichedSite, User, FilterRequest } from "@homevisit/common";
```

## Response Helpers

Use utility functions in `utils/responseHelper.ts`:

```typescript
sendSuccess(res, data);
sendError(res, message, 500, error);
sendValidationError(res, "Missing required parameter");
```

## Frontend Conventions

- **RTL Layout**: All components use `dir="rtl"` - Hebrew is the primary language
- **No px units**: Use `%`, `rem`, or Tailwind relative classes only (never use px)
- **Generic components**: Build components as reusable/generic as possible
- **Context separation**: Split Svelte context into separate files by concern
- **Component drilling**: Pass data via props, use Svelte context for deep sharing
- **Map library**: MapLibre GL for geographic visualization

### Frontend Component Development

When writing any new frontend feature:

1. First create a component for each thing, with drilling properties using best practices
2. Or use context, which is also divided by logic using best practices for frontend
3. **Never use px** - always use `%` and/or relative sizes (we never know which screen will see it)

### Frontend Store Pattern

Svelte stores in `src/stores/visit/` with API clients separated:

- `visitStore.ts` - State management with Svelte writable
- `visitApiClient.ts` - HTTP communication (single responsibility)
- Types separated into individual files

## Architecture Overview

HomeVisit is a monorepo (pnpm workspaces) for aerial photography site visit management with:

- **apps/backend**: Express + TypeScript API (port 4000), connects to PostgREST (port 3000)
- **apps/frontend**: SvelteKit + Tailwind app (port 5173), RTL Hebrew interface
- **packages/common**: Shared TypeScript types used by both apps
- **postgres**: Docker Compose stack with PostGIS + PostgREST

**Data Flow**: Frontend → Backend API → PostgREST → PostgreSQL/PostGIS

## Development Workflow

```bash
# Start full stack (Docker + Backend + Frontend)
# Use VS Code task: "🚀 Start Full Development Stack"

# Or manually:
cd postgres && docker-compose up -d   # Start DB + PostgREST
cd apps/backend && npm run dev        # tsx watch mode (port 4000)
cd apps/frontend && npm run dev       # Vite dev server (port 5173)
```

**API Docs**: http://localhost:4000/api-docs (Swagger UI)

## Figma-to-Code Workflow

Components map from Figma frames (see `apps/frontend/README.md` for element mapping):

- Frame IDs in comments help trace design → code
- Status types: בתהליך ביקור, אין איסוף, מחכה לביקור, בוצע, בוצע חלקית
- Dark theme (#000000 background, #141414 containers)

## Configuration

- **Backend**: Zod-validated env vars in `config/env.ts` - fails fast on invalid config
- **Frontend**: Vite env vars with `VITE_` prefix in `config/env.ts`
- **Enrichment config**: External JSON loaded at startup (`config.json`)
- **Config types**: Separated in `config/types/` directory

## Database

PostGIS-enabled PostgreSQL via PostgREST API. Schema in `postgres/db/init.sql`.

- Geometries stored as WKT strings
- Use `@turf/turf` for geometry operations in backend

## Important Files

- `apps/backend/src/controllers/controllerFactory.ts` - DI composition root
- `apps/backend/src/services/index.ts` - All service exports
- `packages/common/src/models/` - All shared types (one type per file per subdirectory)
- `apps/frontend/src/stores/visit/` - Main frontend state
- `apps/backend/config.json` - External service configuration

## Backend Services

| Service               | Directory                | Purpose                                 |
| --------------------- | ------------------------ | --------------------------------------- |
| `SiteService`         | `services/site/`         | CRUD operations for sites via PostgREST |
| `GroupService`        | `services/group/`        | Group management                        |
| `UserService`         | `services/user/`         | User data access                        |
| `EnrichmentService`   | `services/enrichment/`   | Calls external API to enrich site data  |
| `FilterService`       | `services/filter/`       | Runtime filtering logic                 |
| `SiteHistoryService`  | `services/siteHistory/`  | Site history tracking                   |
| `CoverUpdateService`  | `services/coverUpdate/`  | Cover update fetching                   |
| `HistoryMergeService` | `services/historyMerge/` | Merges cover and visit history          |
| `OverlayService`      | `services/overlay/`      | Fetches overlays from elastic provider  |
| `GeometryService`     | `services/geometry/`     | Geometry conversions                    |
| `PostgRESTClient`     | `services/postgrest/`    | HTTP client for PostgREST API           |

## External Enrichment Service

Configured in `config.json` at startup (loaded in `config/enrichmentConfig.ts`):

```typescript
// Request structure uses configurable keys:
{ [dataKey]: { text: geometries, text_id: siteNames }, [dateKey]: { StartTime: { From, To } } }
```

- Returns site status (`Full`, `Partial`, `No`) and project links
- Fallback: Sites get `coverStatus: "No"` if service fails

## Status Terminology & Merged Statuses

**CRITICAL: Use consistent terminology throughout the codebase:**

### Core Status Types

- **`coverStatus`** (formerly `updateStatus`): Represents the coverage status from external service

  - Type: `CoverStatus = "Full" | "Partial" | "No"`
  - Indicates whether new coverage data came in today
  - **Always use `coverStatus` - never `updateStatus` or `updatedStatus`**

- **`seenStatus`**: Represents the visit status of a site

  - Type: `SeenStatus = "Seen" | "Partial" | "Not Seen"`
  - Indicates whether the site has been visited/inspected

- **`mergedStatus`**: The combined result of `coverStatus` + `seenStatus`
  - Type: `MergedStatus = "Seen" | "Partial Cover" | "Partial Seen" | "Not Seen" | "Not Cover"`
  - Used for display and business logic decisions

### Merged Status Logic

Merged statuses combine coverage availability with visit status. Understanding them is critical:

#### No Coverage (Red) 🔴

- **Meaning**: Cannot see the site because **no new coverage came in today**
- **Cover Status**: `coverStatus = "No"`
- **Visit Status**: Always `seenStatus = "Not Seen"` (cannot visit without coverage)
- **Merged Status**: `"Not Cover"`
- **Display**: Red badge with "אין איסוף" (No Collection)
- **Note**: Currently stored as `"No"` in the database, but understand it means "No Coverage"

#### Partial Coverage (Yellow) 🟡

- **Meaning**: Only **partial coverage came in today**, so sites can only be partially seen
- **Cover Status**: `coverStatus = "Partial"`
- **Visit Status**: Can be either:
  - `seenStatus = "Partial"` → **Merged**: `"Partial Seen"` (Yellow - partially visited)
  - `seenStatus = "Not Seen"` → **Merged**: `"Partial Cover"` (Yellow - waiting for visit)
- **Display**: Yellow badge indicating partial state

#### Full Coverage (Green) 🟢

- **Meaning**: **Full coverage came in today**, allowing complete site inspection
- **Cover Status**: `coverStatus = "Full"`
- **Visit Status**: Can be:
  - `seenStatus = "Seen"` → **Merged**: `"Seen"` (Green - fully visited)
  - `seenStatus = "Partial"` → **Merged**: `"Partial Seen"` (Yellow - partially visited)
  - `seenStatus = "Not Seen"` → **Merged**: `"Not Seen"` (Yellow - waiting for visit)
- **Display**: Green when fully seen, Yellow when waiting or partially done

### Why Merge Statuses?

Merging `coverStatus` and `seenStatus` is important because:

1. **Business Logic**: Determines what actions are available (e.g., can't visit without coverage)
2. **UI Display**: Shows the correct color and text based on both coverage and visit state
3. **User Experience**: Users need to understand both "can I visit?" (coverage) and "have I visited?" (seen)

### Implementation Notes

- Always use `coverStatus` in new code (never `updateStatus` or `updatedStatus`)
- When displaying status, use `mergedStatus` for the final display value
- The `mergedStatusCalculator.ts` service handles the merging logic
- Frontend components should use `getUpdatedStatusDisplay(coverStatus, seenStatus)` for display

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/betterlokaki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
