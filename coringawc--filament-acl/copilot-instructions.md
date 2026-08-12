## filament-acl

> This repository contains the `coringawc/filament-acl` package.

# AGENTS.md

## Purpose

This repository contains the `coringawc/filament-acl` package.

The package solves contextual authorization for Filament v4 or v5 by treating the Filament owner as the permission subject:

- `Resource`
- `RelationManager`
- `Page`
- `Widget`
- free-form custom permissions

This package is intentionally generic. It may be inspired by ideas from `filament-shield`, but it must not depend on Shield at runtime.

## Architectural Rules

### Trait-First

Do not introduce required `BaseResource`, `BaseRelationManager`, `BasePage`, or `BaseWidget` classes unless absolutely necessary.

The preferred integration surface is:

- `HasResourcePermissions`
- `HasRelationManagerPermissions`
- `HasPagePermissions`
- `HasWidgetPermissions`

If a new feature can be implemented through traits, helper services, discovery, or configuration, prefer that.

### Automatic First, Override By Method

Defaults should work without extra boilerplate.

The following methods are optional overrides and should stay optional:

- `getPermissionSubject(): ?string`
- `shouldRegisterPermissions(): bool`
- `getSharedPermissionOwner(): ?string`
- `getPermissionCustomActions(): array`
- `getPermissionActions(): array`
- `getPermissionPanel(): ?string`

Do not add required static properties when a method override is enough.

### Attribute Resolution Is Concrete-Class Only

Permission attributes are read from the concrete class only.

This applies to attributes such as:

- `#[PermissionSubject(...)]`
- `#[PermissionActions([...])]`
- `#[CustomPermissionActions([...])]`
- `#[RegisterPermissions(false)]`

Do not assume these attributes are inherited from a parent Resource, RelationManager, Page, or Widget. If a child class needs the same permission metadata, redeclare the attribute or override the corresponding method.

### Keep Permission Naming Consistent

Use `Permission` and `Action` terminology in public APIs.

Avoid reintroducing public `Acl*` API names except where the package/plugin identity itself already uses `FilamentAcl`.

### No Shield Coupling

The package may copy or reinterpret good DX ideas from Shield, such as:

- install command ergonomics
- stub publishing
- resource-based role management UI

But it must remain independent:

- no `filament-shield` Composer dependency
- no runtime calls into Shield classes
- no config contract that assumes Shield is present

### Generic Package, App-Specific Decisions Stay Outside

Keep the package focused on contextual permission infrastructure.

Good package responsibilities:

- discovering opted-in Filament owners
- building subjects
- building permission keys
- syncing permissions
- panel scoping
- protected-role handling
- built-in role/permissions resource

Responsibilities that usually belong to the consuming app:

- domain-specific policy logic
- custom action business rules
- app-specific naming conventions when defaults are not enough
- extra role metadata

## Policy Contract

Policies remain native Laravel policies.

Package checks are opt-in through `ChecksPermission`.

Typical pattern:

```php
if ($response = $this->denyUnlessPermitted($user, 'update', $permissionAction)) {
    return $response;
}

// domain rules after permission passes
return Response::allow();
```

Keep the extra permission argument last in the signature:

- `viewAny(mixed $user, PermissionAction|string|null $permissionAction = null)`
- `update(mixed $user, Model $record, PermissionAction|string|null $permissionAction = null)`

Never make policies infer the owner from the request or route when the owner can be passed explicitly.

## Custom Action Contract

Do not create a custom Filament action class for the package unless there is no other safe path.

The intended usage is:

```php
auth()->user()?->can('archive', [$record, PostResource::class]);
```

and:

```php
->authorize('archive', PostResource::class)
```

Preserve this native Laravel and Filament style.

## Shared Owners

Shared permissions are a first-class feature.

When an owner returns another owner class from `getSharedPermissionOwner()`:

- it should inherit that owner's permissions
- it should usually disappear from package discovery and permission UI
- the shared owner becomes the canonical visible entry

