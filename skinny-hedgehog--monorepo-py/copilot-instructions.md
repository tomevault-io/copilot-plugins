## monorepo-py

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a polyglot monorepo implementing an event-sourced ledger system with CQRS pattern. The project demonstrates:
- Event sourcing with DynamoDB as the event store
- CQRS with PostgreSQL read models
- OpenTelemetry instrumentation for performance monitoring
- React + Tailwind CSS web UI for ledger management

## Workspace Structure

UV workspace with two Python packages and one Node.js package:
- `packages/sh_dendrite/` - Event sourcing framework library (installable Python package)
- `packages/api/` - FastAPI application (Python application, not installable)
- `packages/web/` - React + Tailwind frontend (Node.js, uses pnpm)

**Important:** The `api` package is structured as an application with `src/domain/`, `src/routes/`, etc. It has its build-system commented out intentionally and relies on `conftest.py` to add `src/` to the Python path for tests.

**Frontend Architecture:** The `web` package uses Vite as the build tool and proxies API requests to the FastAPI backend during development. It connects to `localhost:8000` (FastAPI) via Vite's proxy configuration.

## Development Commands

### Testing
```bash
# Run all tests from root
uv run pytest -v

# Run tests for specific package
cd packages/api && uv run pytest -v
cd packages/sh_dendrite && uv run pytest -v

# Run single test file
uv run pytest packages/api/tests/setup_account/test_create_account_new.py -v
```

### Running the API
```bash
# From packages/api directory
uv run --env-file .env fastapi dev src/main.py
```

### Running the Web Application

The web application requires both the backend API and frontend dev server to be running.

**Terminal 1 - Backend API:**
```bash
cd packages/api
uv run fastapi dev src/main.py
```

**Terminal 2 - Frontend Dev Server:**
```bash
cd packages/web
pnpm install  # First time only
pnpm dev
```

The frontend will be available at `http://localhost:3000` and will proxy API requests to `http://localhost:8000`.

### Frontend Package Management

```bash
# Install dependencies
cd packages/web
pnpm install

# Add a new package
pnpm add <package-name>

# Add a dev dependency
pnpm add -D <package-name>

# Update dependencies
pnpm update

# Build for production
pnpm build  # Creates production build in dist/
pnpm preview  # Preview production build locally
```

### Workspace Management
```bash
# Sync all dependencies
uv sync

# Update lockfile
uv lock
```

### Performance Monitoring
Run Aspire dashboard for OpenTelemetry:
```bash
docker run --rm -it \
  -p 18888:18888 \
  -p 4317:18889 \
  --name aspire-dashboard \
  mcr.microsoft.com/dotnet/aspire-dashboard:latest
```

## Frontend Architecture

### Structure

```
packages/web/
├── src/
│   ├── api/
│   │   └── ledgerClient.js       # API client for backend communication
│   ├── components/
│   │   ├── CreateLedger.jsx      # Create new ledger
│   │   ├── LedgerView.jsx        # Main ledger view container
│   │   ├── LedgerBalance.jsx     # Display ledger balance
│   │   └── TransactionForm.jsx   # Credit/debit form
│   ├── App.jsx                   # Root component
│   └── main.jsx                  # React entry point
├── vite.config.js                # Vite config with proxy
└── tailwind.config.js            # Tailwind CSS config
```

### API Integration

The frontend uses the native Fetch API to communicate with the backend. The API client (`src/api/ledgerClient.js`) provides methods for:

- `createLedger()` - Create a new ledger with $500 initial balance
- `getLedger(ledgerId)` - Get current ledger balance
- `creditLedger(ledgerId, amount)` - Add credit to ledger
- `debitLedger(ledgerId, amount)` - Add debit to ledger

Vite's proxy configuration ensures all requests to `/ledger/*` and `/account/*` are forwarded to the FastAPI backend at `localhost:8000`. This eliminates CORS issues in development.

### Component Design

