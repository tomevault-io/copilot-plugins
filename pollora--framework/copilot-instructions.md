## framework

> Pollora Framework - Dependency Injection & Service Resolution Best Practices


# Dependency Injection & Service Resolution

## NEVER Use Laravel Facades
❌ **Avoid Laravel facades** - Always use the original classes:

```php
// ❌ DON'T
use Illuminate\Support\Facades\Cache;
Cache::get('key');

// ✅ DO
use Illuminate\Cache\Repository;
class MyService {
    public function __construct(private Repository $cache) {}
}
```

## Service Resolution Priority

### 1. Dependency Injection (PREFERRED)
✅ **Always prefer constructor injection or method injection when possible:**

```php
class MyServiceProvider extends ServiceProvider
{
    public function __construct(
        private Action $action,
        private Filter $filter
    ) {}

    public function boot(Action $action, Filter $filter): void
    {
        // Method injection in boot() is supported
        $action->add('init', $this->initMethod(...));
    }
}
```

### 2. Container Resolution
When dependency injection isn't possible, use container resolution:

```php
// ✅ PREFERRED: app()->get() (better performance)
$action = $this->app->get(Action::class);
$filter = $this->app->get(Filter::class);

// ✅ ALTERNATIVE: app()->make() (when get() isn't available)
$service = $this->app->make(SomeService::class);
```

## WordPress Hooks Integration

### Available Hook Services
Use Pollora's dedicated hook services instead of direct WordPress functions:

```php
use Pollora\Hook\Infrastructure\Services\Action;
use Pollora\Hook\Infrastructure\Services\Filter;
```

### Service Resolution for Hooks
```php
class MyServiceProvider extends ServiceProvider
{
    private Action $action;
    private Filter $filter;

    public function boot(): void
    {
        // Resolve hook services
        $this->action = $this->app->get(Action::class);
        $this->filter = $this->app->get(Filter::class);

        $this->registerHooks();
    }
}
```

### Hook Registration Examples
```php
private function registerHooks(): void
{
    // Actions
    $this->action->add('after_setup_theme', $this->registerMenus(...), 1);
    $this->action->add('init', $this->initMethod(...), 10);

    // Filters
    $this->filter->add('nav_menu_link_attributes', $this->handleLinkAttributes(...), 10, 4);
    $this->filter->add('nav_menu_item_attributes', $this->handleItemAttributes(...), 10, 4);
    $this->filter->add('nav_menu_submenu_attributes', $this->handleSubmenuAttributes(...), 10, 3);
    $this->filter->add('the_content', $this->modifyContent(...), 20, 1);
}
```

### Available Hook Methods
Both `Action` and `Filter` services provide:

```php
// Add hooks
$this->action->add('hook_name', $callback, $priority, $accepted_args);
$this->filter->add('hook_name', $callback, $priority, $accepted_args);

// Remove hooks
$this->action->remove('hook_name', $callback, $priority);
$this->filter->remove('hook_name', $callback, $priority);

// Execute hooks
$this->action->do('hook_name', ...$args);
$result = $this->filter->apply('hook_name', $value, ...$args);
```

## Best Practices Summary

1. **Never use Laravel facades** - always use original classes
2. **Prioritize dependency injection** in constructors and methods
3. **Use `app()->get()`** for better performance when DI isn't possible
4. **Use `app()->make()`** only when `get()` isn't available
5. **Use Pollora hook services** instead of direct `add_action`/`add_filter`
6. **Resolve hook services once** and reuse the instances
7. **Register hooks in service provider `boot()` method**

## Performance Notes
- `app()->get()` is faster than `app()->make()` for singleton services
- Dependency injection is the most performant option
- Cache service instances when possible to avoid repeated resolution

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pollora) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
