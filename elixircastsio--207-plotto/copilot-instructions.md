## 207-plotto

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Setup and Dependencies
- `mix setup` - Complete project setup (installs deps, creates/migrates database, installs Node deps)
- `mix deps.get` - Install Elixir dependencies
- `npm install --prefix assets` - Install Node.js dependencies for assets

### Development
- `mix phx.server` or `iex -S mix phx.server` - Start Phoenix server (interactive shell with iex)
- `mix ecto.create` - Create database
- `mix ecto.migrate` - Run database migrations
- `mix ecto.reset` - Drop, create, migrate, and seed database

### Testing
- `mix test` - Run all tests
- `mix test path/to/test_file.exs` - Run specific test file
- `mix test path/to/test_file.exs:LINE` - Run specific test at line number

### Code Quality
- `mix format` - Format code according to .formatter.exs rules
- `mix compile --warnings-as-errors` - Compile with warnings as errors

### Database
- `mix ecto.gen.migration migration_name` - Generate new migration
- `mix run priv/repo/seeds.exs` - Run seed data

## Architecture Overview

This is a Phoenix 1.8 application that demonstrates modern Phoenix patterns with LiveView as the primary UI approach.

### Core Architecture Pattern
The application follows Phoenix's Context-driven design:
- **Contexts** (`lib/teacher/`): Business logic organized by domain (e.g., `Recordings` context)
- **Schemas** (`lib/teacher/recordings/`): Ecto schemas define data structures
- **Web Layer** (`lib/teacher_web/`): HTTP interface using LiveView for interactive UIs

### Key Architectural Components

1. **LiveView-First Approach**: Main UI built with Phoenix LiveView for reactive interfaces
   - LiveView modules in `lib/teacher_web/live/`
   - Minimal traditional controllers (mainly for static pages)

2. **Component System**: Uses Phoenix.Component for reusable UI components
   - Core components in `lib/teacher_web/components/core_components.ex`
   - Layout components for consistent UI structure

3. **Database Layer**: PostgreSQL with Ecto
   - Migrations in `priv/repo/migrations/`
   - Repository pattern via `Teacher.Repo`

4. **Asset Pipeline**: Modern JavaScript/CSS tooling
   - ESBuild for JavaScript bundling
   - Tailwind CSS for styling
   - Configured in `config/dev.exs` and `assets/`

### Testing Strategy
- **Unit Tests**: Context functions tested in isolation (`test/teacher/`)
- **Integration Tests**: LiveView tests verify full user interactions (`test/teacher_web/live/`)
- **Test Helpers**: ConnCase, DataCase, and fixtures in `test/support/`

### Current Domain Model
The application is a small record store: a music catalog sold on vinyl, CD and
digital download.

- **Recordings Context** (`lib/teacher/recordings.ex`): CRUD for the catalog itself
  - **Album**: artist, title, summary, year
- **Sales Context** (`lib/teacher/sales.ex`): the orders and revenue behind it
  - **Sale**: album_id, format, price_cents, buyer_name, buyer_email, sold_on, refunded_on
  - Reports monthly units by format, gross revenue, refunds, cumulative net
    revenue, and the best selling albums

Every "by month" function in `Sales` returns one entry per month in the window,
oldest first, zero-filling months with no activity — so the series are always
complete and evenly spaced. Revenue is counted in the month an album sold; a
refund is counted in the month it was issued, which may be later.

Routes: the album catalog is at `/`, an album at `/albums/:id`, the sales
dashboard at `/dashboard`.

Buying or refunding an album from its page broadcasts `{:sale_created, sale}` or
`{:sale_refunded, sale}` over `Teacher.PubSub`. The dashboard subscribes via
`Sales.subscribe/0` and reloads its numbers, briefly highlighting the tiles the
change affects — green when money comes in, red when it goes back out.

This codebase serves as a good example of modern Phoenix development patterns and can be extended with additional features while maintaining the established architectural patterns.

---
> Source: [elixircastsio/207-plotto](https://github.com/elixircastsio/207-plotto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
