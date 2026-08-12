## filament-relation-pages

> Generates:

# filament-relation-pages — AI Agent Guide

This document provides essential context for AI coding agents working on this package.

---

## Project Overview

**filament-relation-pages** is a Filament plugin that lets you add fully custom, free-form tabs alongside Relation Managers on any Filament resource page. Unlike `RelationManager`, a `RelationPage` has no forced table and no forced relationship — you put whatever you want inside it (Filament schema components, plain HTML, Alpine.js, etc.).

**Package Name**: `gheith3/filament-relation-pages`
**Filament Versions**: ^4.0 | ^5.0
**Main Class**: `RelationPage` (abstract base class that app code extends)

---

## How It Works (Filament Internals)

Filament's `HasRelationManagers` trait calls three static methods on **every entry** in `getRelations()`:

```
canViewForRecord(Model $ownerRecord, string $pageClass): bool
getTabComponent(Model $ownerRecord, string $pageClass): Tab
getDefaultProperties(): array
```

Then it mounts the class as a Livewire component, injecting `ownerRecord` and `pageClass`.

`RelationPage` implements all three methods and declares the two `#[Locked]` Livewire properties. That is the **entire contract** — no interface, no panel registration needed. Just add the class to `getRelations()`.

Key Filament vendor files (for reference when Filament updates):
```
vendor/filament/filament/src/Resources/Pages/Concerns/HasRelationManagers.php
vendor/filament/filament/src/Resources/RelationManagers/RelationManager.php
vendor/filament/schemas/src/Concerns/InteractsWithSchemas.php
```

---

## Directory Structure

```
filament-relation-pages/
├── src/
│   ├── RelationPage.php                  ← THE core class — abstract, app code extends this
│   ├── RelationPagesPlugin.php           ← implements Filament\Contracts\Plugin
│   ├── RelationPagesServiceProvider.php  ← registers command + publishes stubs
│   └── Commands/
│       └── MakeRelationPageCommand.php   ← artisan make:filament-relation-page
├── resources/stubs/
│   ├── RelationPage.stub                 ← PHP class template
│   └── relation-page-view.blade.stub     ← Blade view template
├── tests/
│   ├── TestCase.php
│   ├── Pest.php
│   └── Unit/
│       └── RelationPageTest.php
├── .github/
│   └── workflows/
│       ├── tests.yml
│       └── pint.yml
├── composer.json
├── phpunit.xml
├── pint.json
├── CHANGELOG.md
├── CONTRIBUTING.md
├── SECURITY.md
├── README.md
└── AGENTS.md (this file)
```

---

## Key Design Decisions

- **No Spatie Package Tools** — plain `ServiceProvider` keeps the dependency tree minimal.
- **Abstract base class, not an interface** — `RelationPage extends Livewire\Component` so Filament can mount it directly.
- **`InteractsWithSchemas` trait** — any method named `xxx(Schema $schema): Schema` is auto-discovered and becomes accessible as `$this->xxx` in Blade.
- **`#[Locked]` properties** — `$ownerRecord` and `$pageClass` are Livewire-locked to prevent client-side tampering.

---

## Artisan Command

```bash
php artisan make:filament-relation-page BuildingSummaryPage --resource=Buildings [--force]
```

Generates:
- `app/Filament/Resources/Buildings/RelationPages/BuildingSummaryPage.php`
- `resources/views/filament/resources/buildings/building-summary-page.blade.php`

Stub resolution order: user-published stubs at `stubs/filament-relation-pages/` take precedence over package stubs.

---

## Testing

```bash
vendor/bin/pest
vendor/bin/pest --coverage
vendor/bin/pint --test
vendor/bin/pint
```

Tests live in `tests/Unit/RelationPageTest.php`. They test static methods only — no Livewire mounting needed.

---

## Important Notes

- `RelationPage` must stay compatible with whatever Filament calls on `getRelations()` entries.
- When Filament releases a new major version, verify the three static method signatures in `HasRelationManagers`.
- The `RelationPagesPlugin` class exists mainly to satisfy the Filament plugin directory convention. The plugin works without ever calling `->plugin(RelationPagesPlugin::make())` in the panel.

---
> Source: [gheith3/filament-relation-pages](https://github.com/gheith3/filament-relation-pages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
