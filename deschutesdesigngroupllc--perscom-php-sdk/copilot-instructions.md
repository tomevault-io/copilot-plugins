## perscom-php-sdk

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the PERSCOM PHP SDK - a comprehensive PHP SDK for interacting with the PERSCOM personnel management platform API. Built using Saloon HTTP client library, it provides a fluent interface for all PERSCOM API endpoints.

## Development Commands

### Testing
- `composer test` - Run full test coverage with minimum 100% coverage requirement
- `composer herd:test` - Run tests with Herd (alternative test runner)
- `composer herd:test:filter` - Run filtered tests with Herd
- `composer test:types` - Run type coverage analysis with minimum 100% requirement

### Code Quality
- `composer lint` - Format code using Laravel Pint (runs on src/ directory)
- `composer test:lint` - Test code formatting without making changes
- `./vendor/bin/phpstan` - Run static analysis (configuration in phpstan.dist.neon)

### Dependencies
- PHP 8.2+ required
- Uses Saloon HTTP client as the core HTTP layer
- Laravel/Illuminate Support package for utilities
- Rate limiting via saloonphp/rate-limit-plugin

## Architecture

### Core Components

**PerscomConnection** (`src/PerscomConnection.php`):
- Main connector class extending Saloon\Http\Connector
- Manages API authentication via Bearer token
- Provides factory methods for all resource types
- Handles custom exception mapping for PERSCOM API errors
- Base URL: `https://api.perscom.io/v2`

**Resource Pattern**:
- All resources extend `Resource` base class (`src/Http/Resources/Resource.php`)
- Resources provide methods for CRUD operations on specific PERSCOM entities
- Examples: `UserResource`, `AwardResource`, `RankResource`, etc.

**Request Pattern**:
- Abstract request classes in `src/Http/Requests/`
- Common patterns: `AbstractGetAllRequest`, `AbstractCreateRequest`, `AbstractUpdateRequest`, etc.
- Each entity has specific request classes (e.g., `CreateUserRequest`, `UpdateUserRequest`)

### Key Interfaces

**Batchable** (`src/Contracts/Batchable.php`):
- Enables batch operations (create, update, delete) on resources
- Accepts single `ResourceObject` or arrays of objects

**Searchable** (`src/Contracts/Searchable.php`):
- Provides advanced search capabilities with filters, sorting, and scoping
- Uses `FilterObject`, `SortObject`, and `ScopeObject` for complex queries

**ResourceContract** (`src/Contracts/ResourceContract.php`):
- Standard CRUD interface that most resources implement

### Data Objects

Located in `src/Data/`:
- `ResourceObject` - Base data transfer object
- `FilterObject` - For search filtering
- `SortObject` - For result sorting
- `ScopeObject` - For search scoping

### Exception Handling

Custom exceptions in `src/Exceptions/` mapped to HTTP status codes:
- 401: `AuthenticationException`
- 402: `PaymentRequiredException`
- 403: `ForbiddenException`
- 404: `NotFoundHttpException`
- 429: `RateLimitException`
- 500: `ServerErrorException`
- 503: `ServiceUnavailableException`

## Testing Structure

- Uses Pest PHP testing framework
- Tests organized by feature in `tests/Feature/`
- Mirrors source structure (HTTP/Requests/, HTTP/Resources/)
- Architecture tests via Pest Plugin Arch
- 100% code coverage requirement enforced
- 100% type coverage requirement enforced

## Code Style

- PSR-4 autoloading with `Perscom\` namespace
- Strict types enabled (`declare(strict_types=1)`)
- Laravel Pint for code formatting (configuration in `pint.json`)
- PHPStan for static analysis

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deschutesdesigngroupllc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/deschutesdesigngroupllc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
