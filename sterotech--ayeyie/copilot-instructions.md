## ayeyie

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ayeyie Poultry Feed Software is a Laravel 12 application built for Ayeyie Poultry Feed Depot. It's an integrated payment and pickup system with fraud detection, QR code receipts, and inventory management. The system uses the TALL stack (Tailwind, Alpine.js, Laravel, Livewire) with Livewire Volt and Flux UI components.

## Common Development Commands

### Development Environment
```bash
# Start development environment (includes server, queue, logs, and Vite)
composer run dev
# OR manually start individual services:
php artisan serve
php artisan queue:listen --tries=1
php artisan pail --timeout=0
npm run dev
```

### Code Quality & Testing
```bash
# Run tests
php artisan test
./vendor/bin/pest

# Code formatting with Laravel Pint
./vendor/bin/pint

# Static analysis with PHPStan/Larastan
./vendor/bin/phpstan analyse

# Generate IDE helpers
php artisan ide-helper:generate
php artisan ide-helper:models
```

### Database Management
```bash
# Run migrations
php artisan migrate

# Fresh migration with seeding
php artisan migrate:fresh --seed

# Create migration
php artisan make:migration create_table_name

# Create model with factory and migration
php artisan make:model ModelName -mf
```

### Build and Deployment
```bash
# Build assets for production
npm run build

# Clear various caches
php artisan optimize:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
```

## Architecture and Code Patterns

### Domain Organization
The application follows a domain-driven structure:
- **Admin Module**: Product, user, audit log, stock alert, and suspicious activity management
- **Customer Module**: Order creation, editing, and pickup management  
- **Core Models**: Product, User, Transaction, Receipt, Pickup, etc. with comprehensive relationships

### Livewire Component Structure
- **Full-page Components**: Located in `app/Livewire/` with corresponding Blade views in `resources/views/livewire/`
- **Volt Components**: Single-file components using Livewire Volt syntax
- **Action Classes**: Located in `app/Livewire/Actions/` for complex business logic (e.g., `CreateUser`, `UpdateUser`)
- **Form Classes**: Located in `app/Livewire/Forms/` for form handling and validation

### Model Patterns
- All models use `declare(strict_types=1)` and are marked `final`
- Custom primary keys (e.g., `product_id` instead of `id`) 
- Comprehensive PHPDoc with `@property` annotations for IDE support
- Proper relationship definitions with generic type hints
- Factory support for all models

### Database Design
- 3rd Normal Form (3NF) schema design
- Custom primary key naming convention (`product_id`, `transaction_id`, etc.)
- Comprehensive foreign key relationships
- Migration files include detailed comments explaining field purposes
- Consistent use of `created_at` and `updated_at` timestamps

### Testing Strategy
- Uses Pest testing framework with PHPUnit as the base
- Feature tests for Livewire components in `tests/Feature/Livewire/`
- Unit tests for models in `tests/Unit/Models/`
- Authentication tests in `tests/Feature/Auth/`
- Test configuration includes `RefreshDatabase` trait and Storage faking

### Code Quality Standards
- Laravel Pint with custom configuration in `pint.json`
- PHPStan at maximum level with Larastan for Laravel-specific analysis
- Strict typing enforced throughout the codebase
- Final classes and comprehensive type hints
- Ordered class elements and imports

## Key Framework Integrations

### Livewire & Volt
- Primary frontend framework using Livewire 3.x with Volt integration
- Flux UI components from Livewire for consistent design system
- URL-aware components with `#[Url]` attributes for search and filtering

### Authentication & Authorization
- Laravel Breeze-style authentication
- Policy-based authorization (though policies are currently restrictive placeholders)
- Role-based access control in User model

### Queue & Notifications
- Laravel's built-in queue system for background processing
- Notification system configured for customer SMS/email updates

## Important Development Notes

### Database Conventions
- Always use custom primary keys matching table names (e.g., `product_id` for products table)
- Check existing migration patterns before creating new tables
- Use proper foreign key relationships with descriptive constraint names

### Livewire Best Practices
- Use action classes for complex business logic instead of embedding in components
- Implement proper URL state management for filters and search
- Follow the established pattern of query builders with filtering, sorting, and pagination

### Code Style Requirements
- All new PHP files must include `declare(strict_types=1)`
- Use final classes unless inheritance is specifically needed
- Include comprehensive PHPDoc annotations
- Follow existing naming conventions for routes, views, and components

### Testing Requirements
- Write feature tests for all Livewire components
- Include model tests for relationships and business logic
- Use factories for test data generation
- Test both happy path and edge cases

## Standard Admin Page Layout Structure

All admin pages should follow the standardized layout pattern established by the product index page for consistency and maintainability:

