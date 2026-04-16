## finances-tui

> Personal finance TUI in Rust. Production database on Neon (PostgreSQL), local dev via docker-compose.

# Finances — Developer Guide

Personal finance TUI in Rust. Production database on Neon (PostgreSQL), local dev via docker-compose.

## Build & Run

```sh
make help                     # show all targets
make dev                      # start local PostgreSQL
make migrate                  # run migrations (dev)
make migrate-prod             # run migrations (prod)
make run                      # TUI (local dev DB)
make run-prod                 # TUI (production Neon DB)
make test                     # run all tests (needs local DB running)
make test-one T=test_name     # run a single test
make release                  # optimized release build
make clippy                   # lint check
make deploy                   # full pipeline: test → migrate → release
cargo run -- --notify --prod  # send desktop notifications (for cron)
```

## Deployment

The release binary (`target/release/finances`) is launched on laptop startup
via Hyprland: `exec-once = sh -c 'cd ~/personal/finances && ./target/release/finances --notify --prod'`

`make deploy` runs tests first — if any test fails, the pipeline aborts before
migrating or building. After a successful deploy the release binary is updated
in-place and will be used on the next startup.

## Environment

- `.env` — local dev database URL (docker-compose)
- `.env.prod` — production database URL (Neon, TLS required)
- `.env.example` / `.env.prod.example` — templates tracked in git
- `--prod` flag switches between `.env` and `.env.prod`

## Architecture

- **Pattern**: Elm/TEA — Event → Update (`handle_key`) → Render (`draw`)
- **App struct**: single source of truth, caches DB data, refreshes after mutations
- **Screens**: 10 screens (Dashboard, Transactions, Accounts, Categories, Budgets, Installments, Recurring, Transfers, CC Payments, CC Statements)
- **InputMode**: `Normal` (navigation) vs `Editing` (form input)
- **ConfirmAction enum**: tracks what a popup confirmation will do
- **Forms**: per-screen form structs with validation; early-return for Esc/Enter before `if let Some(form)` to avoid borrow conflicts

## Key Conventions

- **Money**: always `rust_decimal::Decimal`, never `f64`
- **Currency**: BRL (R$), formatted via `format_brl()` with dot thousands / comma decimal
- **Enums**: all implement `Display` + `FromStr`; DB functions accept enum types, not `&str`
- **Balances**: computed from transactions + transfers + credit_card_payments, never stored
- **Migrations**: `sqlx::migrate!()` — idempotent, tracked in `_sqlx_migrations` table
- **Logging**: tracing with daily-rotating file appender in `~/.local/share/finances/`
- **Exports**: CSV files written to `~/.local/share/finances/exports/`

## i18n (Internationalization)

- **Languages**: English (default) and Brazilian Portuguese
- **Runtime toggle**: `Ctrl+L` in the TUI switches `App.locale` between `Locale::En` and `Locale::Pt`
- **CLI flag**: `--lang pt` sets the locale for `--notify` mode only (desktop notifications via cron)
- **Module**: `src/ui/i18n.rs` — `t(locale, key)` for static strings, `tf_*()` format helpers for dynamic messages
- **Enum labels**: `Locale::enum_label(en_label)` translates `.label()` output at the UI layer without changing `Display`/`FromStr` impls (which double as DB values)
- **Category names**: `categories.name_pt` DB column; `App::category_name_localized()` picks PT→EN→"?" fallback
- **Help popup**: `?` key opens per-screen bilingual help overlay (`src/ui/components/help_popup.rs`)
- **Adding a new string**: add the key to both `t_en()` and `t_pt()` in `i18n.rs`; for dynamic messages, add a `tf_*()` helper

## Database

- 9 tables: accounts, categories, transactions, transfers, credit_card_payments, installment_purchases, budgets, recurring_transactions, notifications
- Local: `finances/finances@localhost:5432/finances` (docker-compose)
- Production: Neon free tier with TLS (`tls-rustls` feature in sqlx)
- Migration files: `migrations/20260305_initial.sql`, `migrations/20260320_add_indexes.sql`, `migrations/20260326_add_category_name_pt.sql`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mickaelyoshua) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
