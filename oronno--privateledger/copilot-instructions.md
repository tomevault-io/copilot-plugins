## privateledger

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PrivateLedger is a local-only personal finance application built in Go. It works with exported bank transactions in standard OFX (Microsoft Money) or QFX (Intuit Quicken) formats, provides pattern-based automatic categorization, and generates spending insights - all while keeping all financial data private on the user's machine.

**Core Philosophy**: Privacy-first, single binary, no cloud dependencies.

## Common Commands

### Building and Running
```bash
# Build the binary
make build

# Build for all platforms (Linux, macOS, Windows)
make build-all

# Run the application
make run
# Or directly: ./privateledger

# Run tests
make test

# Clean build artifacts and reset local data
make clean
```

### Development
```bash
# Run from source (for development)
go run ./cmd/privateledger

# Install dependencies
go mod tidy

# Run a specific test
go test -v ./internal/service -run TestCategorizer
```

## Architecture

### High-Level Structure

The application follows a clean architecture pattern with clear separation of concerns:

```
cmd/privateledger/          → Entry point, dependency wiring, embedded assets
internal/
├── config/          → Configuration management (config.json)
├── database/        → SQLite connection, schema migrations
├── model/           → Domain models (Account, Transaction, Category, etc.)
├── repository/      → Data access layer (SQL queries)
├── service/         → Business logic (import, categorization, insights)
├── handler/         → HTTP handlers (Gin routes)
├── parser/          → OFX file parsing
├── middleware/      → HTTP middleware (logging)
└── logger/          → Structured logging setup
```

### Key Architectural Patterns

1. **Dependency Injection**: All dependencies are initialized in `main.go` and injected into constructors. No global state or singletons.

2. **Repository Pattern**: Database operations are isolated in `repository/` layer. Services never interact with the database directly.

3. **Service Layer**: Business logic lives in `service/`:
   - `import_service.go` - OFX/QFX parsing, deduplication, transaction insertion
   - `categorizer.go` - Pattern matching for auto-categorization
   - `insights_service.go` - Monthly aggregations, trend calculations

4. **Embedded Assets**: HTML templates and static files are embedded using `go:embed` in `cmd/privateledger/embed.go`. This enables single-binary distribution.

### Database Schema

Uses SQLite with the following core tables:
- `account` - User-defined bank accounts
- `ledger_transaction` - All transactions (note: avoided reserved keyword "transaction")
- `category` - Spending categories with `category_type` (General/Expense/Income)
- `category_pattern` - Pattern matching rules for auto-categorization
- `import_batch` - Tracks import history for each OFX file

**Critical Constraint**: Deduplication uses composite unique key:
```sql
UNIQUE(account_id, trn_type, fit_id, date_posted)
```

### Transaction Flow

**Import Flow**:
1. User uploads OFX or QFX file via `/api/import` and selects target account
2. `import_service.go` parses OFX/QFX using `ofxgo` library
3. For each transaction:
   - Merge `NAME` + `MEMO` fields into `transaction_details`
   - Derive `transaction_type` (1=debit, 2=credit) from `trn_type` and amount sign
   - Check deduplication constraint
   - If new: apply categorization patterns and insert
4. Create `import_batch` record tracking stats
5. Return summary: imported count, duplicates, auto-categorized count

**Categorization**:
- Pattern matching uses case-insensitive "contains" search
- First matching pattern wins
- `category_source` tracks origin: 0=none, 1=rule, 2=manual
- Manual categorizations (`category_source=2`) are **never** overwritten by rules
- Re-categorization only affects transactions with `category_source=0`

### Custom Month Logic

The application supports custom month boundaries via `start_of_month` config (1-28). This allows aligning with pay cycles:
- `start_of_month: 19` means "December 2025" = Dec 19, 2025 → Jan 18, 2026
- Implemented in `insights_service.go` for all financial calculations

## Frontend Architecture

**Tech Stack**: Bootstrap 5 + HTMX (no build step, no SPA)

**Templates**: Located in `cmd/privateledger/web/templates/`
- `layout.html` - Base template with navigation
- Page templates: `dashboard.html`, `transactions.html`, `categories.html`, etc.
- HTMX enables dynamic updates without full page reloads

**Routing**:
- Page routes: `GET /`, `GET /transactions`, etc. (return HTML)
- API routes: `GET /api/transactions`, `POST /api/import`, etc. (return JSON)

See `API_ROUTES.md` for complete endpoint documentation.

## OFX/QFX Parsing

### Format Compatibility
Both OFX (Microsoft Money) and QFX (Intuit Quicken) formats are supported. QFX is essentially OFX with minor variations, and both use the same underlying SGML/XML structure. Most banks in the US, Canada, and UK offer either OFX or QFX download options.

### Supported Banks
Tested with Canadian banks: TD, RBC, CIBC. Compatible with most US and UK banks that offer OFX/QFX downloads.

### Known Quirks
- TD uses `<NAME>` tag, sometimes lowercase `<n>`
- Date formats vary: `20251215020000[-5:EST]`, `20251215120000.000[-5]`
- RBC includes location info in `MEMO`
- CIBC includes payment context in `MEMO`
- XML entities must be handled (e.g., `&amp;`)

### Transaction Type Derivation
```go
func deriveTransactionType(trnType string, amount float64) int {
    switch strings.ToUpper(trnType) {
    case "DEBIT":
        return 1
    case "CREDIT":
        return 2
    default:
        if amount >= 0 {
            return 2 // credit
        }
        return 1 // debit
    }
}
```

## Configuration

File: `config.json` (created beside binary on first run)

Key settings:
- `server.port` - HTTP server port (default: 8844)
- `server.auto_open_browser` - Open browser on startup
- `start_of_month` - Custom month start day (1-28, default: 1)
- `logging.enable_file_logging` - Write logs to `server.log`
- `logging.log_level` - "debug", "info", "warn", "error"
- `debug_mode` - Enable Gin debug mode

## Important Implementation Notes

### When Working with Transactions
- Always use the composite key for deduplication checks
- Never overwrite `category_source=2` (manual) during re-categorization
- Transaction type (1=debit, 2=credit) must be derived, not directly from OFX

### When Working with Categories
- Patterns are globally unique across all categories
- Adding new patterns triggers automatic re-categorization of uncategorized transactions
- Category deletion sets related transactions to `category_id=NULL`, `category_source=0`

### When Working with the Database
- Use parameterized queries (already standard in repositories)
- Table name is `ledger_transaction`, not `transaction` (SQL reserved keyword)
- All foreign keys have appropriate `ON DELETE` actions (CASCADE or SET NULL)

### When Adding New Features
- Follow the existing layering: handler → service → repository
- Initialize dependencies in `main.go` and inject via constructors
- Use structured logging (`slog`) with appropriate context
- API endpoints should follow RESTful conventions (see `API_ROUTES.md`)

### Logging
- Use `slog` for all logging (not `log` or `fmt.Println`)
- Log levels: Debug (verbose), Info (important events), Warn (recoverable issues), Error (failures)
- HTTP request/response logging is handled by `middleware.LoggingMiddleware()`

## Cross-Platform Considerations

- Uses `modernc.org/sqlite` (pure Go, no CGO) for cross-compilation
- File paths use `filepath.Join()` for OS compatibility
- Browser auto-open logic handles Linux, macOS, Windows in `main.go:openBrowser()`

## Testing

- Test files use `_test.go` suffix
- Place test OFX/QFX files in `testdata/` directory (gitignored)
- Run specific package tests: `go test -v ./internal/parser`

---
> Source: [oronno/privateledger](https://github.com/oronno/privateledger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
