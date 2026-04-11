## workflowy-sdk

> This project is a modern, type-safe PHP SDK for the Workflowy API, designed for PHP 8.2+ and Laravel 12+.

# Workflowy PHP SDK - Project Context

This project is a modern, type-safe PHP SDK for the Workflowy API, designed for PHP 8.2+ and Laravel 12+.

## Project Overview

- **Purpose**: Provide a clean, object-oriented interface for interacting with Workflowy.
- **Key Technologies**:
    - **Language**: PHP 8.2+ (utilizing `readonly` classes, Enums, and strict typing).
    - **HTTP Stack**: PSR-18 (HTTP Client), PSR-17 (HTTP Factories), and PSR-7 (HTTP Messages).
    - **Framework Integration**: Built-in support for Laravel (Service Provider and Facade).
- **Architecture**:
    - **Client**: `Workflowy\WorkflowyClient` is the main entry point, requiring an API key and PSR-compliant HTTP factories/client.
    - **Resources**: API endpoints are organized into resource classes (e.g., `src/Resources/Nodes.php`, `src/Resources/Targets.php`).
    - **DTOs**: Data is returned as structured Data Transfer Objects (e.g., `src/DTO/NodeData.php`) rather than raw arrays.
    - **Exceptions**: Custom exceptions are located in `src/Exceptions/`.

## Building and Running

### Installation
```bash
composer install
```

### Running Tests
The project uses [Pest](https://pestphp.com/).
```bash
vendor/bin/pest
```
*Note: Currently, a `tests/` directory is not present in the repository.*

### Code Styling
The project uses [Laravel Pint](https://laravel.com/docs/11.x/pint) for code style enforcement.
```bash
vendor/bin/pint
```

## Development Conventions

- **PSR-4 Autoloading**: All source code resides in the `src/` directory under the `Workflowy\` namespace.
- **Type Safety**: Use strict typing (`declare(strict_types=1);`) in all PHP files.
- **DTO Usage**: Always return DTOs from resource methods to ensure type safety and IDE completion.
- **Laravel Integration**:
    - Configuration is stored in `config/workflowy.php`.
    - The Service Provider (`src/Laravel/WorkflowyServiceProvider.php`) handles dependency injection.
    - The Facade (`src/Laravel/Facades/Workflowy.php`) provides a static proxy to the client.

## Key Files

- `src/WorkflowyClient.php`: The core client logic and request handling.
- `src/Resources/Nodes.php`: Methods for managing Workflowy nodes (get, create, update, delete).
- `src/DTO/NodeData.php`: The primary data structure for node information.
- `src/Laravel/WorkflowyServiceProvider.php`: Laravel integration logic.
- `config/workflowy.php`: Default configuration for Laravel.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomcoufal)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tomcoufal)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
