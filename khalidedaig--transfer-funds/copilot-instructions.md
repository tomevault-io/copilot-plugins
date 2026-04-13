## transfer-funds

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Laravel 12 application demonstrating concurrent fund transfers between bank accounts. The application showcases Laravel's Concurrency facade for handling parallel operations and database pessimistic locking for transaction safety.

## Development Commands

### Setup
```bash
composer install
npm install
cp .env.example .env
php artisan key:generate
php artisan migrate --seed
```

### Development Server
```bash
# Start all services (server, queue, logs, and vite) concurrently
composer dev

# Or start individually:
php artisan serve              # Start Laravel development server
php artisan queue:listen       # Start queue worker
php artisan pail              # View logs
npm run dev                   # Start Vite dev server
```

### Building for Production
```bash
npm run build                 # Build frontend assets with Vite
```

### Testing
```bash
php artisan test              # Run all tests using Pest
php artisan test --filter=ExampleTest  # Run specific test
```

### Code Quality
```bash
./vendor/bin/pint             # Run Laravel Pint for code formatting
```

## Architecture & Key Patterns

### Concurrency & Database Locking

The core functionality demonstrates handling concurrent fund transfers safely:

- **BankService** (`app/Services/BankService.php`): Implements fund transfer logic using database transactions with pessimistic locking (`lockForUpdate()`)
- **BankController** (`app/Http/Controllers/BankController.php`): Uses Laravel's `Concurrency::run()` to execute multiple transfers in parallel
- **Critical Pattern**: All fund transfers use `lockForUpdate()` to prevent race conditions when transferring funds to multiple beneficiaries simultaneously

### Data Model

- **User Model** (`app/Models/User.php`): Extended with a `balance` field (decimal) that tracks account balance
- **Database Seeder** (`database/seeders/DatabaseSeeder.php`): Creates 3 test users (John with 500 balance, Alicia, and Phillipe)

### Frontend Architecture

- Uses Blade templates (`resources/views/bank.blade.php`) with inline Tailwind CSS
- Assets managed via Vite (`vite.config.js`)
- Form at `/transfer-funds` allows selecting multiple beneficiaries and transfer amount

## Important Implementation Details

### Database Transactions
When working with fund transfers or any balance modifications:
- Always wrap operations in `DB::transaction()`
- Use `lockForUpdate()` when reading records that will be modified
- Validate sufficient balance before deducting

### Configuration
- Database: MySQL (default), configured in `.env`
- Queue driver: Database-backed (`database/migrations/0001_01_01_000002_create_jobs_table.php`)
- Session driver: Database-backed
- Cache store: Database-backed

### Testing Framework
- Uses Pest PHP for testing (`tests/Pest.php`)
- Test structure: Feature tests in `tests/Feature/`, Unit tests in `tests/Unit/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/khalidedaig)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/khalidedaig)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
