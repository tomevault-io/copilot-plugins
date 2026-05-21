## tms

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LoadPartner TMS - An open source Transportation Management System for freight brokers built with Laravel + Inertia.js + React + TypeScript.

## Common Development Commands

### Environment Setup
```bash
# Set up with Laravel Sail
sail up -d
sail artisan migrate
sail npm install
sail artisan key:generate
sail npm run dev

# Fresh test data
sail artisan dev:refresh
```

### Development Workflow
```bash
# Run all dev services (server, queue, logs, vite)
composer run dev

# Individual commands
sail artisan serve
sail npm run dev
sail artisan queue:listen --tries=1
sail artisan pail --timeout=0
```

### Testing & Quality
```bash
# Run tests (uses Pest)
sail artisan test
# or
sail pest

# Run static analysis
sail artisan dev:check  # Runs PHPStan + IDE helper generation
vendor/bin/phpstan analyse

# Lint frontend
npm run lint
```

### Build & Production
```bash
# Build frontend assets
npm run build

# Link storage for file uploads
sail artisan storage:link
```

## Architecture Overview

### Laravel Actions Pattern
- All business logic organized in `app/Actions/` by domain
- Uses `lorisleiva/laravel-actions` package
- Each action has `handle()`, `asController()`, `rules()`, `authorize()` methods
- Actions serve as both controllers and reusable business logic

### Multi-Tenant Architecture
- All models use `HasOrganization` trait for automatic scoping
- `OrganizationScope` provides global filtering
- Use `current_organization_id()` helper for context

### State Machine Pattern
- Shipments use `spatie/laravel-model-states` 
- States: Pending → Booked → Dispatched → AtPickup → InTransit → AtDelivery → Delivered
- State transitions trigger events for side effects

### Frontend Structure
- React + TypeScript with Inertia.js for SPA-like experience
- Components organized by domain in `resources/js/Components/`
- UI components in `resources/js/Components/ui/`
- Pages in `resources/js/Pages/`

### Key Models & Relationships
- `Organization` - Multi-tenant container
- `Shipment` - Core business entity with state machine
- `Carrier` - Transportation providers
- `Customer` - Freight customers
- `Contact` - Polymorphic contacts system
- `Document` - Polymorphic file attachments
- `Note` - Polymorphic notes system

## Development Patterns

### Adding New Features
1. Create action in appropriate domain directory
2. Add enum values if needed (sync with `Permission::syncToDatabase()`)
3. Create policy for authorization
4. Add tests in corresponding test directories
5. Create frontend components following existing patterns

### Permissions System
- Add permissions to `app/Enums/Permission.php`
- Include entry in `label()` method
- Create migration calling `App\Enums\Permission::syncToDatabase()`

### Event-Driven Updates
- Use event bus for real-time updates between components
- Emit events: `emit('event-name-' + id)`
- Subscribe: `subscribe('event-name-' + id, callback)`

### Organization Defaults
- Modify `CreateOrUpdateOrganizationDefaults` action
- Consider both migration for existing orgs and action updates for new ones

## Database & Storage

### Database
- Uses SQLite for development, PostgreSQL for production
- Migrations in `database/migrations/`
- Seeders for test data in `database/seeders/`

### File Storage
- Supports both local and S3 storage
- Configured via `FILESYSTEM_DISK` environment variable
- Uses polymorphic `Document` model for file attachments

## Testing

### Test Structure
- Unit tests in `tests/Unit/`
- Feature tests in `tests/Feature/`
- Uses Pest testing framework
- Factory classes for model creation

### Test Users
Available via `sail artisan dev:refresh`:
- `admin@test.com` / `password` (admin)
- `user@test.com` / `password` (regular user)

## Key Dependencies

### Backend
- `lorisleiva/laravel-actions` - Business logic actions
- `spatie/laravel-model-states` - State machine
- `spatie/laravel-permission` - Authorization
- `inertiajs/inertia-laravel` - Frontend bridge
- `laravel/cashier` - Subscription billing

### Frontend
- `@inertiajs/react` - Inertia.js React adapter
- `@tanstack/react-table` - Data tables
- `react-hook-form` - Form management
- `@radix-ui/*` - UI components
- `tailwindcss` - Styling

## Code Style & Standards

### PHP
- Follow PSR-12 coding standards
- Use PHP 8.2+ features including enums
- Type hints required for all methods
- PHPStan level 5 static analysis

### TypeScript/React
- Strict TypeScript configuration
- Props interfaces for all components
- Use React hooks patterns
- ESLint + Prettier for code formatting

## Troubleshooting

### Common Issues
- Run `sail artisan storage:link` if file uploads fail
- Check organization context if data not appearing
- Verify permissions if authorization fails
- Use event bus for component updates, not direct prop passing

### Development Environment
- All services run via Docker Compose
- Access application at `http://localhost`
- Mailpit at `http://localhost:8025` for email testing
- Database accessible via standard Laravel tools

---
> Source: [loadpartner/tms](https://github.com/loadpartner/tms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
