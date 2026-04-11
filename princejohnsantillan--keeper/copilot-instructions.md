## keeper

> Use when component is used only within a single form/table/infolist, has custom queries, or complex layouts.

<laravel-boost-guidelines>
=== .ai/actions-and-services rules ===

# Actions and Services

## Purpose

Framework-agnostic business logic layer that:
- Centralizes testable logic outside controllers, Livewire, and Filament
- Enables UI swapping (Filament → Inertia, Livewire → API) without rewriting logic
- Maintains single source of truth for domain operations

---

## Actions

**Definition:** Single-responsibility invokable class performing one specific operation.

### Rules

- Must have single public `__invoke()` method
- Must NOT contain UI logic (no Filament/Livewire/HTTP code)
- Each action requires a corresponding feature test

### Location

| Type | Pattern | Location |
|------|---------|----------|
| Action | `{Verb}{Noun}Action` | `app/Actions/` |
| Test | `{Verb}{Noun}ActionTest` | `tests/Feature/Actions/` |

### Basic Structure

```php
declare(strict_types=1);

namespace App\Actions;

use App\Models\Child;

final class CreateChildAction
{
    public function __invoke(array $data): Child
    {
        return Child::query()->create($data);
    }
}
```

### With Dependencies

```php
declare(strict_types=1);

namespace App\Actions;

use App\Models\Child;
use App\Services\Contracts\NotificationServiceInterface;

final class CreateChildAction
{
    public function __construct(
        private NotificationServiceInterface $notificationService,
    ) {}

    public function __invoke(array $data): Child
    {
        $child = Child::query()->create($data);
        $this->notificationService->notifyGuardian($child);
        return $child;
    }
}
```

---

## Services

**Definition:** Library of related methods operating on a specific domain, grouping operations that don't warrant individual action classes.

### Rules

- Must have interface in `app/Services/Contracts/`
- Interface bound to implementation in `ServiceServiceProvider`
- Each public method requires a feature test
- Must NOT contain UI logic

### Location

| Type | Pattern | Location |
|------|---------|----------|
| Service | `{Domain}Service` | `app/Services/` |
| Interface | `{Domain}ServiceInterface` | `app/Services/Contracts/` |
| Test | `{Domain}ServiceTest` | `tests/Feature/Services/` |

### Interface

```php
declare(strict_types=1);

namespace App\Services\Contracts;

use App\Models\Child;
use App\Models\Guardian;

interface ChildServiceInterface
{
    public function create(array $data): Child;
    public function attachGuardian(Child $child, Guardian $guardian, string $relationship): void;
    public function detachGuardian(Child $child, Guardian $guardian): void;
}
```

### Implementation

```php
declare(strict_types=1);

namespace App\Services;

use App\Models\Child;
use App\Models\Guardian;
use App\Models\Relationship;
use App\Services\Contracts\ChildServiceInterface;

final class ChildService implements ChildServiceInterface
{
    public function create(array $data): Child
    {
        return Child::query()->create($data);
    }

    public function attachGuardian(Child $child, Guardian $guardian, string $relationship): void
    {
        Relationship::query()->create([
            'child_id' => $child->id,
            'guardian_id' => $guardian->id,
            'relationship' => $relationship,
        ]);
    }

    public function detachGuardian(Child $child, Guardian $guardian): void
    {
        Relationship::query()
            ->where('child_id', $child->id)
            ->where('guardian_id', $guardian->id)
            ->delete();
    }
}
```

### Binding in ServiceServiceProvider

```php
declare(strict_types=1);

namespace App\Providers;

use App\Services\ChildService;
use App\Services\Contracts\ChildServiceInterface;
use Illuminate\Support\ServiceProvider;

final class ServiceServiceProvider extends ServiceProvider
{
    public array $bindings = [
        ChildServiceInterface::class => ChildService::class,
    ];
}
```

---

## Testing

### Action Test

```php
use App\Actions\CreateChildAction;
use App\Models\Child;

it('creates a child', function () {
    $action = app(CreateChildAction::class);

    $child = $action([
        'first_name' => 'John',
        'last_name' => 'Doe',
    ]);

    expect($child)
        ->toBeInstanceOf(Child::class)
        ->first_name->toBe('John')
        ->last_name->toBe('Doe');

    $this->assertDatabaseHas(Child::class, [
        'first_name' => 'John',
        'last_name' => 'Doe',
    ]);
});
```

### Service Test

```php
use App\Models\Child;
use App\Models\Guardian;
use App\Services\Contracts\ChildServiceInterface;

it('attaches a guardian to a child', function () {
    $service = app(ChildServiceInterface::class);
    $child = Child::factory()->create();
    $guardian = Guardian::factory()->create();

    $service->attachGuardian($child, $guardian, 'parent');

    $this->assertDatabaseHas('relationships', [
        'child_id' => $child->id,
        'guardian_id' => $guardian->id,
        'relationship' => 'parent',
    ]);
});
```

