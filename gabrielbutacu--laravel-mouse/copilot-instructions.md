## laravel-mouse

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Laravel 12 user management application (PHP 8.2+) used as a learning project for Laravel concepts. Uses SQLite database, database-backed queues, and log-based mail in development. UI labels are partially in Italian.

## Common Commands

```bash
# Serve the application (Laravel Herd handles this, but manually:)
php artisan serve

# Run migrations
php artisan migrate

# Run queue worker (processes jobs like TestJob, queued mail)
php artisan queue:work

# Run scheduler
php artisan schedule:run

# Run tests
php artisan test
# or
./vendor/bin/phpunit

# Run a single test file
php artisan test --filter=ExampleTest

# Custom command
php artisan app:send-invoices {number} {date?}
```

## Architecture

### Models & Relationships
- **User** → `belongsTo(City)`, `belongsToMany(Role)` with pivot attributes (`enabled`, timestamps)
- **City** → `hasMany(User)`
- **Role** → `belongsToMany(User)`

### Key Patterns
- **Async mail**: `Mail::queue(new WelcomeMail($user))` in UserController@save — relies on database queue driver
- **Profile images**: Stored in `public/images/`, served via `UserController@getProfileImage` with placeholder fallback
- **Role sync**: Uses `sync()` with pivot data (`enabled` flag) in UserController@saveUpdate
- **Search**: User index supports filtering by name and email via query string

### Queue & Jobs
- Queue connection: `database` (configured in `.env`)
- `TestJob`: Takes user ID, simulates long task (10s sleep)
- Mail is queued via `Mail::queue()`, not sent synchronously

### Console
- `SendInvoices` command scheduled every minute in `routes/console.php`

---
> Source: [gabrielbutacu/laravel-mouse](https://github.com/gabrielbutacu/laravel-mouse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
