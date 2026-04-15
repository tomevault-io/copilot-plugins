## repository-list

> This project is a Laravel 8 application built as part of a course on platzi.com. It is designed to demonstrate Test-Driven Development (TDD) principles within a Laravel environment. The application allows authenticated users to manage a list of repository entries.

# Project: Repository List

This project is a Laravel 8 application built as part of a course on platzi.com. It is designed to demonstrate Test-Driven Development (TDD) principles within a Laravel environment. The application allows authenticated users to manage a list of repository entries.

## Technology Stack
- **Framework:** Laravel 8 (PHP 7.3+)
- **Testing:** PHPUnit
- **Frontend:** Blade Templates, TailwindCSS, Alpine.js
- **Tooling:** Laravel Mix (Webpack)

## Project Architecture
The core of the application centers on managing user-owned repository entries, implemented via the `Repository` model and `RepositoryController`. Access to these repositories is governed by `RepositoryPolicy`, ensuring users can only interact with resources they own.

## Development & Testing
Development follows TDD, with comprehensive feature tests for controllers and unit tests for models.

### Key Commands
- **Install dependencies:** `composer install`
- **Run migrations:** `php artisan migrate`
- **Run seeders:** `php artisan db:seed`
- **Run development server:** `php artisan serve`
- **Run tests:** `php artisan test`
- **Build frontend assets:** `npm run dev` (or `npm run prod` for production)

## Development Conventions
- **Naming:** Follows PSR-4 autoloading standards.
- **Testing:** All new features or bug fixes must be accompanied by corresponding feature or unit tests, as demonstrated in `tests/Feature/Http/Controllers/RepositoryControllerTest.php`.
- **Authorization:** Always utilize policies for resource access control, as seen in `app/Policies/RepositoryPolicy.php`.
- **Validation:** Use `FormRequest` classes (e.g., `app/Http/Requests/RepositoryRequest.php`) for all input validation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/2Fac3R)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/2Fac3R)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