### Mocking Services

```php
use App\Services\Contracts\NotificationServiceInterface;

it('sends notification when child is created', function () {
    $mock = $this->mock(NotificationServiceInterface::class);
    $mock->shouldReceive('notifyGuardian')->once();
    // Test code that triggers notification...
});
```

---

## Filament Integration

Filament UI actions (`app/Filament/Actions/`) must delegate to business logic actions/services. No direct model manipulation in Filament actions.

**Why:**
- Filament actions are UI components, not business logic
- Logic in Filament actions cannot be reused in APIs/other frameworks
- Action classes enable unit testing; Filament requires browser/Livewire testing

### Dependency Injection in Closures

Laravel auto-resolves actions in closure parameters:
```php
->using(fn (array $data, CreateChildAction $action): Child => $action($data))
```

### Avoid (business logic in Filament)

```php
// app/Filament/Actions/CreateChildAction.php
->using(function (array $data): Child {
    $relationship = $data['relationship'];
    unset($data['relationship']);
    $guardian = AuthUser::guardian();
    $child = Child::query()->create($data);
    Relationship::create([...]);
    return $child;
});
```

### Preferred (delegate to action)

```php
// app/Filament/Actions/CreateChildAction.php
->using(fn (array $data, CreateChildBusinessAction $action): Child => $action($data));
```

---

## Action vs Service Decision

| Scenario | Use |
|----------|-----|
| Single focused operation | Action |
| Multiple related domain operations | Service |
| Used in one place | Action |
| Shared across many features | Service |
| Simple CRUD, no side effects | Action |
| Complex multi-step orchestration | Action (inject services) |

---

## Migration Strategy

- **New features:** Use actions/services from start
- **Existing code:** Refactor incrementally when touching related code
- **Priority:** Extract logic from Filament actions, controllers, Livewire components

=== .ai/ai-guidelines rules ===

## Required for AI

- NO NEED TO CREATE TESTS FOR NOW.

=== .ai/filament rules ===

# Filament Guidelines

## Required for Planning

When in plan mode ALWAYS create a plan with Filament Blueprint.

## Core Principle

Separate configuration from layout. All component/column/entry attributes go in reusable App Components or private methods. Only layout concerns are chained in schema/columns arrays.

---

## Directory Structure

| Type | Location |
|------|----------|
| Filament Actions | `app/Filament/Actions/` |
| Form Components | `app/Filament/Components/Forms/` |
| Infolist Components | `app/Filament/Components/Infolists/` |
| Table Components | `app/Filament/Components/Tables/` |
| Business Actions | `app/Actions/` |
| Services | `app/Services/` |
| Service Interfaces | `app/Services/Contracts/` |
| Resources | `app/Filament/Panels/{Panel}/Resources/` |
| Notifications | `app/Filament/Notifications/` |

---

## Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Resource | `{Model}Resource` | `ChildResource` |
| Form Schema | `{Model}Form` | `ChildForm` |
| Infolist Schema | `{Model}Infolist` | `ChildInfolist` |
| Table Schema | `{PluralModel}Table` | `ChildrenTable` |
| Filament Action | `{Verb}{Noun}Action` | `CreateChildAction` |
| Business Action | `{Verb}{Noun}Action` | `CreateChildAction` |
| Form Component | `App{Type}` | `AppTextInput` |
| Infolist Component | `App{Type}` | `AppTextEntry` |
| Table Component | `App{Type}` | `AppTextColumn` |
| Service | `{Domain}Service` | `ChildService` |
| Service Interface | `{Domain}ServiceInterface` | `ChildServiceInterface` |

### Method Naming

- Static factory: `make()`
- Field/entry/column builder: Noun (`firstName()`, `birthDate()`, `avatar()`)
- Private complex component: `{field}Column()`, `{field}Entry()`
- Notification factory: Past participle (`registeredToActivity()`)

### Variable Naming

Never use `$action` for Filament action variables (reserved in callbacks). Use full class name in camelCase:
```php
$createChildAction = CreateChildAction::make(); // Correct
$action = CreateChildAction::make();            // WRONG
```

---

## Schema/Columns Array Rules

### Allowed (Layout Only)

```php
->columnSpanFull()
->columnSpan(int)
->columns(int)
->autofocus()
->toggleable()
->grow()
->alignment()
->width()
->hidden(Closure)
->visible(Closure)
->disabled(Closure) // conditional only
```

### NOT Allowed (Must Be in App Component or Private Method)

```php
// Form
->required(), ->email(), ->tel(), ->url(), ->options(), ->relationship()
->rules(), ->maxLength(), ->minLength(), ->default(), ->dehydrated()
->collection(), ->image(), ->avatar(), ->imageEditor(), ->circleCropper()
->displayFormat(), ->maxDate(), ->minDate(), ->native(false), ->inline()

// Infolist
->icon(), ->copyable(), ->placeholder(), ->date(), ->dateTime()
->markdown(), ->prose(), ->formatStateUsing(), ->size(), ->weight()
->badge(), ->circular(), ->defaultImageUrl()

// Table
->searchable(), ->sortable(), ->date(), ->dateTime(), ->badge()
->copyable(), ->description(), ->icon(), ->iconColor(), ->boolean()
->formatStateUsing(), ->size(), ->weight()
```