- **App.jsx**: Root component managing currentLedgerId state, handles routing between CreateLedger and LedgerView
- **CreateLedger.jsx**: Button component that calls API to create ledger, handles loading/error states
- **LedgerView.jsx**: Container that fetches ledger data and orchestrates LedgerBalance + TransactionForm
- **LedgerBalance.jsx**: Presentational component displaying ledger ID and current balance
- **TransactionForm.jsx**: Form with amount input and credit/debit buttons, refreshes parent on success

## Event Sourcing Architecture

### Core Concepts

**AggregateFactory** (`sh_dendrite.aggregate_factory`):
- Creates new aggregates with `new(AggregateType)` → generates UUID log_id
- Loads existing aggregates with `load(AggregateType, log_id)` → rehydrates from event store
- Wires up event_store, log_id_generator, and event_handlers to aggregates

**Aggregate** (`sh_dendrite.aggregate`):
- Base class for domain aggregates (e.g., Ledger, FamilyAccount)
- `apply(event)` method: persists event → updates aggregate state → dispatches to event handlers
- Must implement `on(event)` for state transitions (match/case pattern on event type)
- Has `log_id` (aggregate ID), `event_store`, and `event_handlers` dict

**Event Flow**:
1. Command arrives at route (e.g., `POST /ledger/{id}/credits`)
2. Route loads aggregate via factory: `aggregate_factory.load(Ledger, ledger_id)`
3. Route invokes command method: `ledger.credit(CreditLedgerCommand(amount))`
4. Command method creates event and calls: `self.apply(LedgerCreditedEvent(...))`
5. Apply chain:
   - Persists to event store (DynamoDB)
   - Updates aggregate state via `self.on(event)`
   - Dispatches to registered event handlers (e.g., LedgerReadModel → PostgreSQL)

**Event Handlers** (`sh_dendrite.event_handler`):
- Registered in main.py: `event_handlers = {EventType: [handler1, handler2]}`
- Called synchronously after event is applied
- Used for read model updates (e.g., denormalized PostgreSQL tables)

### Domain Pattern

Each domain aggregate follows this structure:
```
domain/
  ledger.py
    - Commands: CreateLedgerCommand, CreditLedgerCommand, DebitLedgerCommand
    - Events: LedgerCreatedEvent, LedgerCreditedEvent, LedgerDebitEvent
    - Aggregate: Ledger (extends Aggregate, implements on(event))
    - Read Model: LedgerReadModel (implements EventHandler)
```

Events must be dataclasses that extend `sh_dendrite.event.Event`.

### Router Pattern

Routes use a class-based pattern:
- Constructor receives `AggregateFactory`
- `_register_routes()` wires route methods to paths
- Route methods use factory to `new()` or `load()` aggregates
- Returns `get_router()` for FastAPI inclusion

## Environment Variables

Required for API:
```
EVENT_STORE_TABLE_NAME - DynamoDB table name
AWS_REGION - AWS region
AWS_PROFILE - AWS credential profile
RM_DB_NAME - PostgreSQL database name
RM_DB_USER - PostgreSQL user
RM_DB_PASSWORD - PostgreSQL password
RM_DB_HOST - PostgreSQL host
```

## Testing Patterns

Test structure for `api`:
- `tests/` directory imports from `domain/`, `routes/`, `setup_account/` via conftest.py
- Use `SingleLogEventStore` (in-memory) for testing event sourcing
- Tests wire up `AggregateFactory` with test event store and no handlers

## Performance Considerations

The system is designed for aggregate rehydration performance:
- Target: < 200ms total (including 100ms network overhead)
- Stress test data available with pre-seeded ledgers (10, 100, 1K, 10K, 100K events)
- OpenTelemetry spans instrument: event_store operations, aggregate loading, event replay, event handlers

## Package Naming

Use `sh_dendrite` (underscore) consistently in imports and pyproject.toml dependencies. Python normalizes `sh-dendrite` and `sh_dendrite` automatically but consistency is preferred.

---
> Source: [skinny-hedgehog/monorepo-py](https://github.com/skinny-hedgehog/monorepo-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
