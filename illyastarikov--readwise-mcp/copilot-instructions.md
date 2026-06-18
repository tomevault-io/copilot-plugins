## readwise-mcp

> MCP server that captures authenticated DOM content from Safari tabs (macOS) and provides full Readwise library management via 16 tools. On non-macOS platforms, Safari capture tools are unavailable but all Readwise API tools work, and `capture-page` auto-falls back to direct URL save.

# Safari-Readwise MCP Server

MCP server that captures authenticated DOM content from Safari tabs (macOS) and provides full Readwise library management via 16 tools. On non-macOS platforms, Safari capture tools are unavailable but all Readwise API tools work, and `capture-page` auto-falls back to direct URL save.

## Quick Reference

```bash
npm run build        # Build with tsup → dist/
npm run dev          # Run with tsx (dev mode)
npm run test         # Run all tests (vitest)
npm run test:watch   # Watch mode
npm run typecheck    # tsc --noEmit
```

## Architecture

```
src/
├── index.ts              # Entry point — stdio transport
├── server.ts             # MCP server — registers all 16 tools
├── readwise/
│   ├── shared.ts         # getToken(), handleApiResponse() — used by both API modules
│   ├── reader-api.ts     # Reader v3 API: auth, save, list/update/delete documents, tags
│   ├── classic-api.ts    # Classic v2 API: highlights CRUD, books, export, daily review
│   ├── rate-limiter.ts   # Global fetch queue: 1.3s spacing + 429 auto-retry
│   ├── client.ts         # Barrel re-exports (backward compat for existing imports)
│   ├── types.ts          # Reader v3 types
│   └── classic-types.ts  # Classic v2 types
├── safari/
│   ├── applescript.ts    # osascript runner with error classification
│   ├── diagnostics.ts    # Permission/setup checks
│   ├── dom-capture.ts    # Captures full DOM via Safari JavaScript bridge (+ open-and-capture for URLs)
│   └── tab-list.ts       # Lists open Safari tabs via AppleScript
├── tools/                # One file per MCP tool (schema + handler)
│   ├── list-tabs.ts
│   ├── check-setup.ts
│   ├── capture-page.ts
│   ├── capture-tabs.ts
│   ├── capture-urls.ts
│   ├── list-documents.ts
│   ├── update-document.ts
│   ├── delete-document.ts
│   ├── bulk-update-documents.ts
│   ├── list-tags.ts
│   ├── create-highlights.ts
│   ├── list-highlights.ts
│   ├── manage-highlight.ts
│   ├── list-books.ts
│   ├── export-highlights.ts
│   └── daily-review.ts
└── utils/
    ├── errors.ts         # Error classes + errorToToolResult helper
    ├── logger.ts         # stderr logger
    ├── platform.ts       # isMacOS() — platform detection for Safari tool guards
    └── temp-file.ts      # Temp file for large DOM transfers
```

## Key Patterns

- **Tool structure**: Each tool exports `{toolName}Schema` (Zod shape object) and `{toolName}Handler` (async function). Both registered in `server.ts`.
- **Error handling**: All tool handlers wrap logic in try/catch and return `errorToToolResult(error)` on failure. API layer throws `ReadwiseTokenError` (401/missing token) or `ReadwiseApiError` (429/other HTTP errors).
- **API response handling**: `shared.ts:handleApiResponse()` is the single error handler for all API calls. `saveDocument` is the exception — it checks 200/201 itself, then delegates non-2xx to `handleApiResponse`.
- **Pagination**: v3 tools use cursor-based (`pageCursor`/`nextPageCursor`), v2 tools use page-based (`page`/`page_size`, `next`/`previous`).
- **Output format**: All tools return human-readable text. No JSON responses — the text is structured enough for LLMs to parse.
- **Barrel re-exports**: `client.ts` re-exports from `shared.ts`, `reader-api.ts`, and `classic-api.ts` so any function can be imported via `"./readwise/client.js"`.
- **Rate limiting**: `rate-limiter.ts` exports `rateLimitedFetch()` — a drop-in `fetch` wrapper that queues all Readwise API requests sequentially with 1.3s spacing (~46 req/min). On 429, retries up to 3 times using the `Retry-After` header. All API functions in `reader-api.ts` and `classic-api.ts` use it instead of bare `fetch`.

