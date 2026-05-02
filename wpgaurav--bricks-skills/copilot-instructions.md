## bricks-skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains **Bricks Builder skill files** - comprehensive development guides and reference materials for working with Bricks Builder (a WordPress visual website builder). The actual Bricks theme source code is located in a separate `bricks/` folder (not included in this repo as it's a paid product).

## Repository Contents

| File | Purpose |
|------|---------|
| `bricks-templates.md` | Creating and managing templates programmatically |
| `bricks-elements.md` | Building custom elements with controls and rendering |
| `bricks-hooks.md` | Complete filter and action reference |
| `bricks-layouts.md` | Ready-to-paste sections, JSON formats, Core Framework classes |
| `core-framework.css` | Utility-first CSS framework reference |

## Quick Reference

### Styling Priority

When styling Bricks elements, follow this order:

1. **Core Framework classes** (`core-framework.css`) - Use existing utility classes first
2. **Inline settings** - Use element `settings` for element-specific styling
3. **Custom CSS** - Only for complex patterns not covered by framework

### Two Layout Formats

**Sections (Copy/Paste)** - For pasting directly into builder:
```json
{
  "content": [...elements...],
  "source": "bricksCopiedElements",
  "sourceUrl": "https://example.com",
  "version": "2.0.1",
  "globalClasses": [...],
  "globalElements": []
}
```

**Templates (Site-Wide)** - For import via Templates system:
```json
{
  "id": 1195,
  "name": "header-new",
  "title": "Header New",
  "type": "header",
  "header": [...elements...],
  "global_classes": [...]
}
```

### Element ID Generation

Element IDs must be 6-character alphanumeric strings:
- Semantic: `sec001`, `hdr001`, `btn001`, `img001`
- Random: `jdhuyf`, `hxbbms`, `towhmc`

### Responsive Breakpoints

Append suffix to any setting for responsive behavior:

| Breakpoint | Suffix | Max Width |
|------------|--------|-----------|
| Desktop | (none) | - |
| Tablet Portrait | `:tablet_portrait` | 991px |
| Mobile Portrait | `:mobile_portrait` | 478px |

Example:
```json
{
  "_direction": "row",
  "_direction:tablet_portrait": "column"
}
```

## Bricks Theme Architecture (Reference)

The Bricks theme (when available in `bricks/`) follows this structure:

### Entry Points
- `functions.php` - Constants, global functions, loads `includes/init.php`
- `includes/init.php` - `Theme` singleton class that bootstraps all components

### Core Components (`includes/`)

| File | Class | Purpose |
|------|-------|---------|
| `elements.php` | `Elements` | Registers all builder elements |
| `elements/base.php` | `Element` | Abstract base class for elements |
| `frontend.php` | `Frontend` | Frontend rendering |
| `query.php` | `Query` | Custom query loop system |
| `api.php` | `Api` | REST API endpoints (`bricks/v1/`) |
| `templates.php` | `Templates` | Template management |
| `helpers.php` | `Helpers` | Utility functions |

### Namespace Convention

All classes use `Bricks` namespace. Integrations use nested namespaces:
```php
Bricks\Integrations\Dynamic_Data\Providers
Bricks\Integrations\Form\Init
```

### Key Functions

```php
bricks_is_builder()         // In builder (main or iframe)
bricks_is_builder_iframe()  // In builder iframe preview
bricks_is_frontend()        // Not in builder

bricks_render_dynamic_data( $content, $post_id, $context )
// Context: 'text', 'image', 'link'
```

### Common Filters

```php
// Modify element controls
add_filter( 'bricks/elements/{element_name}/controls', $callback );

// Modify element control groups
add_filter( 'bricks/elements/{element_name}/control_groups', $callback );

// Register custom dynamic data providers
add_filter( 'bricks/dynamic_data/register_providers', $callback );

// Modify rendered element output
add_filter( 'bricks/frontend/render_element', $callback, 10, 2 );
```

## WP-CLI

```bash
wp bricks regenerate_assets  # Regenerate CSS files
```

## Documentation

- Official: https://academy.bricksbuilder.io/
- Developer Docs: https://academy.bricksbuilder.io/collection/developer/

---
> Source: [wpgaurav/bricks-skills](https://github.com/wpgaurav/bricks-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
