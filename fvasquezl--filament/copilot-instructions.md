## filament

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Laravel 12.x application with Filament 3.3 admin panel. The project uses SQLite as the database and follows standard Laravel conventions with Filament resource-based architecture for the admin interface.

## Essential Development Commands

### Starting Development
```bash
# Recommended: Run all services concurrently (server, queue, logs, vite)
composer dev

# Or run services individually:
php artisan serve    # PHP development server
npm run dev         # Vite development server
```

### Testing
```bash
# Run all tests
composer test

# Run specific test suites
php artisan test --testsuite=Unit
php artisan test --testsuite=Feature

# Run a specific test file
php artisan test tests/Feature/ExampleTest.php
```

### Code Quality
```bash
# Format code using Laravel Pint
./vendor/bin/pint

# Check specific files
./vendor/bin/pint app/Models/User.php
```

### Database Operations
```bash
# Run migrations
php artisan migrate

# Rollback migrations
php artisan migrate:rollback

# Fresh migration (drop all tables and re-run)
php artisan migrate:fresh

# Run seeders
php artisan db:seed
```

### Building for Production
```bash
# Build frontend assets
npm run build
```

## Architecture Overview

### Filament Admin Panel
- **URL**: `/app`
- **Provider**: `app/Providers/Filament/AppPanelProvider.php`
- **Resources**: Located in `app/Filament/Resources/`
- **Auto-discovery**: Resources, pages, and widgets are auto-discovered

### Resource Structure
Filament resources follow this pattern:
```
app/Filament/Resources/
├── ModelResource.php           # Resource definition
└── ModelResource/
    └── Pages/
        ├── ListModels.php      # List page
        ├── CreateModel.php     # Create page
        └── EditModel.php       # Edit page
```

### Key Architectural Patterns
1. **Single Page Resources**: Some resources (like UserResource) use `ManageUsers` pattern for single-page CRUD with modals
2. **Form/Table Builders**: Resources define forms and tables using Filament's fluent builders
3. **Actions**: Standard actions (Edit, Delete) with bulk operations support

### Database
- **Driver**: SQLite (file: `database/database.sqlite`)
- **Test Database**: In-memory SQLite
- **Migrations**: Standard Laravel migrations in `database/migrations/`

### Frontend Build
- **Tool**: Vite 6.x
- **CSS**: Tailwind CSS 4.x
- **Entry Points**: 
  - `resources/css/app.css`
  - `resources/js/app.js`

## Common Development Tasks

### Creating a New Filament Resource
```bash
# Generate a resource with all pages
php artisan make:filament-resource ModelName

# Generate a simple resource (single page with modals)
php artisan make:filament-resource ModelName --simple

# Generate with custom operations
php artisan make:filament-resource ModelName --generate
```

### Working with Filament Components
- Forms: Use the `Form` builder in the `form()` method
- Tables: Use the `Table` builder in the `table()` method
- Both support extensive customization through chained methods

### Environment Setup for New Developers
1. Clone repository
2. Copy environment file: `cp .env.example .env`
3. Install dependencies: `composer install && npm install`
4. Generate key: `php artisan key:generate`
5. Create database: `touch database/database.sqlite`
6. Run migrations: `php artisan migrate`
7. Start development: `composer dev`

## Important Notes
- The UserResource uses a simplified single-page pattern with `ManageUsers`
- Filament handles authentication and authorization
- All Filament routes are prefixed with `/app`
- The project uses Filament's default theme with blue as primary color
- Custom font "Popins" is configured (likely should be "Poppins")

---
> Source: [fvasquezl/filament](https://github.com/fvasquezl/filament) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
