## mcp-strapi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP Strapi Server is a Model Context Protocol (MCP) server that provides 16 tools for CRUD operations, batch operations, and content management in Strapi CMS. It uses stdio transport to integrate with Claude Desktop and other MCP clients.

**Key Technologies:**

- TypeScript with ESNext modules
- Node.js 22+ (requires native fetch support)
- MCP SDK (@modelcontextprotocol/sdk)
- Zod for schema validation

## Development Commands

```bash
# Build TypeScript to dist/
npm run build

# Development with hot-reload (uses tsx)
npm run dev

# Production (requires build first)
npm start

# Watch mode for TypeScript compilation
npm run watch

# Clean compiled files
npm run clean
```

## Git Commits

When creating commits:
- Write clear, concise commit messages that describe the changes
- Do NOT add "Generated with Claude Code" footers or "Co-Authored-By" signatures
- Use conventional commit format when appropriate (fix:, feat:, docs:, etc.)
- Keep the commit message focused and professional

## Architecture

### Entry Point Flow

1. `src/index.ts` - Runs health check on startup, then creates stdio transport
2. **Health Check** - Validates Strapi connection and credentials before starting
3. `src/server.ts` - Registers all 16 tools and 1 prompt with the MCP server
4. Tools are loaded from `src/tools/` and registered individually

### Health Check (`src/services/strapi-client.ts`)

Runs automatically on server startup to validate Strapi is accessible:

- **URL Validation**: Checks that `STRAPI_URL` has valid format (http/https)
- **Connection Test**: Attempts to connect to Strapi instance via `GET /health` endpoint
- **Clear Error Messages**: Fails early with helpful diagnostics

**Endpoint Used:** `GET /health` (no authentication required)

**Error Cases Handled:**
- Invalid URL format → `Invalid STRAPI_URL format`
- Server unreachable → `Connection refused. Is Strapi running?`
- Any other HTTP error → `HTTP {status}: {statusText}`

If health check fails, the server exits with status code 1 and won't accept connections.

### Critical Components

**StrapiClient (`src/services/strapi-client.ts`)**

- Singleton HTTP client using native fetch (Node.js 22+)
- Handles authentication via `STRAPI_API_TOKEN` env var
- Includes `healthCheck()` method for startup validation
- Builds Strapi-compliant query strings for filters, pagination, populate, etc.
- All errors logged to stderr (stdout reserved for MCP protocol)

**i18n Validator (`src/services/i18n-validator.ts`)**

- Validates content language consistency in multilingual entries
- Detects inherited vs native translations
- Used by read, update, and list tools when locale parameter is specified

### Tool Categories

**CRUD Operations (6 tools):**

- `strapi-create` - Create entry (supports locale)
- `strapi-create-with-locales` - Create entry with multiple languages simultaneously
- `strapi-read` - Read by documentId (supports locale, validateLanguage, strictMode)
- `strapi-list` - List with filters (supports locale, showLocalizationSummary)
- `strapi-update` - Update by documentId (supports locale, validateBeforeUpdate, strictMode)
- `strapi-delete` - Delete by documentId

**Batch Operations (3 tools):**

- `strapi-batch-create` - Create multiple entries in parallel (ideal for migrations and bulk data loading)
- `strapi-batch-update` - Update multiple entries in parallel (supports locale for multilingual updates)
- `strapi-batch-delete` - Delete multiple entries in parallel

**Content Type Management (3 tools):**

- `strapi-list-content-types` - List all available content types
- `strapi-get-schema` - Get detailed schema of a content type
- `strapi-add-field` - Add field to content type (requires content-type-builder permissions)

**Internationalization (1 tool):**

- `strapi-get-i18n-locales` - Get all available locales

**Media Management (3 tools):**

- `strapi-search-media` - Search files in upload library
- `strapi-get-media` - Get specific media file by ID or documentId
- `strapi-upload-media` - Upload file to media library

## Critical Implementation Details

### Content Type Naming Convention

**MOST IMPORTANT:** Different tools use different content type identifiers:

- **CRUD tools** (create, read, list, update, delete) use **PLURAL name**:

  - ✅ `"products"`, `"subcategories"`, `"categories"`
  - ❌ NOT `"api::product.product"`, NOT `"product"`

- **Schema tools** (get-schema, add-field) use **UID format**:
  - ✅ `"api::product.product"`
  - ❌ NOT `"products"`

Use `strapi-list-content-types` to get both formats - check the `pluralName` field for CRUD operations.

### Strapi v5 IDs

- Strapi v5 uses **documentId** (string) as the stable identifier
- All read/update/delete operations use documentId, not numeric id
- Responses include both `id` (number) and `documentId` (string)

