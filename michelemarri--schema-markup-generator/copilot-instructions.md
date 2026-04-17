## schema-markup-generator

> - **Usare sempre Tailwind CSS** - Tutti gli stili vanno scritti con classi Tailwind, non usare più il Metodo Design System (classi mds-*)

# Metodo Workspace Guidelines

## Styling con Tailwind CSS

- **Usare sempre Tailwind CSS** - Tutti gli stili vanno scritti con classi Tailwind, non usare più il Metodo Design System (classi mds-*)
- **Mai stili inline nel JavaScript** - Usare solo classi Tailwind
- **Mai SVG inline** - Usare sempre WordPress Dashicons (`<span class="dashicons dashicons-*">`)

---

# Schema Markup Generator - Project Rules

## Versioning

When making changes, update the version in these files (must be aligned):
1. `schema-markup-generator.php` - Header "Version:" (line 9) and constant `SMG_VERSION` (line 31)
2. `package.json` - "version" field (line 3)

Use semantic versioning (MAJOR.MINOR.PATCH):
- PATCH: bug fixes, minor corrections
- MINOR: new backwards-compatible features
- MAJOR: breaking changes

## Documentation

When modifying code, also update:
1. `README.md` - Changelog and features
2. `docs/DOCUMENTATION.md` - Detailed documentation
3. `docs/HOOKS.md` - If adding/modifying hooks and filters
4. `docs/EXTENDING.md` - If adding new extension possibilities

## Build Assets

After changes to CSS/JS in `assets/src/`, run:
```bash
npm run build
```

## Code Structure

- PHP 8.2+ with strict types
- Namespace: `flavor\SchemaMarkupGenerator`
- PSR-4 autoloading
- WordPress coding standards

## Schema Types

Schema types are in `src/Schema/Types/`. Each schema extends `AbstractSchema` and implements:
- `getType()` - schema.org type
- `build()` - Builds the schema data
- `getPropertyDefinitions()` - Defines mappable properties

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michelemarri) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