## Environment

- **READWISE_TOKEN** (required): API token from https://readwise.io/access_token
- macOS required for Safari tools (DOM capture, tab listing); Readwise API tools work on any platform
- Node >= 18 (uses native `fetch`)

## Testing

Tests are in `tests/` mirroring `src/` structure. All API calls are mocked via `vi.fn()` on `global.fetch` or `vi.mock()`. No real API calls in tests.

- 26 test files, 147 tests
- API layer tests verify URL construction, headers, body serialization, and error mapping
- Tool handler tests verify output formatting, empty states, pagination display, and error propagation

## Tools (16 total)

### Safari (5) — macOS only (except `capture-page`/`capture-urls` with URLs, which work cross-platform via directSave fallback)
| Tool | Description |
|------|-------------|
| `list-tabs` | List open Safari tabs (macOS only) |
| `check-setup` | Diagnostic check — Safari permissions (macOS) + Readwise token |
| `capture-page` | Capture a page DOM → Readwise Reader. On non-macOS, auto-falls back to directSave |
| `capture-tabs` | Capture multiple tabs → Readwise Reader (macOS only) |
| `capture-urls` | Capture a list of URLs → Readwise Reader. Uses Safari DOM capture on macOS, direct save on other platforms |

### Reader v3 (5)
| Tool | Description |
|------|-------------|
| `list-documents` | Search/filter documents (location, category, tag, date) |
| `update-document` | Update document metadata |
| `delete-document` | Delete a document |
| `bulk-update-documents` | Update up to 50 documents at once |
| `list-tags` | List all Reader tags |

### Classic v2 (6)
| Tool | Description |
|------|-------------|
| `create-highlights` | Create highlights (batch) with book metadata |
| `list-highlights` | List/get highlights (page-based pagination) |
| `manage-highlight` | Update or delete a highlight |
| `list-books` | List/get books/sources |
| `export-highlights` | Export highlights with full book metadata (cursor pagination) |
| `daily-review` | Get today's daily review highlights |

## Design Decisions & Trade-offs

- **`validateToken()` pre-check in capture tools**: `capture-page` and `capture-tabs` call `validateToken()` before `saveDocument()`. This is an extra network round-trip since `saveDocument` already throws `ReadwiseTokenError` on 401, but it provides a friendlier error message before attempting DOM capture. Kept intentionally.
- **`saveDocument` special 200/201 handling**: Unlike all other API functions that delegate entirely to `handleApiResponse()`, `saveDocument` checks 200 (already exists) and 201 (created) itself, then delegates non-2xx to `handleApiResponse`. This is because it needs to distinguish between the two success statuses to set `alreadyExists`.
- **`list-documents` schema `withHtmlContent` default**: `.optional().default(false)` is technically unnecessary since the API layer treats `undefined` as falsy, but it's harmless and makes the default explicit in the schema.
- **Global rate limiter vs per-tool delays**: All Readwise API calls go through `rateLimitedFetch()` which serializes requests with 1.3s spacing. This handles concurrent MCP tool calls (e.g., 50 parallel `delete-document` calls) gracefully. `capture-tabs.ts` no longer needs its own delay logic.
- **Platform-aware Safari tools**: `isMacOS()` from `utils/platform.ts` guards Safari-dependent code. On non-macOS: `capture-page` and `capture-urls` auto-fall back to `directSave` (URL-only, Readwise fetches content); `list-tabs`, `capture-tabs` return clear errors; `check-setup` skips Safari diagnostics but still checks the Readwise token. All 16 tools remain registered on all platforms.

---
> Source: [IllyaStarikov/readwise-mcp](https://github.com/IllyaStarikov/readwise-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
