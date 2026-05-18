## mason

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mason is a block-based drag-and-drop page/document builder field for Filament (Laravel admin panel framework). It allows users to compose content using configurable "bricks" that can be added, edited, reordered, and removed.

## Development Commands

```bash
# Run all tests (refactor check, lint check, unit tests)
composer test

# Run only unit tests (with parallel execution)
composer test:unit

# Run a single test file
./vendor/bin/pest tests/src/FieldTest.php

# Run a specific test
./vendor/bin/pest --filter "test name"

# Fix code style
composer lint

# Check code style without fixing
composer test:lint

# Run Rector refactoring
composer refactor

# Check Rector without applying changes
composer test:refactor
```

## Architecture

### Core Components

- **Mason** (`src/Mason.php`): The main Filament form field component. Extends `Field` and handles state hydration/dehydration, brick commands (insert, update, delete, move), and preview layout configuration.

- **MasonEntry** (`src/MasonEntry.php`): Infolist entry component for displaying Mason content in read-only views.

- **Brick** (`src/Brick.php`): Abstract base class for all bricks. Each brick must implement:
  - `getId()`: Unique identifier
  - `getLabel()`: Display name (auto-generated from ID by default)
  - `getIcon()`: Optional icon
  - `toHtml()`: Renders the brick's HTML output
  - `configureBrickAction()`: Configures the Filament action for editing the brick

### Support Classes

- **MasonRenderer** (`src/Support/MasonRenderer.php`): Converts stored JSON content to HTML, text, or array formats
- **Faker** (`src/Support/Faker.php`): Helper for generating fake Mason content in tests
- **IframeRenderer** (`src/Support/IframeRenderer.php`): Handles rendering content within the editor iframe

### Artisan Commands

- `php artisan make:mason-brick {Name}`: Scaffolds a new brick class and view templates

## Key Patterns

### Creating Bricks

Bricks are Filament Actions with an associated view. The generator creates:
1. A PHP class in `App\Mason` namespace
2. Blade templates in `resources/views/mason/`

### Data Storage

Mason stores content as JSON arrays. Each brick entry has:
- `type`: Always "masonBrick"
- `attrs.id`: The brick's ID
- `attrs.config`: The brick's form data
- `attrs.preview` and `attrs.label`: Stripped before saving (editor-only)

### Rendering Content

```php
// Helper function
mason($content, $bricks)->toHtml()

// Full renderer API
MasonRenderer::make($content)->bricks($bricks)->toHtml()
```

## Code Style

- Uses Laravel Pint with strict rules (see `pint.json`)
- Requires `declare(strict_types=1)` in all PHP files
- PHPStan level 5 for static analysis
- Follow PSR-2 with class element ordering defined in pint.json

---
> Source: [awcodes/mason](https://github.com/awcodes/mason) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
