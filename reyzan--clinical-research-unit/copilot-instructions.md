## clinical-research-unit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Laravel 12 + Filament 4 application for a Clinical Research Unit management system. The project uses:
- **Backend**: Laravel 12 (PHP 8.2+)
- **Admin Panel**: Filament 4 (admin panel framework)
- **Testing**: Pest 4 with Laravel plugin
- **Frontend**: Vite + Tailwind CSS 4
- **Database**: MySQL (configurable via .env)

## Development Commands

### Initial Setup
```bash
composer setup
# Runs: composer install, creates .env, generates key, migrates DB, npm install, npm run build
```

### Development Server
```bash
composer dev
# Runs 4 concurrent processes:
# - Laravel dev server (php artisan serve)
# - Queue worker (php artisan queue:listen --tries=1)
# - Log viewer (php artisan pail)
# - Vite dev server (npm run dev)
```

### Testing
```bash
composer test
# Clears config and runs Pest tests

# Run specific test file
php artisan test tests/Feature/ExampleTest.php

# Run specific test method
php artisan test --filter test_example
```

### Database
```bash
php artisan migrate              # Run migrations
php artisan migrate:fresh --seed # Fresh DB with seeders
php artisan db:seed              # Run seeders only
```

### Code Quality
```bash
vendor/bin/pint                  # Format code (Laravel Pint)
```

### Asset Building
```bash
npm run build                    # Production build
npm run dev                      # Development with hot reload
```

## Architecture

### Filament Admin Panel

The application uses Filament for the admin interface:

- **Panel Configuration**: `app/Providers/Filament/AdminPanelProvider.php`
  - Admin panel accessible at `/admin`
  - Uses Amber as primary color
  - Auto-discovers Resources, Pages, and Widgets from `app/Filament/` directories

- **Filament Components**:
  - **Resources**: Place in `app/Filament/Resources/` (auto-discovered)
  - **Pages**: Place in `app/Filament/Pages/` (auto-discovered)
  - **Widgets**: Place in `app/Filament/Widgets/` (auto-discovered)

- When creating Filament resources, use:
  ```bash
  php artisan make:filament-resource ResourceName --generate
  ```

### Directory Structure

- `app/Models/` - Eloquent models
- `app/Filament/Resources/` - Filament CRUD resources (auto-discovered)
- `app/Filament/Pages/` - Custom Filament pages (auto-discovered)
- `app/Filament/Widgets/` - Dashboard widgets (auto-discovered)
- `app/Http/Controllers/` - Standard Laravel controllers (for non-Filament routes)
- `app/Providers/Filament/` - Filament panel providers
- `database/migrations/` - Database migrations
- `database/seeders/` - Database seeders
- `database/factories/` - Model factories
- `routes/web.php` - Public web routes
- `routes/console.php` - Artisan commands
- `tests/Feature/` - Feature tests (Pest)
- `tests/Unit/` - Unit tests (Pest)

### Testing Configuration

- **Framework**: Pest 4
- **Config**: `tests/Pest.php`
- **PHPUnit XML**: `phpunit.xml`
- Tests in `Feature/` directory automatically extend `Tests\TestCase` with RefreshDatabase trait available (currently commented out)

### Queue & Jobs

- **Queue Connection**: Database (configurable in .env)
- Queue jobs are processed by `php artisan queue:listen` when running `composer dev`

### Frontend Assets

- **Vite Config**: `vite.config.js`
- **Entry Points**:
  - `resources/css/app.css`
  - `resources/js/app.js`
- Vite ignores `storage/framework/views/**` for performance

## Database Configuration

Default database is MySQL with these settings (from `.env.example`):
- **Database**: `clinical_research_unit`
- **Host**: 127.0.0.1
- **Port**: 3306

## Important Notes

- The project uses database-backed sessions and cache by default
- Queue connection is set to `database`
- Filament resources, pages, and widgets are auto-discovered from their respective directories
- The `composer dev` script runs all development processes concurrently

---
> Source: [Reyzan/clinical-research-unit](https://github.com/Reyzan/clinical-research-unit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