### 1. Page Header Section
```blade
<!-- Page Header -->
<div class="flex flex-col items-start justify-between gap-4 md:flex-row md:items-center">
    <div>
        <h1 class="text-3xl font-bold text-text-primary">Page Title</h1>
        <p class="text-text-secondary">Page description</p>
    </div>
    <div class="flex items-center space-x-4">
        <!-- Breadcrumb Navigation -->
        <nav class="flex items-center space-x-2 text-sm">
            <a href="{{ route('dashboard') }}" class="text-text-secondary hover:text-text-primary transition-colors">Dashboard</a>
            <flux:icon.chevron-right class="w-4 h-4 text-text-secondary" />
            <span class="text-text-primary font-medium">Current Page</span>
        </nav>
        <!-- Action Buttons -->
        <flux:button variant="primary" icon="plus">Primary Action</flux:button>
    </div>
</div>
```

### 2. Statistics Cards Section (Optional)
```blade
<!-- Statistics Cards -->
<div class="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-4">
    <div class="bg-card rounded-xl shadow-sm p-6 border border-border">
        <div class="flex items-center">
            <div class="flex-shrink-0">
                <div class="w-8 h-8 bg-primary/10 rounded-lg flex items-center justify-center">
                    <flux:icon.squares-2x2 class="w-5 h-5 text-primary" />
                </div>
            </div>
            <div class="ml-4">
                <dt class="text-sm font-medium text-text-secondary">Metric Label</dt>
                <dd class="text-2xl font-bold text-text-primary">{{ number_format($value) }}</dd>
            </div>
        </div>
    </div>
</div>
```

### 3. Filters and Search Section
```blade
<!-- Filters and Search -->
<div class="bg-card rounded-xl shadow-sm border border-border">
    <div class="p-6">
        <div class="flex items-center justify-between mb-4">
            <h3 class="text-lg font-semibold text-text-primary">Filters & Search</h3>
            @if($hasActiveFilters)
                <flux:button variant="ghost" wire:click="resetFilters" size="sm">
                    <flux:icon.x-mark class="w-4 h-4 mr-1" />
                    Clear Filters
                </flux:button>
            @endif
        </div>
        
        <div class="grid grid-cols-1 gap-4 md:grid-cols-4">
            <!-- Search Field -->
            <div class="md:col-span-2">
                <flux:field>
                    <flux:label>Search</flux:label>
                    <flux:input wire:model.live.debounce.300ms="search" placeholder="Search..." icon="magnifying-glass" />
                </flux:field>
            </div>
            <!-- Additional Filters -->
        </div>
    </div>
</div>
```

### 4. Main Content Section (Table/Cards)
```blade
<!-- Main Content -->
<div class="bg-card rounded-xl shadow-sm border border-border overflow-hidden">
    <div class="overflow-x-auto">
        <table class="min-w-full divide-y divide-border">
            <thead class="bg-muted">
                <tr>
                    <th wire:click="sortBy('field')" class="px-6 py-3 text-left text-xs font-medium text-text-secondary uppercase tracking-wider cursor-pointer hover:bg-muted-hover transition-colors">
                        <div class="flex items-center space-x-1">
                            <span>Column Name</span>
                            <!-- Sort indicators -->
                        </div>
                    </th>
                </tr>
            </thead>
            <tbody class="bg-card divide-y divide-border">
                <!-- Table rows -->
            </tbody>
        </table>
    </div>
</div>
```

### 5. Pagination Section
```blade
<!-- Results Info and Pagination -->
<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between">
    <div class="flex items-center space-x-2 text-sm text-text-secondary">
        <span>Showing {{ $items->firstItem() ?? 0 }} to {{ $items->lastItem() ?? 0 }} of {{ $items->total() }} items</span>
    </div>
    <div class="mt-3 sm:mt-0">
        {{ $items->links() }}
    </div>
</div>
```

### Design System Requirements
- **Colors**: Use design tokens (text-primary, text-secondary, bg-card, border-border, etc.)
- **Spacing**: Consistent `space-y-6` for main sections
- **Cards**: All content cards use `bg-card rounded-xl shadow-sm border border-border`
- **Icons**: Use Flux icons with consistent sizing (w-4 h-4 for small, w-5 h-5 for medium)
- **Buttons**: Use Flux button variants (primary, outline, ghost) with appropriate icons
- **Typography**: Follow hierarchy (text-3xl for h1, text-lg for h3, text-sm for descriptions)

### Interactive Elements Standards
- **Hover States**: All interactive elements should have `hover:` state changes
- **Transitions**: Use `transition-colors` for smooth color changes
- **Loading States**: Consider loading indicators for async operations
- **Empty States**: Provide meaningful empty state messages with call-to-action buttons