---

## App Component Structure

### Forms (`app/Filament/Components/Forms/`)

```php
declare(strict_types=1);

namespace App\Filament\Components\Forms;

use Filament\Forms\Components\TextInput;

final class AppTextInput
{
    public static function firstName(string $field = 'first_name', string $label = 'First name'): TextInput
    {
        return TextInput::make($field)->label($label)
            ->required()
            ->rules(['max:80']);
    }
}
```

### Infolists (`app/Filament/Components/Infolists/`)

```php
declare(strict_types=1);

namespace App\Filament\Components\Infolists;

use Filament\Infolists\Components\TextEntry;

final class AppTextEntry
{
    public static function email(string $field = 'email', string $label = 'Email'): TextEntry
    {
        return TextEntry::make($field)->label($label)
            ->icon('heroicon-o-envelope')
            ->copyable()
            ->placeholder('—');
    }
}
```

### Tables (`app/Filament/Components/Tables/`)

```php
declare(strict_types=1);

namespace App\Filament\Components\Tables;

use Filament\Tables\Columns\TextColumn;

final class AppTextColumn
{
    public static function firstName(string $field = 'first_name', string $label = 'First name'): TextColumn
    {
        return TextColumn::make($field)->label($label)
            ->searchable()
            ->sortable();
    }
}
```

---

## Usage Examples

### Correct Form Schema

```php
return $schema->components([
    AppSpatieMediaLibraryFileUpload::avatar()
        ->columnSpanFull(),
    AppTextInput::firstName()
        ->autofocus(),
    AppTextInput::lastName(),
    AppDatePicker::birthDate(),
    AppToggleButtons::gender(),
    AppTextarea::notes()
        ->columnSpanFull(),
]);
```

### Correct Table Columns

```php
return $table->columns([
    AppSpatieMediaLibraryImageColumn::avatar(),
    AppTextColumn::firstName(),
    AppTextColumn::lastName(),
    AppTextColumn::birthDate(),
    AppIconColumn::gender(),
    AppTagsColumn::tags()
        ->toggleable(),
]);
```

### Correct Infolist Schema

```php
return $schema->components([
    Section::make()
        ->schema([
            AppSpatieMediaLibraryImageEntry::avatar()
                ->columnSpanFull(),
            AppTextEntry::fullName(),
            AppTextEntry::birthday(),
            AppIconEntry::gender(),
        ]),
]);
```

### WRONG (Configuration in Schema)

```php
// DON'T DO THIS
return $schema->components([
    TextInput::make('first_name')
        ->required()          // Should be in App Component
        ->rules(['max:80']),  // Should be in App Component
]);
```

---

## Filament Actions

### Structure (`app/Filament/Actions/`)

```php
declare(strict_types=1);

namespace App\Filament\Actions;

use Filament\Actions\Action;

final class DoSomethingAction
{
    public static function make(?string $name = null, string $label = 'Do Something'): Action
    {
        return Action::make($name)->label($label)
            // configuration...
    }
}
```

### Business Logic Delegation

Filament actions delegate to business actions/services. No direct model manipulation.

```php
// Correct: Delegate to business action
->using(fn (array $data, CreateChildAction $createChild): Child => $createChild($data))

// WRONG: Business logic in Filament action
->using(function (array $data): Child {
    return Child::query()->create($data); // Should be in app/Actions/
});
```

---

## Business Actions (`app/Actions/`)

Single-responsibility invokable classes. No UI logic.

```php
declare(strict_types=1);

namespace App\Actions;

use App\Models\Child;

final class CreateChildAction
{
    public function __invoke(array $data): Child
    {
        return Child::query()->create($data);
    }
}
```

### With Dependencies

```php
final class CreateChildAction
{
    public function __construct(
        private NotificationServiceInterface $notificationService,
    ) {}

    public function __invoke(array $data): Child
    {
        $child = Child::query()->create($data);
        $this->notificationService->notifyGuardian($child);
        return $child;
    }
}
```

---

## Services (`app/Services/`)

Related domain operations grouped together. Must have interface.

### Interface (`app/Services/Contracts/`)

```php
interface ChildServiceInterface
{
    public function create(array $data): Child;
    public function attachGuardian(Child $child, Guardian $guardian, string $relationship): void;
}
```

### Implementation

```php
final class ChildService implements ChildServiceInterface
{
    public function create(array $data): Child
    {
        return Child::query()->create($data);
    }

    public function attachGuardian(Child $child, Guardian $guardian, string $relationship): void
    {
        Relationship::query()->create([
            'child_id' => $child->id,
            'guardian_id' => $guardian->id,
            'relationship' => $relationship,
        ]);
    }
}
```

