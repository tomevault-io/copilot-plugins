## core-template

> > For Gemini Code Assist, Jules, and other Google AI tools.

# Core PHP Framework - AI Assistant Context

> For Gemini Code Assist, Jules, and other Google AI tools.

## Project Type

Laravel 12 application using Core PHP Framework - a modular monolith with event-driven architecture.

## Directory Structure

```
app/
├── Mod/              # Feature modules (your code)
│   └── {Name}/
│       ├── Boot.php  # Event listeners
│       ├── Models/
│       ├── Routes/
│       ├── Views/
│       └── Livewire/
├── Core/             # Local framework overrides (EUPL-1.2)
└── Providers/

config/core.php       # Framework configuration
```

## Module Pattern

Every module has a `Boot.php` with static `$listens` array:

```php
<?php

declare(strict_types=1);

namespace App\Mod\Shop;

use Core\Events\WebRoutesRegistering;
use Core\Events\ApiRoutesRegistering;
use Core\Events\AdminPanelBooting;

class Boot
{
    public static array $listens = [
        WebRoutesRegistering::class => 'onWebRoutes',
        ApiRoutesRegistering::class => 'onApiRoutes',
        AdminPanelBooting::class => 'onAdminPanel',
    ];

    public function onWebRoutes(WebRoutesRegistering $event): void
    {
        $event->routes(fn() => require __DIR__.'/Routes/web.php');
        $event->views('shop', __DIR__.'/Views');
    }

    public function onApiRoutes(ApiRoutesRegistering $event): void
    {
        $event->routes(fn() => require __DIR__.'/Routes/api.php');
    }

    public function onAdminPanel(AdminPanelBooting $event): void
    {
        $event->navigation('Shop', 'shop.admin.index', 'shopping-cart');
    }
}
```

## Code Style Requirements

### Language: UK English
- colour (not color)
- organisation (not organization)
- centre (not center)
- behaviour (not behavior)
- licence (noun), license (verb)

### PHP Standards
- `declare(strict_types=1);` in all files
- Full type hints (parameters + return types)
- PSR-12 formatting (use Laravel Pint)
- Pest for testing

### Naming Conventions
| Type | Convention | Example |
|------|------------|---------|
| Model | Singular PascalCase | `Product` |
| Table | Plural snake_case | `products` |
| Controller | `{Model}Controller` | `ProductController` |
| Livewire Page | `{Feature}Page` | `ProductListPage` |
| Livewire Modal | `{Feature}Modal` | `EditProductModal` |

## UI Stack

- **Livewire 3** - Server-side reactivity
- **Flux Pro** - UI component library (NOT vanilla Alpine)
- **Tailwind CSS** - Utility-first styling
- **Font Awesome Pro** - Icons (NOT Heroicons)

## Common Commands

```bash
php artisan make:mod Blog --all    # Create module with all features
php artisan serve                   # Development server
vendor/bin/pint --dirty            # Format changed files
vendor/bin/pest                    # Run tests
```

## Packages

| Package | Namespace | Purpose |
|---------|-----------|---------|
| host-uk/core | `Core\` | Framework core |
| host-uk/core-admin | `Core\Admin\` | Admin panel |
| host-uk/core-api | `Core\Api\` | REST API |
| host-uk/core-mcp | `Core\Mcp\` | AI agent tools |

## Avoid

- American English spellings
- Heroicons (use Font Awesome)
- Vanilla Alpine components (use Flux)
- Over-engineering / premature abstraction
- PHPUnit syntax (use Pest)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/host-uk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/host-uk)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