This standardized structure ensures consistency across all admin pages and provides a solid foundation for maintainable, accessible, and visually coherent interfaces.

## Reusable Admin UI Components

To reduce code duplication and ensure consistency, use these reusable components located in `resources/views/components/ui/`:

### 1. Admin Page Layout Component
Use `<x-ui.admin-page-layout>` for all admin pages:

```blade
<x-ui.admin-page-layout
    title="Page Title"
    description="Page description"
    :breadcrumbs="[['label' => 'Current Page']]"
    :stats="$statsArray"
    :show-filters="true"
    search-placeholder="Search..."
    :has-active-filters="$hasFilters"
>
    <x-slot:actions>
        <flux:button variant="primary" icon="plus">Add Item</flux:button>
    </x-slot:actions>
    
    <x-slot:filterSlot>
        <!-- Custom filters here -->
    </x-slot:filterSlot>
    
    <!-- Main content here -->
</x-ui.admin-page-layout>
```

**Props:**
- `title` (required): Page title
- `description`: Page description
- `breadcrumbs`: Array of breadcrumb items `[['label' => 'Name', 'url' => 'optional-url']]`
- `actions`: Slot for action buttons in header
- `stats`: Array of statistics `[['label' => 'Label', 'value' => '100', 'icon' => 'icon-name', 'iconBg' => 'bg-class', 'iconColor' => 'text-class']]`
- `showFilters`: Boolean to show/hide filter section
- `filterSlot`: Slot for custom filters
- `searchPlaceholder`: Search input placeholder
- `searchModel`: Livewire model for search (default: 'search')
- `hasActiveFilters`: Boolean to show clear filters button
- `resetFiltersMethod`: Method name to clear filters (default: 'resetFilters')

### 2. Admin Table Component
Use `<x-ui.admin-table>` for data tables:

```blade
<x-ui.admin-table 
    :headers="[
        ['label' => 'Name', 'field' => 'name', 'sortable' => true],
        ['label' => 'Status', 'field' => 'status', 'sortable' => false]
    ]"
    :items="$paginatedItems"
    empty-title="No Items Found"
    empty-description="Custom empty message"
    :has-active-filters="$hasFilters"
    :sort-by="$sortBy"
    :sort-direction="$sortDirection"
>
    @foreach($paginatedItems as $item)
        <tr class="hover:bg-muted transition-colors">
            <td class="px-6 py-4">{{ $item->name }}</td>
            <td class="px-6 py-4">{{ $item->status }}</td>
        </tr>
    @endforeach                    

    <x-slot:emptyAction>
        <flux:button variant="primary">Add First Item</flux:button>
    </x-slot:emptyAction>
</x-ui.admin-table>
```

**Props:**
- `headers`: Array of table headers with labels, fields, and sortable flags
- `items`: Paginated collection of items
- `emptyTitle`: Title for empty state
- `emptyDescription`: Description for empty state
- `emptyAction`: Slot for action button in empty state
- `hasActiveFilters`: Boolean for empty state message variation
- `resetFiltersMethod`: Method to clear filters (default: 'resetFilters')
- `sortBy`: Current sort field for header indicators
- `sortDirection`: Current sort direction ('asc' or 'desc')

**Important Notes:**
- The component handles the `@forelse` loop and `<tr>` wrapper automatically
- Your slot content should only contain `<td>` elements
- The `$item` variable is available in the slot for each row
- Row hover effects are built-in

### 3. Admin Pagination Component
Use `<x-ui.admin-pagination>` for consistent pagination:

```blade
<x-ui.admin-pagination 
    :items="$paginatedItems" 
    item-name="products"
    :has-active-filters="$hasFilters"
/>
```

**Props:**
- `items`: Paginated collection
- `itemName`: Singular/plural item name for display (default: 'items')
- `hasActiveFilters`: Boolean to show filtered results indicator

### Component Usage Benefits

1. **Consistency**: All admin pages follow the same visual and interaction patterns
2. **Maintainability**: Changes to design system can be made in one place
3. **Productivity**: Faster development with pre-built, tested components
4. **Code Reduction**: Significantly less template code (60 lines vs 330+ lines)
5. **Accessibility**: Consistent accessibility patterns across all pages
6. **Responsiveness**: Built-in responsive design patterns

### Migration Strategy

When working on admin pages:
1. **New Pages**: Always use the UI components from the start
2. **Existing Pages**: Gradually refactor to use components when making updates
3. **Testing**: Ensure component props work correctly with your data structure
4. **Customization**: Use slots for page-specific content that doesn't fit the standard pattern

See `resources/views/components/ui/admin-page-layout-example.blade.php` for a complete refactoring example.

- when running command do not cd we are already at the root of the project
- i cannot add or chnage a migration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SterotECH) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
