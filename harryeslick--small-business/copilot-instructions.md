## small-business

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Small business accounting and job management system for Australian sole traders, built with Python 3.13+. The backend is feature-complete with models, storage, bank imports, classification, workflows, reports, and document generation (277 tests passing). The Textual TUI frontend exposes all backend capabilities through 10 screens, with a command palette, keyboard-driven navigation, and custom widgets.

## Development Commands

### Environment Setup
```bash
# Setup virtual environment and install dependencies
uv sync

# Install pre-commit hooks
pre-commit install-hooks
```

### Testing
```bash
# Run all tests
uv run pytest

# Run tests with coverage
uv run pytest --cov

# Run a single test file
uv run pytest tests/path/to/test_file.py

# Run a specific test function
uv run pytest tests/path/to/test_file.py::test_function_name
```

### Code Quality
```bash
# Run ruff linter
uv run ruff check .

# Run ruff linter with auto-fix
uv run ruff check --fix .

# Run ruff formatter
uv run ruff format .

# Run all pre-commit hooks manually
pre-commit run --all-files
```

### Documentation
```bash
# Serve documentation locally
mkdocs serve

# Build documentation
mkdocs build

# Deploy documentation to GitHub Pages
mkdocs gh-deploy
```

## Code Style & Formatting

- **Line length**: 100 characters
- **Indentation**: Tabs (width 4)
- **Quote style**: Double quotes
- **Python version**: 3.13+
- **Linter/Formatter**: ruff (replaces black, flake8, isort)

## Architecture & Structure

### Project Layout
```
src/small_business/     # Main package source code
  models/               # Pydantic models (Client, Quote, Job, Invoice, Account, Transaction, etc.)
  storage/              # In-memory StorageRegistry with JSONL/JSON disk persistence
  bank/                 # Bank statement CSV import pipeline
  classification/       # Rule-based transaction classification engine
  reports/              # Financial reporting (Balance Sheet, P&L, BAS/GST) with typed models
  documents/            # DOCX document generation via Jinja2 templates
  workflows.py          # Entity lifecycle transitions (Quote→Job→Invoice)
  init_business.py      # Business directory structure initialization
  tui/                  # Textual TUI frontend (10 screens + widgets + modals)
    app.py              # SmallBusinessApp — main App class, StorageRegistry, reactive counts
    __main__.py         # CLI entry point (`uv run small-business [DATA_DIR]`)
    commands.py         # Command palette providers (NavigationProvider, ActionProvider, EntityProvider)
    utils.py            # Formatting helpers (currency, dates, financial year)
    screens/            # One file per screen (dashboard, clients, quotes, jobs, invoices, etc.)
    widgets/            # Custom widgets (CurrencyDisplay, SparklinePanel, NotificationBar, etc.)
    modals/             # Modal dialogs (setup wizard, confirm, client form, line item editor)
    styles/             # TCSS stylesheets (app theme, dashboard layout, etc.)
docs/                   # MkDocs documentation
  notebooks/            # Jupyter notebooks for examples (executed in docs)
  api_docs/             # API reference documentation
  plans/                # Design documents for each phase
tests/                  # pytest test suite (277 tests)
  models/               # Tests for Pydantic models
  storage/              # Tests for StorageRegistry
  bank/                 # Tests for bank import pipeline
  classification/       # Tests for classification engine
  reports/              # Tests for financial reports
  documents/            # Tests for document generation
  integration/          # End-to-end integration tests
```

### Key Technologies
- **Textual 8.0**: Terminal UI framework — 10 screens, command palette, custom widgets, TCSS styling
- **uv**: Package and dependency management (replaces pip/poetry)
- **Pydantic v2**: Data validation, modeling, and typed report return types
- **pandas**: CSV/data manipulation for bank imports and report exports
- **docxtpl**: DOCX template rendering via Jinja2
- **PyYAML**: Chart of accounts and classification rule storage
- **pytest**: Testing framework with code coverage (277 tests)
- **MkDocs Material**: Documentation with mkdocs-jupyter for executable notebooks
- **pre-commit**: Code quality enforcement with ruff and codespell

### Documentation
MkDocs is configured to execute Jupyter notebooks during build (via mkdocs-jupyter plugin). Python files in `docs/notebooks/` are treated as notebooks and executed when building documentation.

