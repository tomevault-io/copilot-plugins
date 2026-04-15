## efinance

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

<laravel-boost-guidelines>
=== foundation rules ===

## Foundational Context

**Package Versions:**
- php 8.3.6, laravel/framework v12, react v19, tailwindcss v4
- inertiajs/inertia-laravel v2, @inertiajs/react v2
- pestphp/pest v4, laravel/pint v1, laravel/wayfinder v0
- eslint v9, prettier v3

**Core Philosophy:** Write simple, well-tested, secure code. Code for humans first, computers second.

**KISS & DRY:**
- Choose the simplest solution - avoid premature abstraction and clever code
- Extract common code only after third repetition (Rule of Three)
- Abstract when pattern is clear, stable, and appears 3+ times

**Conventions:**
- Follow existing code conventions - check sibling files for structure, approach, naming
- Use descriptive names: `isRegisteredForDiscounts`, not `discount()`
- Check for existing components before writing new ones
- Stick to existing directory structure - don't create new base folders without approval
- Only create documentation files if explicitly requested


=== development rules ===

## Development Commands

**Frontend:** `npm run dev` (Vite HMR) | `npm run build` | `npm run lint` | `npm run format` | `npm run types`

**Backend:** `composer dev` (full stack) | `composer test` | `php artisan test --filter=TestName` | `php artisan pail` (logs) | `php artisan optimize:clear`


=== architecture rules ===

## Service-Repository Architecture

**Flow:** Controller (HTTP) → Service (Business Logic) → Repository (Data Access) → Model → Database

- **Controllers:** Accept requests, validate input, call services, return responses - nothing else
- **Services:** Business logic, coordinate repositories, handle transactions and events
- **Repositories:** Abstract database operations behind interfaces, injected via DI
- **Models:** Relationships and accessors only - no business logic or queries

**Directory Structure:**
```
app/Http/Controllers/     # Thin controllers, HTTP only
app/Services/             # Business logic
app/Repositories/         # Data access (with interfaces)
app/Models/               # Eloquent models
resources/js/pages/       # Inertia page components
resources/js/components/  # Reusable components (ui/ for Ant Design wrappers)
resources/js/layouts/     # Page layouts
resources/js/actions/     # Wayfinder generated routes
resources/js/hooks/       # Custom React hooks
resources/js/types/       # TypeScript definitions
tests/Feature/            # Feature tests (preferred)
tests/Unit/               # Unit tests (minimal)
```


=== php rules ===

## PHP 8.3

- Start every file with `declare(strict_types=1);`
- Use typed class constants, constructor property promotion, explicit return types
- Add `#[\Override]` when overriding parent methods
- Use readonly classes for DTOs and value objects
- Validate JSON with `json_validate()` before decoding
- Always use curly braces for control structures
- Prefer PHPDoc blocks over inline comments
- Enum keys should be TitleCase: `FavoritePerson`, `Monthly`

✗ Never use `mixed` types unless absolutely necessary
✗ Never allow empty `__construct()` methods with zero parameters


=== typescript rules ===

## TypeScript

- Enable strict mode and all strict flags in tsconfig
- Use `unknown` for uncertain types, narrow with type guards
- Define explicit interfaces for all component props
- Use discriminated unions for state with status field
- Type error objects in catch blocks explicitly

✗ Never use `any` - always use `unknown`
✗ Never leave parameters or return types implicit
✗ Never use @ts-ignore


=== performance rules ===

## Performance

**Laravel:**
- Always eager load with `->with()` - N+1 prevention is critical
- Add `Model::preventLazyLoading(!app()->isProduction())` in AppServiceProvider
- Select only needed columns, index foreign keys and queried columns
- Chunk large datasets with `->chunk(100)`
- Cache expensive queries, queue long-running operations

**React 19:**
- Let React Compiler handle optimization - no manual useMemo/useCallback
- Code split with lazy() and Suspense, virtualize lists over 100 items
- Add lazy loading and dimensions to images


=== security rules ===

## Security (OWASP Top 10)

**Input Validation:** Always use Form Requests, validate on both client and server, use whitelist validation
✗ Never use `$request->all()` directly

**XSS:** Use Blade `{{ }}` and React JSX for escaping, sanitize with Purify before `{!! !!}` or dangerouslySetInnerHTML

**SQL Injection:** Use Eloquent/Query Builder exclusively, parameter bindings for raw queries
✗ Never concatenate user input into SQL

**Auth:** Protect routes with middleware, use policies/gates, call `$this->authorize()` before sensitive operations

**File Uploads:** Validate MIME type/extension, limit sizes, store outside public directory, use UUID filenames

**Environment:** Use `config()` in code, `env()` only in config files
✗ Never commit .env or hardcode credentials


=== react rules ===

## React 19

- Use `useActionState` for forms (handles state, loading, errors)
- Use `useOptimistic` for instant UI updates with auto-rollback
- Use `useFormStatus` for submit button pending states
- Pass `ref` as regular prop - forwardRef is obsolete
- Use `use()` hook for Promises and Context

**State Management:**
- Inertia shared data for global state (user, flash, config)
- Custom hooks for component-specific state
- Context for theme/appearance
- React 19 Actions for all form submissions


=== antd rules ===

## Ant Design & Theming

- Wrap app in ConfigProvider with theme configuration
- Use `theme.darkAlgorithm` or `theme.defaultAlgorithm` based on preference
- Get colors from `theme.useToken()` - never hardcode
- Use tokens: colorPrimary, colorBgContainer, colorText, colorBorder
- Combine Tailwind utilities with Ant Design components