### i18n Features

- **locale parameter**: Available in create, read, list, update
- **validateLanguage**: Auto-validates content language (default: true for read)
- **strictMode**: Rejects inherited/fallback content (default: false)
- **showLocalizationSummary**: Shows translation status per entry (default: true for list)

When updating multilingual content, ALWAYS specify the `locale` parameter to avoid accidentally updating the wrong language version.

### Batch Operations

**Overview:**
Batch operations allow processing multiple entries in parallel, significantly reducing execution time for bulk operations:

- `strapi-batch-create` - Creates multiple entries simultaneously
- `strapi-batch-update` - Updates multiple entries simultaneously
- `strapi-batch-delete` - Deletes multiple entries simultaneously

**Key Features:**

1. **Parallel Execution**: All operations run in parallel using Promise.all for maximum performance
2. **Detailed Reporting**: Each operation returns a comprehensive report including:
   - Total entries processed
   - Number of successful/failed operations
   - Individual result for each entry (documentId, error message if failed)
3. **Error Handling**:
   - `continueOnError` parameter (default: true) allows operations to continue even if individual entries fail
   - Set to false to stop on first error
4. **i18n Support**: Batch update supports `locale` parameter for multilingual content
5. **Use Cases**:
   - Migrating content from external sources
   - Bulk data updates and corrections
   - Testing data operations
   - Syncing data across environments

**Example Usage:**

```typescript
// Batch create multiple products
strapi-batch-create({
  contentType: "products",
  entries: [
    { data: { name: "Product 1", price: 10 } },
    { data: { name: "Product 2", price: 20 } },
    { data: { name: "Product 3", price: 30 } }
  ],
  continueOnError: true
})

// Batch update with locale
strapi-batch-update({
  contentType: "articles",
  entries: [
    { documentId: "abc123", data: { title: "Updated Title" }, locale: "en" },
    { documentId: "def456", data: { title: "Título Actualizado" }, locale: "es" }
  ],
  continueOnError: true
})

// Batch delete
strapi-batch-delete({
  contentType: "products",
  documentIds: ["abc123", "def456", "ghi789"],
  continueOnError: true
})
```

### Environment Configuration

**Required:**

- `STRAPI_URL` - Base URL of Strapi instance (default: http://localhost:1337)
  - Must be a valid URL with protocol (http or https)
  - Validated on server startup via health check

**Optional but Recommended:**

- `STRAPI_API_TOKEN` - API token for authentication
  - Required for any authenticated operations (most CRUD operations)
  - Required for content-type-builder operations (add-field, get-schema)
  - Validated on server startup via health check
  - Missing or invalid token will cause server startup to fail

**Optional:**

- `NODE_ENV` - Defaults to production

### Logging Strategy

All logs go to stderr because MCP protocol uses stdout for JSON-RPC communication. Use `console.error()` for all debugging output.

### Known Issues

**strapi-delete error:**

- Returns "JSON input" error after successful deletion
- Entry IS deleted correctly - Strapi returns empty response
- This is expected behavior, not a bug

**Media endpoints (strapi-search-media, strapi-get-media, strapi-upload-media):**

- Use `/api/upload` and `/api/upload/files` endpoints (not Document Service API)
- Return different structure than content type endpoints
- Strapi v5 includes both `id` (numeric) and `documentId` (string) in responses
- `strapi-get-media` accepts either numeric id or documentId as parameter
- `strapi-upload-media` requires absolute file path and uses multipart/form-data

## Testing with Claude Desktop

Add to Claude Desktop config (`%APPDATA%\Claude\claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "strapi": {
      "command": "node",
      "args": ["C:\\absolute\\path\\to\\mcp-strapi\\dist\\index.js"],
      "env": {
        "STRAPI_URL": "http://localhost:1337",
        "STRAPI_API_TOKEN": "your-token-here"
      }
    }
  }
}
```

Restart Claude Desktop after config changes.

## Type System

TypeScript is configured with `strict: false` for flexibility. Key types:

- `CreateParams`, `ReadParams`, `ListParams`, `UpdateParams`, `DeleteParams` - Tool parameters
- `StrapiResponse`, `StrapiListResponse` - API response wrappers
- All types in `src/types/index.ts`

## Adding New Tools

1. Create new file in `src/tools/your-tool.ts`
2. Export `yourToolSchema` (Zod schema) and `yourToolHandler` (async function)
3. Register in `src/server.ts` using `server.registerTool()`
4. Follow existing tool structure for consistency
5. Use `strapiClient` singleton for API calls
6. Log all operations to stderr with `[TOOL NAME]` prefix

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devlitus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
