## midash

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Midash is a personal dashboard app (glance-style) built with Phoenix 1.7 + LiveView 1.0, Tailwind CSS, and PostgreSQL.

## Common Commands

```bash
mix setup                  # Install deps, create DB, build assets
mix phx.server             # Start dev server on localhost:4000
iex -S mix phx.server      # Start with interactive shell
mix test                   # Run all tests (auto-creates/migrates DB)
mix test test/path_test.exs       # Run single test file
mix test test/path_test.exs:42    # Run single test at line
mix ecto.migrate           # Run pending migrations
mix ecto.reset             # Drop + recreate + seed DB
```

## Architecture

### Dashboard Layout Model

Every page is a LiveView that renders a **glance-style column layout**: columns sit side-by-side (flexbox), widgets stack vertically within each column.

**Flow:** Router → LiveView (sets `:dashboard` layout) → `dashboard_layout` component → `col` components → `widget` cards → `live_component` widgets

### Key Components (`lib/midash_web/components/dashboard_components.ex`)

- `dashboard_layout` — full-page wrapper with nav + flexbox column container
- `dashboard_nav` — top nav bar driven by a `@nav_pages` list of `%{id, label, path}`
- `col` — vertical column, either `:small` (w-64 fixed) or `:full` (flex-1)
- `widget` — card wrapper with optional title bar

### Widget Pattern

Widgets live in `lib/midash_web/widgets/` as `live_component` modules. Self-updating widgets use this pattern:

1. Widget calls `Process.send_after(self(), {:some_tick, assigns.id}, interval)` in `update/2`
2. The **parent LiveView** handles the tick message in `handle_info/2` and calls `send_update(WidgetModule, id: id)`
3. `handle_info` is NOT a valid callback on `live_component` — don't use `@impl true` with it there

### Adding a New Page

1. Create a LiveView in `lib/midash_web/live/`
2. Add a route in `lib/midash_web/router.ex` under the browser scope
3. Add nav entry to the `@nav_pages` module attribute (currently duplicated per LiveView)

### Visual Style

Dark monochrome theme: `#0d0d0d` background, `#141414` widget cards, monospace font throughout, minimal gray borders. Use Tailwind utility classes — no custom CSS.

---
> Source: [bluzky/midash](https://github.com/bluzky/midash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