### Binding (`app/Providers/ServiceServiceProvider.php`)

```php
public array $bindings = [
    ChildServiceInterface::class => ChildService::class,
];
```

---

## Private Methods for Complex/Context-Specific Components

Use when component is used only within a single form/table/infolist, has custom queries, or complex layouts.

### Form Example

```php
final class GatepassForm
{
    public static function configure(Schema $schema): Schema
    {
        return $schema->components([
            self::code(),
            self::activitySelect()
                ->columnSpanFull(),
        ])->columns(2);
    }

    private static function code(): TextInput
    {
        return TextInput::make('code')
            ->default(ReadableCode::generate())
            ->disabled()
            ->dehydrated()
            ->required();
    }

    private static function activitySelect(): Select
    {
        return Select::make('activity_id')
            ->relationship('activity', 'title')
            ->required();
    }
}
```

### Table Example (Custom Searchable/Sortable)

```php
private static function guardianFullNameColumn(): TextColumn
{
    return TextColumn::make('guardian.full_name')
        ->searchable(query: function (Builder $query, string $search): Builder {
            return $query->whereHas('guardian', function (Builder $query) use ($search): void {
                $query->where('first_name', 'ilike', "%{$search}%")
                    ->orWhere('last_name', 'ilike', "%{$search}%");
            });
        })
        ->sortable(query: function (Builder $query, string $direction): Builder {
            return $query
                ->join('guardians', 'gatepasses.guardian_id', '=', 'guardians.id')
                ->orderBy('guardians.first_name', $direction)
                ->select('gatepasses.*');
        });
}
```

---

## Decision Guide

| Scenario | Solution |
|----------|----------|
| Component used in 2+ forms/tables/infolists | App Component |
| Component used only in one place | Private static method |
| Custom searchable/sortable query | Private static method |
| Complex Stack/Split/Repeater layout | Private static method |
| Layout component (Section, Grid, Fieldset) | Inline in schema |
| Single focused operation | Business Action |
| Multiple related domain operations | Service |
| Simple CRUD, no side effects | Business Action |
| Complex multi-step orchestration | Business Action (inject services) |

---

## Available App Components

### Forms

| Class | Methods |
|-------|---------|
| `AppTextInput` | `firstName()`, `middleName()`, `lastName()`, `nickname()`, `email()`, `phone()`, `title()`, `name()`, `location()`, `type()` |
| `AppSelect` | `relationship()`, `term()`, `gender()` |
| `AppDatePicker` | `birthDate()` |
| `AppDateTimePicker` | `startsAt()`, `endsAt()`, `publishedAt()` |
| `AppTextarea` | `notes()`, `description()` |
| `AppToggleButtons` | `gender()` |
| `AppSpatieMediaLibraryFileUpload` | `avatar()`, `thumbnail()` |
| `AppTagsSelect` | `tags()` |
| `AppMarkdownEditor` | `content()` |

### Infolists

| Class | Methods |
|-------|---------|
| `AppTextEntry` | `firstName()`, `middleName()`, `lastName()`, `fullName()`, `nickname()`, `email()`, `phone()`, `age()`, `birthday()`, `notes()`, `title()`, `content()`, `createdAt()`, `updatedAt()` |
| `AppIconEntry` | `gender()` |
| `AppSpatieMediaLibraryImageEntry` | `avatar()`, `thumbnail()` |

### Tables

| Class | Methods |
|-------|---------|
| `AppTextColumn` | `firstName()`, `middleName()`, `lastName()`, `fullName()`, `nickname()`, `email()`, `phone()`, `birthDate()`, `title()`, `location()`, `code()`, `name()`, `type()`, `createdAt()`, `updatedAt()` |
| `AppIconColumn` | `gender()` |
| `AppSpatieMediaLibraryImageColumn` | `avatar()`, `thumbnail()` |
| `AppTagsColumn` | `tags()` |

=== .ai/naming-conventions rules ===

# Naming Conventions

## PHP Classes

- Always use `final class` for concrete implementations

## Class Patterns

| Type | Pattern | Example | Location |
|------|---------|---------|----------|
| Action (Business) | `{Verb}{Noun}Action` | `CreateChildAction` | `app/Actions/` |
| Action (Filament) | `{Verb}{Noun}Action` | `CreateChildAction` | `app/Filament/Actions/` |
| Service | `{Domain}Service` | `ChildService` | `app/Services/` |
| Service Interface | `{Domain}ServiceInterface` | `ChildServiceInterface` | `app/Services/Contracts/` |
| Panel Provider | `{Panel}PanelProvider` | `KeeperPanelProvider` | `app/Providers/` |
| Scope | `{Entity}Scope` | `OrganizationScope` | `app/Models/Scopes/` |

## Filament Patterns