## Development Notes

### General
- Use absolute imports (configured in pytest with `pythonpath = ["src"]`)
- Version is managed in `src/small_business/__init__.py` via hatchling
- Pre-commit excludes `dev.py` and `docs/*.py` files from checks
- Tests should have short docstrings describing what is being tested
- Use relative dates in tests (`date.today() + timedelta(...)`) — never hardcode dates that will expire

### Data Models
The project uses Pydantic models for all core entities. Key design patterns:

- **Decimal for money**: All monetary values use `Decimal` for precision
- **String IDs for relationships**: Simple string IDs (not nested objects) for serialization
- **Computed fields**: Automatic calculations for totals, GST, financial year using `@computed_field`
- **Date-driven status**: Quote, Job, Invoice statuses are computed from date fields, NOT stored
- **Model validators**: Data integrity validation (e.g., balanced transactions, account hierarchy)
- **Double-entry accounting**: Transaction model enforces debits = credits

See `docs/plans/2025-11-15-phase1-datastructures-design.md` for complete architecture details.

### Storage Layer
`StorageRegistry` is the central data access layer (src/small_business/storage/registry.py). Key patterns:

- **In-memory with disk persistence**: All data loaded at init, queries served from memory
- **JSONL for append-only data**: Clients, transactions (compacted on load)
- **Versioned JSON for entities**: Quotes, invoices, jobs stored as `{FY}/{type}/{ID}_v{N}.json`
- **Financial year organized**: Data directories organized by Australian FY (July–June, e.g., `2025-26`)
- **Full CRUD**: Clients, Quotes, Invoices, Jobs, Transactions, Settings, ChartOfAccounts, BankFormats
- **Transaction operations**: delete, void (reversing entry), search, filter by date/account/amount
- **Convenience methods**: `get_unclassified_transactions()`, `get_account_codes()`, `search_transactions()`

### Workflow Service
`src/small_business/workflows.py` orchestrates entity lifecycle transitions:

- `accept_quote_to_job(quote_id, data_dir)` — validates SENT quote, creates linked Job
- `complete_job_to_invoice(job_id, data_dir)` — validates COMPLETED job, creates Invoice from quote line items

### Reports
Report generators return typed Pydantic models (not dicts):
- `generate_balance_sheet()` → `BalanceSheetReport`
- `generate_profit_loss_report()` → `ProfitLossReport`
- `generate_bas_report()` → `BASReport`

Report models are in `src/small_business/reports/models.py` and exported from `reports/__init__.py`.

### TUI Frontend
The Textual TUI (`src/small_business/tui/`) is a pure consumer of the backend APIs. Key patterns:

- **Single shared StorageRegistry**: Instantiated once on `SmallBusinessApp`, accessed by all screens via `self.app.storage`
- **Reactive counts**: `unclassified_count` and `overdue_count` on App trigger UI updates across screens
- **Data refresh**: After any mutation, screens call `self.app.action_data_changed()` which reloads storage and updates reactive counts
- **Screen registration**: `SCREENS` dict maps names to classes (lazy instantiation). Navigate with `app.switch_screen('name')` or number keys `1`–`9`
- **Command palette**: Three providers — `NavigationProvider` (screen nav), `ActionProvider` (create/import/generate), `EntityProvider` (search clients/quotes/jobs/invoices by ID/name)
- **TCSS styling**: Separate `.tcss` files in `styles/` for theme and layout. Semantic colour classes: `.money-positive`, `.money-negative`, `.status-overdue`, etc.
- **Workflow integration**: Screens call `accept_quote_to_job()` and `complete_job_to_invoice()` then `storage.reload()` since workflow functions create their own StorageRegistry
- **Console entry point**: `small-business` (from within business dir), `small-business init` (setup wizard), or `small-business /path` (explicit dir)
- **Global install**: `uv tool install small-business` — installs as a system-wide CLI tool
- **Business detection**: `_is_business_dir()` checks for `config/settings.json` or `settings.json`

**Known issue**: Textual 8.0.0's `run_test()` prints a `get_height` traceback during test cleanup — this is a Textual internal bug during widget measurement on shutdown, not a code issue. The app runs correctly at runtime.

