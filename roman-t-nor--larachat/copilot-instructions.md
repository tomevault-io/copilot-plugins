## larachat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Frontend Development
```bash
npm run dev          # Start Vite dev server
npm run build        # Production build
npm run build:ssr    # Build with SSR support
npm run format       # Format code with Prettier
npm run lint         # Fix linting issues with ESLint
npm run types        # TypeScript type checking
```

### Backend Development
```bash
composer dev         # Runs concurrent server, queue, logs, and vite
composer dev:ssr     # Development with SSR
php artisan serve    # Laravel dev server only
php artisan queue:listen  # Queue worker
php artisan pail     # Log viewer
php artisan inertia:start-ssr  # SSR server
```

### Testing
```bash
composer test        # Run all tests (Pest PHP)
php artisan test     # Alternative test command
php artisan test --filter TestName  # Run specific test
php artisan test tests/Feature/Auth  # Run tests in specific directory
```

### Code Quality
```bash
./vendor/bin/pint    # Format PHP code (Laravel Pint)
npm run lint         # Fix JS/TS linting issues
npm run format       # Format JS/TS with Prettier
npm run format:check # Check formatting without fixing
```

## Architecture Overview

### Tech Stack
- **Backend**: Laravel 11 with PHP 8.2+
- **Frontend**: React 19 with TypeScript
- **State Management**: Inertia.js for server-driven UI
- **Styling**: Tailwind CSS v4 with shadcn/ui components
- **Database**: SQLite (default), configurable via .env
- **Build Tool**: Vite with SSR support
- **AI Integration**: OpenAI PHP client for chat functionality
- **Real-time**: Laravel Stream package for SSE streaming

### Key Architectural Patterns

1. **Inertia.js Integration**
   - Server-side routing drives the application
   - Pages are React components in `resources/js/pages/`
   - Props are passed from Laravel controllers to React components
   - No separate API needed - controllers return Inertia responses

2. **Component Structure**
   - UI components: `resources/js/components/ui/` (shadcn/ui based)
   - Feature components: `resources/js/components/` 
   - Layouts: `resources/js/layouts/` with nested layout support
   - Custom hooks: `resources/js/hooks/`

3. **Authentication Flow**
   - Laravel Breeze scaffolding with full auth implementation
   - Auth pages in `resources/js/pages/auth/`
   - Protected routes use Laravel middleware
   - Session-based authentication

4. **Frontend State Management**
   - Server state managed by Inertia.js
   - Local state with React hooks
   - Appearance preferences stored in localStorage and cookies
   - Mobile detection via custom hook

5. **Routing Structure**
   - Main routes: `routes/web.php`
   - Auth routes: `routes/auth.php` 
   - Settings routes: `routes/settings.php`
   - All routes return Inertia responses

6. **Middleware**
   - `HandleInertiaRequests`: Shares global data to all pages
   - `HandleAppearance`: Manages theme/color preferences

### Important Implementation Details

- TypeScript is used throughout the frontend - ensure all new components are properly typed
- The project uses path aliases configured in tsconfig.json (e.g., `@/` maps to `resources/js/`)
- shadcn/ui components are customized and stored locally in `resources/js/components/ui/`
- SSR is configured but optional - use `npm run dev:ssr` for SSR development
- OpenAI integration requires setting OPENAI_API_KEY in .env
- Chat feature with streaming responses using useStream hook
- Chat persistence for authenticated users with sidebar display
- Tests use an in-memory SQLite database - no setup required

### Development Notes
- Always use the updated Inertia 2.0 useForm implementation

---
> Source: [roman-t-nor/LaraChat](https://github.com/roman-t-nor/LaraChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