| Type | Pattern | Example | Location |
|------|---------|---------|----------|
| Resource | `{Model}Resource` | `ChildResource` | `app/Filament/Panels/{Panel}/Resources/` |
| Resource Folder | Plural | `Children/`, `Guardians/` | - |
| Form | `{Model}Form` | `ChildForm` | `Resources/{Model}/Schemas/` |
| Infolist | `{Model}Infolist` | `ChildInfolist` | `Resources/{Model}/Schemas/` |
| Table | `{PluralModel}Table` | `ChildrenTable` | `Resources/{Model}/Tables/` |
| Form Component | `App{Type}` | `AppTextInput`, `AppSelect` | `app/Filament/Components/Forms/` |
| Infolist Component | `App{Type}` | `AppTextEntry`, `AppIconEntry` | `app/Filament/Components/Infolists/` |
| Table Component | `App{Type}` | `AppTextColumn`, `AppIconColumn` | `app/Filament/Components/Tables/` |
| Notification | `AppNotification` | - | `app/Filament/Notifications/` |

## Method Patterns

| Type | Convention | Example |
|------|------------|---------|
| Static factory | `make()` | `AttendActivityAction::make()` |
| Form field builder | Noun | `AppTextInput::firstName()` |
| Infolist entry builder | Noun | `AppTextEntry::fullName()` |
| Table column builder | Noun | `AppTextColumn::birthDate()` |
| Notification factory | Past participle | `AppNotification::registeredToActivity()` |
| Qualified relationship | Qualifier + noun | `checkinAttendance()`, `ownedOrganizations()` |
| Complex column (private) | `{field}Column()` | `self::childrenColumn()` |
| Complex entry (private) | `{field}Entry()` | `self::descriptionEntry()` |

## Variable Naming

| Type | Convention | Example |
|------|------------|---------|
| Filament action variable | Full class name in camelCase | `$createChildAction = CreateChildAction::make()` |

**Note:** Never use `$action` as a variable name for Filament actions. It is a reserved parameter name in Filament callbacks. See [Filament Actions guidelines](./filament-actions.md) for details.

=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to ensure the best experience when building Laravel applications.

## Foundational Context

This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.4
- filament/filament (FILAMENT) - v5
- laravel/framework (LARAVEL) - v12
- laravel/horizon (HORIZON) - v5
- laravel/nightwatch (NIGHTWATCH) - v1
- laravel/prompts (PROMPTS) - v0
- livewire/livewire (LIVEWIRE) - v4
- larastan/larastan (LARASTAN) - v3
- laravel/boost (BOOST) - v2
- laravel/mcp (MCP) - v0
- laravel/pail (PAIL) - v1
- laravel/pint (PINT) - v1
- pestphp/pest (PEST) - v4
- phpunit/phpunit (PHPUNIT) - v12
- tailwindcss (TAILWINDCSS) - v4

## Skills Activation

This project has domain-specific skills available. You MUST activate the relevant skill whenever you work in that domain—don't wait until you're stuck.

- `laravel-best-practices` — Apply this skill whenever writing, reviewing, or refactoring Laravel PHP code. This includes creating or modifying controllers, models, migrations, form requests, policies, jobs, scheduled commands, service classes, and Eloquent queries. Triggers for N+1 and query performance issues, caching strategies, authorization and security patterns, validation, error handling, queue and job configuration, route definitions, and architectural decisions. Also use for Laravel code reviews and refactoring existing Laravel code to follow best practices. Covers any task involving Laravel backend PHP code patterns.
- `configuring-horizon` — Use this skill whenever the user mentions Horizon by name in a Laravel context. Covers the full Horizon lifecycle: installing Horizon (horizon:install, Sail setup), configuring config/horizon.php (supervisor blocks, queue assignments, balancing strategies, minProcesses/maxProcesses), fixing the dashboard (authorization via Gate::define viewHorizon, blank metrics, horizon:snapshot scheduling), and troubleshooting production issues (worker crashes, timeout chain ordering, LongWaitDetected notifications, waits config). Also covers job tagging and silencing. Do not use for generic Laravel queues without Horizon, SQS or database drivers, standalone Redis setup, Linux supervisord, Telescope, or job batching.
- `configure-nightwatch` — Configures Laravel Nightwatch data collection, sampling rates, filtering rules, and redaction policies. Use when setting up Nightwatch, managing data volume, protecting sensitive data (PII), or optimizing event collection for production workloads.
- `pest-testing` — Use this skill for Pest PHP testing in Laravel projects only. Trigger whenever any test is being written, edited, fixed, or refactored — including fixing tests that broke after a code change, adding assertions, converting PHPUnit to Pest, adding datasets, and TDD workflows. Always activate when the user asks how to write something in Pest, mentions test files or directories (tests/Feature, tests/Unit, tests/Browser), or needs browser testing, smoke testing multiple pages for JS errors, or architecture tests. Covers: test()/it()/expect() syntax, datasets, mocking, browser testing (visit/click/fill), smoke testing, arch(), Livewire component tests, RefreshDatabase, and all Pest 4 features. Do not use for factories, seeders, migrations, controllers, models, or non-test PHP code.
- `tailwindcss-development` — Always invoke when the user's message includes 'tailwind' in any form. Also invoke for: building responsive grid layouts (multi-column card grids, product grids), flex/grid page structures (dashboards with sidebars, fixed topbars, mobile-toggle navs), styling UI components (cards, tables, navbars, pricing sections, forms, inputs, badges), adding dark mode variants, fixing spacing or typography, and Tailwind v3/v4 work. The core use case: writing or fixing Tailwind utility classes in HTML templates (Blade, JSX, Vue). Skip for backend PHP logic, database queries, API routes, JavaScript with no HTML/CSS component, CSS file audits, build tool configuration, and vanilla CSS.
- `medialibrary-development` — Build and work with spatie/laravel-medialibrary features including associating files with Eloquent models, defining media collections and conversions, generating responsive images, and retrieving media URLs and paths.
- `filament-blueprint` — Filament Blueprint is a guideline in how to write best practice implementation plans for Filament v5. ALWAYS activates when in plan mode.

