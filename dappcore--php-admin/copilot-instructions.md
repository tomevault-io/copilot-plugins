## php-admin

> This is a Laravel application using the Core PHP Framework - a modular monolith architecture with event-driven module registration.

# GitHub Copilot Instructions

## Project Overview

This is a Laravel application using the Core PHP Framework - a modular monolith architecture with event-driven module registration.

## Architecture

### Module System
- Modules live in `app/Mod/{ModuleName}/`
- Each module has a `Boot.php` class with event listeners
- Events: `WebRoutesRegistering`, `ApiRoutesRegistering`, `AdminPanelBooting`

### Example Boot.php
```php
<?php

namespace App\Mod\Blog;

use Core\Events\WebRoutesRegistering;
use Core\Events\ApiRoutesRegistering;

class Boot
{
    public static array $listens = [
        WebRoutesRegistering::class => 'onWebRoutes',
        ApiRoutesRegistering::class => 'onApiRoutes',
    ];

    public function onWebRoutes(WebRoutesRegistering $event): void
    {
        $event->routes(fn() => require __DIR__.'/Routes/web.php');
        $event->views('blog', __DIR__.'/Views');
    }
}
```

## Coding Standards

### Language
- Use UK English (colour, organisation, centre, behaviour)
- No American spellings (color, organization, center, behavior)

### PHP Style
- PSR-12 with Laravel conventions
- Strict types: `declare(strict_types=1);`
- Type hints on all parameters and return types
- Final classes by default unless inheritance is intended

### Naming
- Models: singular PascalCase (`Post`, `Comment`)
- Tables: plural snake_case (`posts`, `comments`)
- Controllers: `{Model}Controller`
- Livewire: `{Feature}Page`, `{Feature}Modal`

### Testing
- Use Pest, not PHPUnit directly
- Feature tests for HTTP/Livewire
- Unit tests for services/utilities

## UI Framework

- **Livewire 3** for reactive components
- **Flux Pro** for UI components (not vanilla Alpine)
- **Tailwind CSS** for styling
- **Font Awesome Pro** for icons (not Heroicons)

## Key Packages

| Package | Purpose |
|---------|---------|
| `host-uk/core` | Core framework, events, modules |
| `host-uk/core-admin` | Admin panel, modals |
| `host-uk/core-api` | REST API, rate limiting |
| `host-uk/core-mcp` | AI agent tools (MCP) |

## Don't

- Don't use Heroicons (use Font Awesome Pro)
- Don't use vanilla Alpine components (use Flux Pro)
- Don't create controllers for Livewire pages
- Don't use American English spellings
- Don't add unnecessary abstractions

---
> Source: [dAppCore/php-admin](https://github.com/dAppCore/php-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
