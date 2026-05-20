## ux-datatables

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Symfony UX bundle (`pentiminax/ux-datatables`) integrating DataTables.net with server-side and client-side rendering. PHP 8.3+, Symfony 7.0/8.0. Frontend uses a Stimulus controller (TypeScript).

## Commands

```bash
# Install dependencies
composer install

# Run tests
vendor/bin/phpunit

# Run a single test
vendor/bin/phpunit tests/Unit/Path/To/TestFile.php

# Fix code style (PSR-12 + Symfony rules)
composer fix

# Audit dependencies
composer audit
```

## Architecture

**Namespace**: `Pentiminax\UX\DataTables\` → `src/`
**Test namespace**: `Pentiminax\UX\DataTables\Tests\` → `tests/`

### Core Data Flow

```
Twig template (render_datatable) → Stimulus controller (assets/src/controller.ts)
  → Lazy-loads DataTables.net + extensions
  → Server-side: AJAX → DataTableRequest → QueryFilterChain → Doctrine ORM → DataTableResponseBuilder → JSON
  → Client-side: Data embedded directly
```

### Key Components

- **AbstractDataTable** (`src/Model/AbstractDataTable.php`): Base class users extend. Configured with `#[AsDataTable]` attribute. Defines columns, data provider, and extensions via `configure()` method.
- **DataTable** (`src/Model/DataTable.php`): Immutable configuration object with fluent builder API — `columns()`, `ajax()`, `serverSide()`, `responsive()`, etc.
- **Column system** (`src/Column/`): 12 column types extending `AbstractColumn` — TextColumn, NumberColumn, DateColumn, BooleanColumn, ActionColumn, etc. Serialized via `ColumnDto`.
- **Data Providers** (`src/DataProvider/`): `DoctrineDataProvider` (ORM QueryBuilder integration) and `ArrayDataProvider` (in-memory). Implement `DataProviderInterface`.
- **Query pipeline** (`src/Query/`): Chain-of-responsibility pattern. `QueryFilterChain` runs filters (GlobalSearchFilter, ColumnSearchFilter, OrderFilter) using 14 search strategies (Contains, StartsWith, Equal, GreaterThan, etc.).
- **Extensions** (`src/Model/Extensions/`): Buttons, Select, Responsive, Scroller, KeyTable, ColReorder, FixedColumns, ColumnControl. Each implements `ExtensionInterface`.
- **API Platform integration** (`src/ApiPlatform/`): Optional. Auto-detects columns from entity metadata, maps property types, resolves collection URLs.
- **Stimulus controller** (`assets/src/controller.ts`): Frontend entry point. Lazy-loads DataTables.net and extensions, handles delete actions and boolean toggling.
- **Maker** (`src/Maker/`): `make:datatable` Symfony console command to scaffold new DataTable classes.

### Patterns Used

- **Builder pattern**: DataTable fluent configuration
- **Chain of responsibility**: QueryFilterChain for query filtering/sorting
- **Strategy pattern**: SearchStrategy implementations in `src/Query/Strategy/`
- **Attributes**: `#[AsDataTable]` for service metadata
- **Generators**: Memory-efficient result iteration in DataTableResult

## Code Style

Enforced by PHP-CS-Fixer (`.php-cs-fixer.dist.php`): PSR-12, Symfony rules, single quotes, ordered imports, aligned binary operators. CI runs on PHP 8.3–8.5.

## Frontend

Package: `@pentiminax/ux-datatables` (ES module). Built TypeScript in `assets/src/`, compiled output in `assets/dist/`. Stimulus controller registered as `datatable`.

---
> Source: [pentiminax/ux-datatables](https://github.com/pentiminax/ux-datatables) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
