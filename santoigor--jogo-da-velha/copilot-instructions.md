## jogo-da-velha

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a tic-tac-toe game ("jogo-da-velha" in Portuguese) built with Laravel 12 and Vue 3, using Inertia.js for seamless SPA-like functionality. The stack includes:

- **Backend**: Laravel 12, PHP 8.2+, Laravel Fortify for authentication
- **Frontend**: Vue 3 with TypeScript, Inertia.js, Tailwind CSS 4
- **Build Tools**: Vite with Laravel Wayfinder for type-safe routing
- **UI Components**: Reka UI (headless components) + custom shadcn-style components
- **Testing**: Pest PHP for backend tests

## Development Commands

### Running the Application

```bash
# Full development environment (server + queue + vite)
composer dev

# SSR development mode (server + queue + logs + SSR)
composer dev:ssr

# Individual services
php artisan serve           # Start server only
php artisan queue:listen    # Start queue worker
npm run dev                 # Start Vite dev server
```

### Building Assets

```bash
npm run build       # Client-side build
npm run build:ssr   # SSR build (includes client build)
```

### Testing

```bash
# Run all tests
composer test
# or
php artisan test

# Run specific test file
php artisan test tests/Feature/GameTest.php

# Run with Pest directly
./vendor/bin/pest
./vendor/bin/pest --filter=test_name
```

### Code Quality

```bash
npm run lint           # Auto-fix with ESLint
npm run format         # Format with Prettier
npm run format:check   # Check formatting without changes

# PHP formatting
./vendor/bin/pint      # Laravel Pint (auto-fix)
```

### Initial Setup

```bash
composer setup  # Runs install, env setup, key generation, migrations, and npm build
```

## Architecture

### Backend Structure

- **Models**: Basic Eloquent models. The `Game` model has relationships to `User` via `playerOne()` and `playerTwo()` methods (note: currently missing `BelongsTo` import)
- **Controllers**: Resource controllers in `app/Http/Controllers/`. Game logic is in `GameController`
- **Routes**:
  - `routes/web.php` - Main application routes, uses Inertia for rendering
  - `routes/settings.php` - User settings routes (profile, password, 2FA, appearance)
  - `routes/console.php` - Console commands
- **Authentication**: Laravel Fortify handles auth (login, register, password reset, 2FA)
- **Policies**: Authorization logic in `app/Policies/`

### Frontend Structure

- **Entry Point**: `resources/js/app.ts` initializes Inertia app with Vue 3
- **Pages**: `resources/js/pages/` - Inertia page components
  - `auth/` - Authentication pages (Login, Register, etc.)
  - `settings/` - User settings pages
  - `Welcome.vue`, `Dashboard.vue` - Main pages
- **Components**: `resources/js/components/` - Reusable Vue components
  - `ui/` - UI component library (shadcn-style with Reka UI)
  - App-level components (AppShell, AppHeader, AppSidebar, etc.)
- **Type-Safe Routing**: Laravel Wayfinder generates TypeScript route definitions
  - Generated routes are in `resources/js/routes/`
  - Route actions mirror Laravel controller structure in `resources/js/actions/`
  - Use these for type-safe navigation instead of hardcoded URLs
- **Composables**: `resources/js/composables/` - Vue composition functions
  - `useAppearance.ts` - Theme/appearance management
  - `useTwoFactorAuth.ts` - 2FA logic
  - `useInitials.ts` - User initials generation
- **Types**: `resources/js/types/` - TypeScript definitions

### Laravel Wayfinder Integration

Wayfinder automatically generates type-safe route helpers by analyzing Laravel routes. The generated files in `resources/js/routes/` and `resources/js/actions/` mirror the backend controller structure, providing autocomplete and type safety for navigation.

### Database

- Uses SQLite by default (check `.env` for `DB_CONNECTION`)
- Migrations in `database/migrations/`
- The `games` table tracks tic-tac-toe games with `player_one_id` and optional `player_two_id`

### Inertia.js Pattern

- Controllers return `Inertia::render('PageName', $data)` instead of Blade views
- Vue pages receive props from controllers
- Shared data is available to all pages via Inertia's shared data mechanism
- No traditional API endpoints needed for most page interactions

## Key Technical Details

### Vite Configuration

The project uses several Vite plugins:
- `laravel-vite-plugin` for Laravel integration
- `@tailwindcss/vite` for Tailwind CSS 4
- `@vitejs/plugin-vue` for Vue SFC support
- `@laravel/vite-plugin-wayfinder` for route generation (with `formVariants: true`)

### TypeScript Configuration

Strict TypeScript is enabled (`tsconfig.json`). The project uses `.vue` files with `<script setup lang="ts">`.

### Authentication Flow

Laravel Fortify provides auth endpoints. Frontend auth pages are in `resources/js/pages/auth/`. The application supports:
- Email/password authentication
- Two-factor authentication
- Email verification
- Password reset

### Testing Configuration

Pest is configured with Laravel plugin. Test database uses in-memory SQLite (`:memory:`). Tests are organized into Feature and Unit suites.

---
> Source: [santoigor/jogo-da-velha](https://github.com/santoigor/jogo-da-velha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