## Conventions

- You must follow all existing code conventions used in this application. When creating or editing a file, check sibling files for the correct structure, approach, and naming.
- Use descriptive names for variables and methods. For example, `isRegisteredForDiscounts`, not `discount()`.
- Check for existing components to reuse before writing a new one.

## Verification Scripts

- Do not create verification scripts or tinker when tests cover that functionality and prove they work. Unit and feature tests are more important.

## Application Structure & Architecture

- Stick to existing directory structure; don't create new base folders without approval.
- Do not change the application's dependencies without approval.

## Frontend Bundling

- If the user doesn't see a frontend change reflected in the UI, it could mean they need to run `npm run build`, `npm run dev`, or `composer run dev`. Ask them.

## Documentation Files

- You must only create documentation files if explicitly requested by the user.

## Replies

- Be concise in your explanations - focus on what's important rather than explaining obvious details.

=== boost rules ===

# Laravel Boost

## Tools

- Laravel Boost is an MCP server with tools designed specifically for this application. Prefer Boost tools over manual alternatives like shell commands or file reads.
- Use `database-query` to run read-only queries against the database instead of writing raw SQL in tinker.
- Use `database-schema` to inspect table structure before writing migrations or models.
- Use `get-absolute-url` to resolve the correct scheme, domain, and port for project URLs. Always use this before sharing a URL with the user.
- Use `browser-logs` to read browser logs, errors, and exceptions. Only recent logs are useful, ignore old entries.

## Searching Documentation (IMPORTANT)

- Always use `search-docs` before making code changes. Do not skip this step. It returns version-specific docs based on installed packages automatically.
- Pass a `packages` array to scope results when you know which packages are relevant.
- Use multiple broad, topic-based queries: `['rate limiting', 'routing rate limiting', 'routing']`. Expect the most relevant results first.
- Do not add package names to queries because package info is already shared. Use `test resource table`, not `filament 4 test resource table`.

### Search Syntax

1. Use words for auto-stemmed AND logic: `rate limit` matches both "rate" AND "limit".
2. Use `"quoted phrases"` for exact position matching: `"infinite scroll"` requires adjacent words in order.
3. Combine words and phrases for mixed queries: `middleware "rate limit"`.
4. Use multiple queries for OR logic: `queries=["authentication", "middleware"]`.

## Artisan

- Run Artisan commands directly via the command line (e.g., `php artisan route:list`). Use `php artisan list` to discover available commands and `php artisan [command] --help` to check parameters.
- Inspect routes with `php artisan route:list`. Filter with: `--method=GET`, `--name=users`, `--path=api`, `--except-vendor`, `--only-vendor`.
- Read configuration values using dot notation: `php artisan config:show app.name`, `php artisan config:show database.default`. Or read config files directly from the `config/` directory.
- To check environment variables, read the `.env` file directly.

## Tinker

- Execute PHP in app context for debugging and testing code. Do not create models without user approval, prefer tests with factories instead. Prefer existing Artisan commands over custom tinker code.
- Always use single quotes to prevent shell expansion: `php artisan tinker --execute 'Your::code();'`
  - Double quotes for PHP strings inside: `php artisan tinker --execute 'User::where("active", true)->count();'`

=== php rules ===

# PHP

- Always declare `declare(strict_types=1);` at the top of every `.php` file.
- Always use curly braces for control structures, even for single-line bodies.
- Use PHP 8 constructor property promotion: `public function __construct(public GitHub $github) { }`. Do not leave empty zero-parameter `__construct()` methods unless the constructor is private.
- Use explicit return type declarations and type hints for all method parameters: `function isAccessible(User $user, ?string $path = null): bool`
- Use TitleCase for Enum keys: `FavoritePerson`, `BestLake`, `Monthly`.
- Prefer PHPDoc blocks over inline comments. Only add inline comments for exceptionally complex logic.
- Use array shape type definitions in PHPDoc blocks.