Do not break this behavior by reintroducing duplicate visible tabs or duplicate sync rows unless the feature is explicitly about surfacing shared ownership better in the UI.

## Opt-In And Opt-Out

The package defaults to explicit opt-in through config:

- `filament-acl.integration.require_explicit_opt_in = true`

When an owner returns `shouldRegisterPermissions(): false`:

- it must not be synced
- it must not be displayed in the permission UI
- package-level permission checks should be skipped for that owner

This is essential. Do not regress it.

## Panel Scope

Panel scope is configurable separately for:

- roles
- permissions

Changes to panel-scope behavior must always be reflected consistently across:

- sync commands
- built-in resource queries
- hidden-role helpers
- permission lookup helpers
- protected-role assignment

Do not change one layer without checking the others.

## Protected Role

The protected role is intentionally special.

Expected behavior:

- hidden from package UI when configured
- optionally bypasses package-level checks via `Gate::before()`
- protected from edit/delete by `RolePolicy`
- assignable through the admin-user command

Do not expose the protected role in normal selects or package tables unless the feature explicitly requires it and is configurable.

## Built-In Permissions Resource

The built-in resource manages roles and their assigned permissions.

Important expectations:

- opt-in owners only
- respects shared owners
- respects hidden protected role
- supports resource, relation manager, page, widget, and custom-permission tabs
- supports managing another panel through `getManagedPermissionPanel()` and plugin configuration

When improving the UI, prefer staying close to the ergonomics used in the `siasgfacil-filament` project:

- grouped, navigable permission sections
- nested-resource awareness
- page/widget/custom-permission visibility
- no duplicated owners when permissions are shared

## Commands

Current commands:

- `filament-acl:install`
- `filament-acl:sync`
- `filament-acl:admin-user`

Production safety matters. Commands are prohibited in production by default.

If you change command behavior, keep these goals:

- no destructive silent overwrites
- detect existing config and migration files
- ask before replacing when interactive
- work with UUID, ULID, string, and integer user keys

## Utilities

`Support\\Utils` is intentionally public-facing.

Before adding a new helper, ask:

- is this reused by multiple package subsystems?
- is this likely useful to consuming applications?

If yes, `Utils` is a good home.

If a helper is purely local to one class, keep it local.

## Translations

The package uses `filament-acl::filament-acl.*` keys for all user-facing labels.

Ability label resolution follows this chain:

1. `permission_labels.{camelCase}` (e.g., `permission_labels.viewAny`)
2. `permission_labels.{snake_case}` (e.g., `permission_labels.view_any`)
3. `Str::headline($ability)` fallback

When adding new abilities, add both camelCase and snake_case keys to `resources/lang/{locale}/filament-acl.php` under `permission_labels`.

Section toggle labels live under `resources.permissions.edit.tabs.section_toggle`.

For relation-manager custom actions, preserve compatibility between camelCase action names declared in PHP and snake_case ability keys used by policies and translations.

## Subject Resolution Strategy

`SubjectResolutionStrategy` is a backed enum at `CoringaWc\FilamentAcl\Enums\SubjectResolutionStrategy`.

Values:

- `Basename` — derive subject from class basename minus suffix
- `Fqcn` — use the fully qualified class name
- `Custom` — defer to a registered callback

This enum is not yet integrated into the config. When implementing the integration, update `FluentSubjectResolver` to read the strategy from config and switch behavior accordingly.

## Permission Actions Configuration

Keep `filament-acl.relation_managers.actions` aligned with Filament's inherent relation-manager authorization surface. This includes the broader action set used by the package today:

- `viewAny`
- `view`
- `create`
- `update`
- `delete`
- `deleteAny`
- `forceDelete`
- `forceDeleteAny`
- `restore`
- `restoreAny`
- `replicate`
- `reorder`
- `associate`
- `attach`
- `detach`
- `detachAny`
- `dissociate`
- `dissociateAny`

