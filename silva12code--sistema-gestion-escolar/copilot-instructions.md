## sistema-gestion-escolar

> This project is a comprehensive school management system built with the Laravel 12 framework. It appears to be designed for a Peruvian audience, as evidenced by the integration with RENIEC (the Peruvian national identity registry). The application follows a modular architecture, with distinct modules for managing students, parents, teachers, enrollments, courses, schedules, and more. It uses Laravel Sanctum for API authentication and `spatie/laravel-permission` for fine-grained role-based access control.

# Project Overview

This project is a comprehensive school management system built with the Laravel 12 framework. It appears to be designed for a Peruvian audience, as evidenced by the integration with RENIEC (the Peruvian national identity registry). The application follows a modular architecture, with distinct modules for managing students, parents, teachers, enrollments, courses, schedules, and more. It uses Laravel Sanctum for API authentication and `spatie/laravel-permission` for fine-grained role-based access control.

## Key Technologies

*   **Backend:** PHP 8.2, Laravel 12
*   **Authentication:** Laravel Sanctum
*   **Authorization:** spatie/laravel-permission
*   **Database:** (Not explicitly defined, but likely MySQL, PostgreSQL, or SQLite, as is common with Laravel)
*   **Frontend:** (Not explicitly defined, but the presence of `vite.config.js` and `package.json` suggests a JavaScript frontend, possibly Vue.js or React)
*   **Other:**
    *   `azure-oss/storage-blob-laravel`: For interacting with Azure Blob Storage.

# Building and Running

## Development Environment

To run the development environment, use the following command:

```bash
composer run dev
```

This command concurrently starts the following services:

*   `php artisan serve`: The Laravel development server.
*   `php artisan queue:listen`: The Laravel queue worker.
*   `php artisan pail`: A command-line tool for tailing logs.
*   `npm run dev`: The Vite development server for the frontend.

## Testing

To run the test suite, use the following command:

```bash
composer run test
```

This command clears the configuration cache and then runs the tests using `php artisan test`.

# Development Conventions

*   **Modular Architecture:** The application is organized into modules, each with its own controllers, models, requests, and services. This pattern should be followed when adding new features.
*   **API Versioning:** The API is versioned with a `v1` prefix. All new API routes should be added to this version.
*   **Authentication and Authorization:** All protected routes must use the `auth:sanctum` middleware. Role-based access control should be implemented using the `spatie/laravel-permission` package.
*   **Code Style:** While not explicitly defined, it is recommended to follow the PSR-12 coding style standard, which is the default for Laravel.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/silva12Code)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/silva12Code)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