=== herd rules ===

# Laravel Herd

- The application is served by Laravel Herd at `https?://[kebab-case-project-dir].test`. Use the `get-absolute-url` tool to generate valid URLs. Never run commands to serve the site. It is always available.
- Use the `herd` CLI to manage services, PHP versions, and sites (e.g. `herd sites`, `herd services:start <service>`, `herd php:list`). Run `herd list` to discover all available commands.

=== laravel/core rules ===

# Do Things the Laravel Way

- Use `php artisan make:` commands to create new files (i.e. migrations, controllers, models, etc.). You can list available Artisan commands using `php artisan list` and check their parameters with `php artisan [command] --help`.
- If you're creating a generic PHP class, use `php artisan make:class`.
- Pass `--no-interaction` to all Artisan commands to ensure they work without user input. You should also pass the correct `--options` to ensure correct behavior.

### Model Creation

- When creating new models, create useful factories and seeders for them too. Ask the user if they need any other things, using `php artisan make:model --help` to check the available options.

## APIs & Eloquent Resources

- For APIs, default to using Eloquent API Resources and API versioning unless existing API routes do not, then you should follow existing application convention.

## URL Generation

- When generating links to other pages, prefer named routes and the `route()` function.

## Testing

- When creating models for tests, use the factories for the models. Check if the factory has custom states that can be used before manually setting up the model.
- Faker: Use methods such as `$this->faker->word()` or `fake()->randomDigit()`. Follow existing conventions whether to use `$this->faker` or `fake()`.
- When creating tests, make use of `php artisan make:test [options] {name}` to create a feature test, and pass `--unit` to create a unit test. Most tests should be feature tests.

## Vite Error

- If you receive an "Illuminate\Foundation\ViteException: Unable to locate file in Vite manifest" error, you can run `npm run build` or ask the user to run `npm run dev` or `composer run dev`.

## Deployment