✗ Never hardcode color values
✗ Never use Ant Design without theme tokens


=== ux rules ===

## UI/UX Design Standards

See `.ai/rules/ux.md` for comprehensive guidelines on:
- Visual design (shadow-free aesthetic), typography, color/contrast (WCAG 2.2)
- Accessibility, human-centered content, form design
- Loading states, empty states, navigation, responsive/mobile


=== inertia-laravel/core rules ===

## Inertia Core

- Components in `resources/js/Pages` directory
- Use `Inertia::render()` instead of Blade views
- Always use Wayfinder's `route()` from @/actions - never hardcode URLs
- Use `<Head title="..." />` for page titles
- Configure shared data in HandleInertiaRequests middleware
- Access shared data via `usePage<PageProps>().props`
- Use `search-docs` for Inertia guidance


=== inertia-laravel/v2 rules ===

## Inertia v2 Features

- Polling, Prefetching, Deferred props, Infinite scrolling with `WhenVisible`
- Use `<Form>` component for forms (recommended) or `useForm` helper
- Add skeleton loading states for deferred props


=== laravel/core rules ===

## Laravel Core

- Use `php artisan make:` commands with `--no-interaction`
- Prefer `Model::query()` over `DB::`, eager load to prevent N+1
- Create Form Request classes for validation (not inline)
- Create factories and seeders when creating models
- Use queued jobs for long operations (`ShouldQueue`)
- Use policies/gates for authorization, named routes for URLs
- Use `config()` not `env()` in application code


=== laravel/v12 rules ===

## Laravel 12 Structure

- No middleware files in `app/Http/Middleware/`
- Register middleware/exceptions/routing in `bootstrap/app.php`
- Service providers in `bootstrap/providers.php`
- Commands auto-register from `app/Console/Commands/`
- Use `casts()` method on models (not `$casts` property)


=== pint/core rules ===

## Laravel Pint

Run `vendor/bin/pint --dirty` before finalizing changes.


=== pest/core rules ===

## Pest Testing

- Use AAA pattern: Arrange, Act, Assert
- Use `describe()` blocks to organize related tests
- Write descriptive test names explaining what is tested
- Use `php artisan make:test --pest <name>`
- Use factories for test data, datasets for validation rules
- Run minimal tests with `--filter=testName`
- Use specific assertions: `assertForbidden`, `assertNotFound` (not `assertStatus`)

✗ Don't aim for 100% coverage - test what matters
✗ Don't test framework features or trivial getters/setters


=== pest/v4 rules ===

## Pest 4 Features

- Browser testing in `tests/Browser/`, smoke testing, visual regression
- Use `visit()`, interact with page, `assertNoJavascriptErrors()`
- Test multiple browsers/devices/viewports if requested


=== inertia-react/core rules ===

## Inertia + React

Use `router.visit()` or `<Link>` for navigation.


=== inertia-react/v2/forms rules ===

## Inertia Forms

```jsx
import { Form } from '@inertiajs/react'

<Form action="/users" method="post">
  {({ errors, processing, wasSuccessful }) => (
    <>
      <input type="text" name="name" />
      {errors.name && <div>{errors.name}</div>}
      <button disabled={processing}>{processing ? 'Creating...' : 'Create User'}</button>
    </>
  )}
</Form>
```


=== tailwindcss rules ===

## Tailwind CSS

See `.ai/rules/tailwindcss.md` for Tailwind v4 rules and guidelines.


=== anti-patterns rules ===

## Anti-Patterns to AVOID

**Over-Engineering:** Unnecessary interfaces before patterns emerge, abstracting before 3+ repetitions, "flexible" solutions for hypothetical requirements

**Performance:** Loading relationships in loops, missing indexes, selecting all columns

**Architecture:** Repositories without interfaces, business logic in controllers

**Type Safety:** Using `any` in TypeScript, disabling strict mode

**Code Quality:** Duplicating diverged code, unclear variable names, complex nested ternaries, empty catch blocks


=== checklist rules ===

## Development Checklist

**Code Quality:**
- [ ] PHP files start with `declare(strict_types=1);`
- [ ] TypeScript strict mode enabled
- [ ] Service-Repository pattern followed
- [ ] Controllers under 20 lines per method
- [ ] KISS/DRY applied

**Security:** Form Requests for validation, OWASP practices, no hardcoded credentials

**Performance:** Relationships eager loaded, indexes added, React Compiler leveraged

**Testing:** Feature tests for critical flows, edge cases tested

**UI/UX:** See `.ai/rules/ux.md`

**Routes:** Wayfinder routes used, TypeScript interfaces for page props, React 19 Actions for forms


=== boost rules ===

## Laravel Boost MCP Tools

- `list-artisan-commands` - Check available Artisan parameters
- `get-absolute-url` - Get correct URL for sharing with user
- `tinker` - Execute PHP to debug or query models
- `database-query` - Read-only database queries
- `browser-logs` - Read browser errors/exceptions
- `search-docs` - **Use first** for Laravel ecosystem docs (version-specific)
  - Pass multiple queries: `['rate limiting', 'routing']`
  - Don't add package names to queries


=== tests rules ===

## Test Enforcement

Every change must be tested. Run minimal tests with `php artisan test --filter=testName`.
</laravel-boost-guidelines>
- Use playwright for UI/UX and functional feature and module testing.
- Local URL: http://efinance.test
- Use info@fayazk.com as email and `@Password1` as password

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FayazK)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/FayazK)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