At the owner level, prefer narrowing the real permission surface with `getPermissionActions()` when a specific relation manager only exposes a subset of those actions.

Recent behavior that must remain documented and preserved:

- relation-manager permission tabs resolve labels from `getTitle()` first and `getRelationshipTitle()` as fallback
- relation-manager permission tabs resolve icons from `getIcon()`
- relation-manager permission-tab badges reflect the number of permission options for that owner
- `RegisterPermissions(false)` is the package-native way to hide an owner from sync and the permissions UI

Default permission actions are defined per owner type in config:

- `filament-acl.policies.methods` — resource actions (used by `DefaultPermissionActionRegistry::forResource()`)
- `filament-acl.relation_managers.actions` — relation manager actions (includes RM-specific actions like associate, attach, detach, etc.)
- `filament-acl.pages.actions` — page actions (default: `['view']`)
- `filament-acl.widgets.actions` — widget actions (default: `['view']`)

```php
'actions' => [
    'viewAny',
    'view',
    'create',
    'update',
    'delete',
],
```

`DefaultPermissionActionRegistry` reads these config keys and supplies defaults to each owner type. Resources merge these with any custom actions from `getPermissionCustomActions()`.

Each owner trait exposes `getPermissionActions()` which returns the final merged, deduplicated list of actions. Override this method only when you need to completely replace the action list for a specific owner, or use `#[PermissionActions([...])]` when you want the same behavior declaratively on the concrete class.

### Exclude Lists

Each non-resource owner type supports an `exclude` config key to remove specific classes from sync and UI discovery:

- `filament-acl.relation_managers.exclude`
- `filament-acl.pages.exclude`
- `filament-acl.widgets.exclude`

Classes listed in `exclude` are ignored even if they use the package traits.

## Core Runtime Services

The package resolves three core services through the Laravel container:

- `ResolvesPermissionSubject` → `ConfiguredPermissionSubjectResolver` (default)
- `BuildsPermissionKey` → `DefaultPermissionKeyBuilder` (default)
- `StoresPermissions` → `SpatiePermissionStore` (default)

These are registered as singletons in `FilamentPermissionServiceProvider::packageRegistered()`. To override, bind your own implementation in your application's service provider:

```php
$this->app->singleton(ResolvesPermissionSubject::class, MyCustomResolver::class);
```

The consuming app's provider registers after the package, so the container will use the app's binding.

> **Note:** These services were previously configurable via `subject_resolver`, `permission_key_builder`, `permission_store`, and `callbacks` config keys. Those keys have been removed. Use container bindings or the `FilamentPermissionManager` facade instead.

## Permissions Resource Table Customization

The plugin exposes `configurePermissionsTable(Closure $callback)` for customizing the built-in permissions resource table:

```php
FilamentAclPlugin::make()
    ->permissionsResource()
    ->configurePermissionsTable(function (Table $table): Table {
        return $table->defaultSort('name');
    })
```

The closure receives a `Table` instance after the package has applied its default columns and actions. Use this to add filters, change sorting, or modify columns without overriding the entire resource.

Similarly, `configurePermissionsResource(Closure $callback)` allows customizing the `PermissionResourceConfiguration` object before the resource registers with the panel.

## Table Defaults Pattern

Filament's `Table::configureUsing()` applies global defaults to every table in the panel. The workbench uses this in `WorkbenchServiceProvider::boot()`:

```php
Table::configureUsing(static function (Table $table): void {
    $table->recordUrl(null);
});
```

This is a useful pattern for setting consistent defaults like disabling record URLs, enabling striped rows, or setting default pagination. Register it in a service provider's `boot()` method.

## Seeder Translation Pattern

Workbench seeders use `__('workbench::workbench.seeds.*')` for all user-facing strings (names, titles, content). Emails and technical identifiers remain hardcoded.

Translation keys live in `workbench/lang/{locale}/workbench.php` under the `seeds` key:

