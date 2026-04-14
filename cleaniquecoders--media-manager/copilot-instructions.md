## media-manager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Laravel package called "Media Manager" (`cleaniquecoders/media-manager`) that provides media management capabilities built on top of Spatie Media Library. The package is designed to manage media through a user interface.

## Development Commands

```bash
# Run tests
composer test

# Run tests with coverage
composer test-coverage

# Run a single test file
./vendor/bin/pest tests/ExampleTest.php

# Run a specific test by name
./vendor/bin/pest --filter "test name"

# Static analysis
composer analyse

# Code formatting
composer format
```

## Architecture

This is a Laravel package built using `spatie/laravel-package-tools`. The package structure follows standard Laravel package conventions:

- **Service Provider**: `src/MediaManagerServiceProvider.php` - Configures the package (config, views, migrations, commands)
- **Facade**: `src/Facades/MediaManager.php` - Provides static access to the MediaManager class
- **Core Class**: `src/MediaManager.php` - Main package functionality (currently a stub)
- **Artisan Command**: `src/Commands/MediaManagerCommand.php` - CLI command accessible via `php artisan media-manager`
- **Config**: `config/media-manager.php` - Package configuration
- **Views**: `resources/views/` - Blade templates

## Testing

Tests use Pest PHP with Orchestra Testbench for Laravel package testing. The test configuration:

- Base test case: `tests/TestCase.php` extends Orchestra's TestCase
- Pest configuration: `tests/Pest.php` binds the base TestCase
- Architecture tests: `tests/ArchTest.php` ensures no debugging functions are used

## Static Analysis

PHPStan is configured at level 5 with:
- Octane compatibility checks enabled
- Model property checks enabled
- Analyzes `src/`, `config/`, and `database/` directories

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cleaniquecoders)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cleaniquecoders)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