- Laravel can be deployed using [Laravel Cloud](https://cloud.laravel.com/), which is the fastest way to deploy and scale production Laravel applications.

=== laravel/v12 rules ===

# Laravel 12

- CRITICAL: ALWAYS use `search-docs` tool for version-specific Laravel documentation and updated code examples.
- Since Laravel 11, Laravel has a new streamlined file structure which this project uses.

## Laravel 12 Structure

- In Laravel 12, middleware are no longer registered in `app/Http/Kernel.php`.
- Middleware are configured declaratively in `bootstrap/app.php` using `Application::configure()->withMiddleware()`.
- `bootstrap/app.php` is the file to register middleware, exceptions, and routing files.
- `bootstrap/providers.php` contains application specific service providers.
- The `app/Console/Kernel.php` file no longer exists; use `bootstrap/app.php` or `routes/console.php` for console configuration.
- Console commands in `app/Console/Commands/` are automatically available and do not require manual registration.

## Database

- When modifying a column, the migration must include all of the attributes that were previously defined on the column. Otherwise, they will be dropped and lost.
- Laravel 12 allows limiting eagerly loaded records natively, without external packages: `$query->latest()->limit(10);`.

### Models

- Casts can and likely should be set in a `casts()` method on a model rather than the `$casts` property. Follow existing conventions from other models.

=== pint/core rules ===

# Laravel Pint Code Formatter

- If you have modified any PHP files, you must run `vendor/bin/pint --dirty --format agent` before finalizing changes to ensure your code matches the project's expected style.
- Do not run `vendor/bin/pint --test --format agent`, simply run `vendor/bin/pint --format agent` to fix any formatting issues.

=== pest/core rules ===

## Pest

- This project uses Pest for testing. Create tests: `php artisan make:test --pest {name}`.
- Run tests: `php artisan test --compact` or filter: `php artisan test --compact --filter=testName`.
- Do NOT delete tests without approval.

=== filament/filament rules ===

## Filament

- Filament is used by this application. Follow the existing conventions for how and where it is implemented.
- Filament is a Server-Driven UI (SDUI) framework for Laravel that lets you define user interfaces in PHP using structured configuration objects. Built on Livewire, Alpine.js, and Tailwind CSS.
- Use the `search-docs` tool for official documentation on Artisan commands, code examples, testing, relationships, and idiomatic practices. If `search-docs` is unavailable, refer to https://filamentphp.com/docs.

### Artisan

- Always use Filament-specific Artisan commands to create files. Find available commands with the `list-artisan-commands` tool, or run `php artisan --help`.
- Always inspect required options before running a command, and always pass `--no-interaction`.

### Patterns

Always use static `make()` methods to initialize components. Most configuration methods accept a `Closure` for dynamic values.

Use `Get $get` to read other form field values for conditional logic:

<code-snippet name="Conditional form field visibility" lang="php">
use Filament\Forms\Components\Select;
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Components\Utilities\Get;

Select::make('type')
    ->options(CompanyType::class)
    ->required()
    ->live(),

TextInput::make('company_name')
    ->required()
    ->visible(fn (Get $get): bool => $get('type') === 'business'),

</code-snippet>

Use `state()` with a `Closure` to compute derived column values:

<code-snippet name="Computed table column value" lang="php">
use Filament\Tables\Columns\TextColumn;

TextColumn::make('full_name')
    ->state(fn (User $record): string => "{$record->first_name} {$record->last_name}"),

</code-snippet>

Actions encapsulate a button with an optional modal form and logic:

<code-snippet name="Action with modal form" lang="php">
use Filament\Actions\Action;
use Filament\Forms\Components\TextInput;

Action::make('updateEmail')
    ->schema([
        TextInput::make('email')
            ->email()
            ->required(),
    ])
    ->action(fn (array $data, User $record) => $record->update($data))

</code-snippet>

### Testing

Always authenticate before testing panel functionality. Filament uses Livewire, so use `Livewire::test()` or `livewire()` (available when `pestphp/pest-plugin-livewire` is in `composer.json`):

<code-snippet name="Table test" lang="php">
use function Pest\Livewire\livewire;

livewire(ListUsers::class)
    ->assertCanSeeTableRecords($users)
    ->searchTable($users->first()->name)
    ->assertCanSeeTableRecords($users->take(1))
    ->assertCanNotSeeTableRecords($users->skip(1));

</code-snippet>

<code-snippet name="Create resource test" lang="php">
use function Pest\Laravel\assertDatabaseHas;
use function Pest\Livewire\livewire;

livewire(CreateUser::class)
    ->fillForm([
        'name' => 'Test',
        'email' => 'test@example.com',
    ])
    ->call('create')
    ->assertNotified()
    ->assertRedirect();

assertDatabaseHas(User::class, [
    'name' => 'Test',
    'email' => 'test@example.com',
]);

</code-snippet>

<code-snippet name="Testing validation" lang="php">
use function Pest\Livewire\livewire;

livewire(CreateUser::class)
    ->fillForm([
        'name' => null,
        'email' => 'invalid-email',
    ])
    ->call('create')
    ->assertHasFormErrors([
        'name' => 'required',
        'email' => 'email',
    ])
    ->assertNotNotified();

</code-snippet>

<code-snippet name="Calling actions in pages" lang="php">
use Filament\Actions\DeleteAction;
use function Pest\Livewire\livewire;

livewire(EditUser::class, ['record' => $user->id])
    ->callAction(DeleteAction::class)
    ->assertNotified()
    ->assertRedirect();

</code-snippet>

<code-snippet name="Calling actions in tables" lang="php">
use Filament\Actions\Testing\TestAction;
use function Pest\Livewire\livewire;

livewire(ListUsers::class)
    ->callAction(TestAction::make('promote')->table($user), [
        'role' => 'admin',
    ])
    ->assertNotified();

</code-snippet>

### Correct Namespaces

- Form fields (`TextInput`, `Select`, etc.): `Filament\Forms\Components\`
- Infolist entries (`TextEntry`, `IconEntry`, etc.): `Filament\Infolists\Components\`
- Layout components (`Grid`, `Section`, `Fieldset`, `Tabs`, `Wizard`, etc.): `Filament\Schemas\Components\`
- Schema utilities (`Get`, `Set`, etc.): `Filament\Schemas\Components\Utilities\`
- Actions (`DeleteAction`, `CreateAction`, etc.): `Filament\Actions\`. Never use `Filament\Tables\Actions\`, `Filament\Forms\Actions\`, or any other sub-namespace for actions.
- Icons: `Filament\Support\Icons\Heroicon` enum (e.g., `Heroicon::PencilSquare`)

### Common Mistakes

- **Never assume public file visibility.** File visibility is `private` by default. Always use `->visibility('public')` when public access is needed.
- **Never assume full-width layout.** `Grid`, `Section`, and `Fieldset` do not span all columns by default. Explicitly set column spans when needed.

=== spatie/laravel-medialibrary rules ===

## Media Library

- `spatie/laravel-medialibrary` associates files with Eloquent models, with support for collections, conversions, and responsive images.
- Always activate the `medialibrary-development` skill when working with media uploads, conversions, collections, responsive images, or any code that uses the `HasMedia` interface or `InteractsWithMedia` trait.

=== filament/blueprint rules ===

## Filament Blueprint

You are writing Filament v5 implementation plans. Plans must be specific enough
that an implementing agent can write code without making decisions.

**Start here**: Read
`/vendor/filament/blueprint/resources/markdown/planning/overview.md` for plan format,
required sections, and what to clarify with the user before planning.

</laravel-boost-guidelines>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/princejohnsantillan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/princejohnsantillan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