```php
'seeds' => [
    'users' => ['admin' => ['name' => 'João Silva'], ...],
    'posts' => ['draft' => ['title' => 'Post Rascunho', 'content' => '...'], ...],
    'comments' => ['draft_1' => 'Comentário do rascunho...', ...],
    'categories' => ['announcements' => ['name' => 'Anúncios', ...], ...],
],
```

When adding new seeded data, follow this pattern:

- add translation keys to both `en` and `pt_BR` files
- use `__()` in the seeder for any user-visible text
- keep technical identifiers (emails, slugs, status values) as literal strings

## Relation Manager Delegation

Relation managers can delegate authorization to a related resource instead of maintaining their own permission set. This is controlled by:

- `filament-acl.relation_managers.delegate_to_related_resource_by_default` (default: `false`)
- `HasRelationManagerPermissions::shouldUseRelatedResourcePermissions()` — reads the config key above

When `true`, a relation manager that defines a related resource will use that resource's permissions instead of its own. Individual relation managers can override this by implementing `shouldUseRelatedResourcePermissions()` themselves.

## Custom Permission Translation Pattern

Custom permission labels in config support translation through `__()`.

The `PermissionResource::getCustomPermissionOptions()` method wraps every label with `__()` before rendering. This means config labels should be **translation keys**, not literal human-readable strings:

```php
// config/filament-acl.php — CORRECT
'custom_permissions' => [
    'content.export' => 'acl::permissions.custom.export',  // translation key
    [
        'name' => 'content.publish',
        'label' => 'acl::permissions.custom.publish',      // translation key
    ],
],

// config/filament-acl.php — WRONG (hardcoded English)
'custom_permissions' => [
    'content.export' => 'Export content',       // will display English regardless of locale
],
```

The resolution chain is:

1. If a label is provided (string or array with `label` key), `__($label)` is called
2. If no label is provided, the permission name itself is used as fallback via `__($name)`

When adding custom permissions, always provide translation keys and add corresponding entries to the application's language files.

## Workbench

The workbench is not throwaway.

It is a real package test harness and should stay healthy.

Current workbench goals:

- locale `pt_BR`
- faker locale `pt_BR`
- real Filament panel
- nested resources
- relation managers
- page example
- widget example (non-lazy for HTTP test compatibility)
- built-in permissions resource enabled
- seeded roles, users, and permissions
- custom permissions with translation keys (`workbench::workbench.custom_permissions.*`)

Infrastructure notes:

- `testbench.yaml` uses `:memory:` SQLite for tests; `workbench/.env` overrides to file-based SQLite and file session for `serve`
- `testbench.yaml` must have `discovers.config: true` nested under the `workbench:` key (NOT at root level) — this loads `workbench/config/*.php` automatically
- `docker-compose.yml` sets `PHP_CLI_SERVER_WORKERS=4` to handle concurrent Livewire requests
- `testbench.yaml` provider names must use single quotes with single backslashes (YAML single-quote strings are literal)
- `TestCase::setUp()` registers `DataStore` as singleton to work around Filament's `bind()` registration

If you change runtime behavior, add or update workbench coverage rather than relying only on isolated unit tests.

## Testing Standards

Before finalizing changes:

1. Run focused Pest coverage for the affected area.
2. Run the full Pest suite.
3. Run PHPStan.
4. Run Pint.

Preferred commands in this repository:

```bash
docker compose exec php vendor/bin/pest --ci
docker compose exec php vendor/bin/phpstan analyse --memory-limit=1G
docker compose exec php vendor/bin/pint --dirty
```

When touching workbench behavior, make sure smoke tests continue to pass.

## Documentation Expectations

Keep the README accurate.

Whenever you change public API, review these README sections:

- installation
- plugin registration
- trait usage
- policy usage
- commands
- permission resource
- panel scoping
- workbench usage

If you add new behavior that future agents need in order to work safely, update this file too.

---
> Source: [CoringaWc/filament-acl](https://github.com/CoringaWc/filament-acl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
