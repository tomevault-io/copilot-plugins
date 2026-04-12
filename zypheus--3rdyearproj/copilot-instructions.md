## 3rdyearproj

> Laravel project conventions and rules

# 2. Project Conventions and Rules

This document outlines the specific conventions, file structures, and general rules for this Laravel 12 project.

## Quick Reference

### Common Patterns

- Define models in `app/Models/` with Eloquent relationships.
- Define controllers in `app/Http/Controllers/` organized by feature.
- Define routes in `routes/web.php` with middleware groups.
- Define views in `resources/views/` organized by feature.
- Group feature code in `app/Features/{featureName}/` or similar directories.
- Use services in `app/Services/` for business logic reusability.
- Document features in `ai/docs/` with:
  - One markdown file per feature (e.g. `ai/docs/{featureName}.md`).
  - Database schema and model relationships.
  - User workflows and business logic.
  - Role-based access requirements (Admin, Officer, Member).
  - Update documentation when implementing feature changes.
- Reference the relevant `ai/docs/` files when writing or modifying feature code.

### Common Patterns & Import Rules

- **PHP Imports:** Use proper namespace imports at the top of files.
  - ✅ `use App\Models\Loan;`
  - ✅ `use App\Http\Controllers\LoanController;`
  - ✅ `use Illuminate\Support\Facades\Auth;`
  - ❌ `require_once 'path/to/file.php';` (Use namespaces instead)
- **Blade Template Imports:** Use Blade directives.
  - ✅ `@include('components.header')`
  - ✅ `@component('components.card')`
  - ❌ `<?php include '../components/header.php'; ?>`
- **Model Relationships:** Define relationships in model classes using Eloquent methods.
  - ✅ `public function loans() { return $this->hasMany(Loan::class); }`
  - ❌ Defining relationships in migrations or controllers
- **Middleware:** Apply middleware to routes or route groups.
  - ✅ `Route::middleware('auth')->group(function () { ... })`
  - ✅ `Route::middleware(['auth', 'role:admin'])->group(function () { ... })`
  - ❌ Role checking only in controllers without middleware
- **Form Requests:** Use Form Request classes for validation.
  - ✅ `use App\Http\Requests\StoreLoanRequest;`
  - ❌ Validation logic directly in controllers
- **Services:** Extract business logic into service classes.
  - ✅ `use App\Services\LoanService;`
  - ❌ Complex logic directly in controllers
- **Asset Compilation:** Use Vite for frontend assets.
  - ✅ Run `yarn dev` for development HMR
  - ✅ Compile with `yarn build` for production
  - ❌ Manual asset management without Vite

## Rules

### General Rules

- Always reference `routes/web.php` as your source of truth for the app's routes and structure.
- Always reference the `ai/docs/` directory for information on the app's features and functionality when writing code.
- Group feature code into feature directories (e.g. `app/Http/Controllers/Loans/`, `resources/views/loans/`).
- Use Laravel 12, as defined in `composer.json`.
- Combine related controllers, models, and views into feature-specific directories.
- Always use PHP 8+ syntax and type hints.
- Keep these rules files in `.cursor/rules/` updated with any new project conventions or changes in Laravel best practices.
- **Role System**: Only use three roles: Admin, Officer, and Member. Never add or reference other roles.
- **Asset Pipeline**: Use Vite for CSS and JavaScript compilation, configured in `vite.config.js`.

### Laravel Dependencies

- Manage dependencies via Composer using `composer.json`.
- Install dependencies via `composer install` or `composer require package-name`.
- Update dependencies via `composer update`.
- Keep `composer.lock` committed to version control.
- Manage frontend dependencies via Yarn/npm using `package.json`.
- Install frontend dependencies via `yarn install` or `npm install`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zypheus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zypheus)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