## API Design Principles

These principles guide the design of all modules and should be applied consistently across the codebase.

### 1. Auto-Lookup Dependencies
High-level functions should automatically load required dependencies using `data_dir` rather than requiring callers to pass everything explicitly.

**Pattern:**
```python
# Good - auto-loads dependencies
generate_quote_document(quote, output_path, data_dir)

# Avoid - requires caller to load everything
client = load_client(quote.client_id, data_dir)
settings = load_settings(data_dir)
generate_quote_document(quote, client, settings, output_path)
```

**Rationale:** Reduces boilerplate, centralizes loading logic, makes APIs easier to use correctly.

### 2. Sensible Defaults for Optional Parameters
Optional parameters should default to the most common use case rather than requiring explicit values.

**Pattern:**
```python
# Good - defaults to latest version (most common)
load_quote(quote_id, data_dir, version=None)

# Avoid - always requiring version even when latest is wanted
load_quote(quote_id, data_dir, version=get_latest_version(quote_id))
```

**Rationale:** Reduces cognitive load, makes common cases simple while keeping advanced cases possible.

### 3. Human-Readable Identifiers Where Appropriate
Use business-meaningful identifiers for entities that have natural unique keys.

**Pattern:**
```python
# Good - business name is naturally unique and meaningful
client_id = "Woolworths"

# Avoid - generated ID is opaque and meaningless
client_id = "C-20251116-001"
```

**When to use:**
- **Human-readable IDs**: Entities with natural unique keys (client business names)
- **Generated IDs**: Entities needing audit trails or without natural keys (quotes, jobs, invoices)

**Rationale:** Improves debuggability, makes data more understandable, reduces need for lookups.

### 4. Case-Insensitive Lookups for User-Provided IDs
When IDs are user-provided strings (like business names), implement case-insensitive matching.

**Pattern:**
```python
# Normalize for comparison
normalized_id = client_id.lower()
for client in clients:
    if client.client_id.lower() == normalized_id:
        return client
```

**Rationale:** More forgiving for user input, prevents duplicate records due to case differences.

### 5. Template-Based Document Generation
Use template systems (like Jinja2) for document generation rather than programmatic assembly.

**Pattern:**
```python
# Good - use templates
doc = DocxTemplate(template_path)
doc.render(context)

# Avoid - programmatic assembly
doc = Document()
doc.add_heading("Quote")
doc.add_paragraph(f"Date: {quote.date}")
# ... many more doc.add_* calls
```

**Rationale:** Separates presentation from logic, enables non-developers to modify layouts, easier to test.

### 6. Settings-Based Configuration
Store configurable values in the Settings model rather than hard-coding or requiring as function arguments.

**Pattern:**
```python
# Good - configured in Settings
settings = load_settings(data_dir)
template_path = settings.quote_template_path

# Avoid - hard-coded paths
template_path = "templates/quote.docx"

# Avoid - requiring as parameter everywhere
def generate_quote(quote, template_path, output_path): ...
```

**Rationale:** Centralizes configuration, enables customization without code changes, reduces parameter sprawl.

### 7. Structured Data with Computed Display Fields
For complex data like addresses, store both structured fields (for validation/formatting) and computed display fields.

**Pattern:**
```python
class Client(BaseModel):
    # Structured fields
    street_address: str | None
    suburb: str | None
    state: str | None
    postcode: str | None

    # Computed display field
    formatted_address: str | None
```

**Rationale:** Structured fields enable validation and flexible formatting; display fields provide convenience.

## Applying These Principles

When implementing new features:

1. **Check for auto-lookup opportunities** - Can the function load its own dependencies?
2. **Consider sensible defaults** - What's the most common use case?
3. **Evaluate ID strategies** - Natural unique key or generated ID?
4. **Plan for user input** - Should lookups be case-insensitive?
5. **Separate presentation from logic** - Use templates for documents/reports
6. **Centralize configuration** - Settings model vs. hard-coding vs. parameters
7. **Balance structure and display** - Store structured data with computed display fields

See `docs/plans/2025-11-23-phase4-revisions-design.md` for detailed examples of these principles in practice.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/harryeslick)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/harryeslick)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
