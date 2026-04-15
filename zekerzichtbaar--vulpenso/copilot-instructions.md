## vulpenso

> Anything related to Laravel/PHP


@base.mdc

# Laravel Integration in Sage

## Key Laravel Features Used
- Blade templating
- Service providers
- View composers
- Laravel helpers

## Blade Template Features
- `@extends()` - Template inheritance
- `@section()` and `@endsection` - Content sections
- `@include()` - Include partials
- `@if`, `@foreach`, etc. - Control structures
- `@class()` - Conditional classes
- `{!! !!}` - Unescaped output (for WP content with HTML)
- `{{ }}` - Escaped output

## Components
- Standard Blade components with `<x-component-name />`
- Component parameters: `<x-button :link="$url" color="primary" />`
- Common components:
  - `<x-buttons :buttons="$buttons" />` - Button collection
  - `<x-button :link="$link" :color="$color" />` - Individual button
  - Card components in `cards/` directory

## Conditional Class Example
```blade
<div @class([
    'base-class',
    'conditional-class' => $condition,
    $dynamicClass => $dynamicClass,
])>
    {{ $content }}
</div>
```

## Loops in Blade
```blade
@if($items)
    @foreach($items as $item)
        <div class="item">
            {{ $item->title }}
        </div>
    @endforeach
@endif
```

## Service Providers
- Register services in `app/Providers/`
- Register view composers
- Register custom directives

# Laravel Conventions in Sage
- Use Laravel's naming conventions for classes and files
- Follow PSR-4 autoloading standards
- Use namespaces for all PHP classes
- Use type hints and return types for method signatures
- Use dependency injection instead of global functions

# Service Providers
- Register services in `app/Providers/`
- Extend `Roots\Acorn\ServiceProvider`
- Use `register()` method for binding services to the container
- Use `boot()` method for code that needs to run after all services are registered

```php
namespace App\Providers;

use Roots\Acorn\ServiceProvider;

class ExampleServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->bind('example', function () {
            return new \App\Services\Example();
        });
    }

    public function boot()
    {
        // Code that runs after all services are registered
    }
}
```

# Blade Templates
- Use `@extends` and `@section` for template inheritance
- Use `@include` for partial templates
- Use `@component` or `<x-...>` for components
- Escape output with `{{ }}` instead of `{!! !!}` when possible
- Use `@php @endphp` sparingly - prefer moving PHP logic to Controllers/Composers
- Use compact, readable syntax for conditional rendering

```blade
@if($condition)
    <div>Conditional content</div>
@endif

@foreach($items as $item)
    <div>{{ $item }}</div>
@endforeach

@forelse($items as $item)
    <div>{{ $item }}</div>
@empty
    <div>No items found</div>
@endforelse
```

# View Composers
- Use View Composers to share data with specific views
- Keep the Blade templates focused on presentation
- Create reusable components for consistent UI elements

# Helper Functions
- Create helper functions in `app/helpers.php`
- Use Laravel Collections for data manipulation
- Follow Laravel's functional style for helper methods

```php
// Example helper function
function example_helper($data)
{
    return collect($data)
        ->filter(fn($item) => !empty($item))
        ->map(fn($item) => transform_item($item))
        ->all();
}
```

# Routing and Controllers
- Use WordPress template system for primary routing
- Use WordPress conditionals in controllers/composers
- Create specialized controllers for AJAX/API endpoints

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zekerzichtbaar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zekerzichtbaar)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
