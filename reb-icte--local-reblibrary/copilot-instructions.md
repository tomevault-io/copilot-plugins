## local-reblibrary

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Plugin Overview

**REB Library** (`local_reblibrary`) is a Moodle local plugin for managing the Rwanda Education Board digital library. It provides a complete resource management system with categorization, author management, and integration with Rwanda's education structure (levels, sublevels, classes, and A-Level sections).

**Location:** `moodle_app/public/local/reblibrary/`

**Version:** 1.3.1 (2025102403)

**Tech Stack:**
- Backend: PHP 8.2+ (Moodle external API / web services)
- Frontend: TypeScript + Preact + Tailwind CSS v4
- Build: Vite (compiles TypeScript to AMD modules)
- Database: MariaDB 11.4 (8 custom tables)

## Architecture Overview

### Database Schema

The plugin defines 8 custom tables in `db/install.xml`:

**Education Structure Tables:**
- `local_reblibrary_edu_levels` - Main levels (Pre-primary, Primary, Secondary)
- `local_reblibrary_edu_sublevels` - Sublevels (Nursery, Lower Primary, A Level, etc.)
- `local_reblibrary_classes` - Grade levels (N1, P4, S6, etc.) with unique class codes
- `local_reblibrary_sections` - A-Level subject combinations (PCM, HEG, etc.)

**Resource Management Tables:**
- `local_reblibrary_resources` - Main resource/book table (title, ISBN, description, file URLs)
- `local_reblibrary_authors` - Author information (first name, last name, bio)
- `local_reblibrary_categories` - Hierarchical categories (self-referencing for nesting)

**Junction Tables:**
- `local_reblibrary_res_assigns` - Links resources to classes/sections
- `local_reblibrary_res_categories` - Many-to-many resources ↔ categories

### Web Services API

All CRUD operations are exposed via Moodle external API (AJAX-enabled). Defined in `db/services.php`:

**Resources API** (`classes/external/resources.php`):
- `local_reblibrary_get_all_resources`
- `local_reblibrary_get_resource_by_id`
- `local_reblibrary_create_resource`
- `local_reblibrary_update_resource`
- `local_reblibrary_delete_resource`

**Education Structure API** (`classes/external/edu_structure.php`):
- Level, sublevel, class, and section CRUD operations

**Categories API** (`classes/external/categories.php`):
- Category CRUD operations

**Authors API** (`classes/external/authors.php`):
- Author CRUD operations

### Frontend Architecture

**TypeScript + Preact + Tailwind CSS:**
- Source: `amd/src/*.ts` and `amd/src/*.tsx`
- Components: `amd/src/components/` (admin UI, education structure, shared components)
- Build output: `amd/build/*.js` (AMD modules for Moodle's RequireJS)

**Key Frontend Modules:**
- `library-home.ts` - Public library home page
- `dashboard.ts` - Admin dashboard
- `resources.ts` - Resource management UI
- `categories.ts` - Category management UI
- `ed-structure.ts` - Education structure management UI

**Build System:**
- Vite configuration: `vite.config.ts`
- Auto-discovers all root-level `.ts`/`.tsx` files in `amd/src/`
- Excludes utility files: `app`, `types`, `store` (imported by other modules)
- Compiles to AMD format with bundled dependencies (Preact, Tailwind CSS)
- CSS is injected via JavaScript (no separate CSS files needed)

### Admin Interface

Admin pages located in `admin/`:
- `admin/index.php` - Main dashboard
- `admin/resources.php` - Resource management
- `admin/categories.php` - Category management
- `admin/ed_structure.php` - Education structure setup

Access requires `moodle/site:config` capability.

### Capabilities

Defined in `db/access.php`:
- `local/reblibrary:view` - View library (all users)
- `local/reblibrary:manageresources` - Manage resources (teachers, managers)
- `local/reblibrary:manageauthors` - Manage authors (teachers, managers)
- `local/reblibrary:managecategories` - Manage categories (teachers, managers)
- `local/reblibrary:manageedulevels` - Manage education levels (managers only)
- `local/reblibrary:manageedusublevels` - Manage sublevels (managers only)
- `local/reblibrary:manageclasses` - Manage classes (managers only)
- `local/reblibrary:managesections` - Manage A-Level sections (managers only)
- `local/reblibrary:manageassignments` - Assign resources to classes/sections

## Development Commands

### Frontend Build (TypeScript/Preact)

All commands run from the plugin directory:

```bash
# Install dependencies (first time only)
pnpm install

# Build once
pnpm run build

# Watch for changes (development mode)
pnpm run dev
```

**Critical:** After building frontend, always purge Moodle caches:

```bash
# From project root
docker compose exec php php /var/www/html/moodle_app/admin/cli/purge_caches.php
```

### Backend Development

```bash
# Purge caches (after PHP, template, or language string changes)
docker compose exec php php /var/www/html/moodle_app/admin/cli/purge_caches.php

# Run database upgrade (after changing db/install.xml or db/upgrade.php)
docker compose exec php php /var/www/html/moodle_app/admin/cli/upgrade.php --non-interactive

# Check plugin status
docker compose exec -T mariadb mariadb -u moodleuser -pmoodlepass moodle -e \
  "SELECT name, value FROM mdl_config_plugins WHERE plugin='local_reblibrary';"
```

### Database Inspection

```bash
# Access database shell
docker compose exec mariadb mariadb -u moodleuser -pmoodlepass moodle

# List plugin tables
docker compose exec -T mariadb mariadb -u moodleuser -pmoodlepass moodle -e \
  "SHOW TABLES LIKE 'mdl_local_reblibrary_%';"

# View resources
docker compose exec -T mariadb mariadb -u moodleuser -pmoodlepass moodle -e \
  "SELECT * FROM mdl_local_reblibrary_resources;"
```

## Common Development Tasks

### Adding a New TypeScript Module

1. Create file in `amd/src/` (e.g., `amd/src/mymodule.ts`)
2. Export an `init` function or other exports
3. Build: `pnpm run build`
4. Load in PHP: `$PAGE->requires->js_call_amd('local_reblibrary/mymodule', 'init');`
5. Purge caches: `docker compose exec php php /var/www/html/moodle_app/admin/cli/purge_caches.php`

**Example:**

```typescript
// amd/src/mymodule.ts
import { h, render } from 'preact';

export const init = () => {
  const container = document.getElementById('app');
  render(<div className="p-4 bg-blue-500">Hello Tailwind!</div>, container);
};
```

### Adding a New Web Service

1. Create method in appropriate external API class (e.g., `classes/external/resources.php`)
2. Define parameters with `{method}_parameters()` function
3. Define return value with `{method}_returns()` function
4. Implement the method with proper validation, context, and capability checks
5. Register in `db/services.php` with capability requirements
6. Increment `$plugin->version` in `version.php`
7. Run upgrade and purge caches

**Pattern:**

```php
// classes/external/myapi.php
namespace local_reblibrary\external;

class myapi extends external_api {
    public static function my_method_parameters() {
        return new external_function_parameters([...]);
    }

    public static function my_method($param) {
        $params = self::validate_parameters(self::my_method_parameters(), ['param' => $param]);
        $context = context_system::instance();
        self::validate_context($context);
        require_capability('local/reblibrary:view', $context);
        // Implementation...
    }

    public static function my_method_returns() {
        return new external_single_structure([...]);
    }
}
```

### Adding Database Tables

1. Edit `db/install.xml` using XMLDB syntax
2. For existing installations, create upgrade step in `db/upgrade.php`:
   ```php
   if ($oldversion < 2025102404) {
       // Create/modify tables using $dbman
       upgrade_plugin_savepoint(true, 2025102404, 'local', 'reblibrary');
   }
   ```
3. Increment `$plugin->version` in `version.php` to `2025102404`
4. Run upgrade: `docker compose exec php php /var/www/html/moodle_app/admin/cli/upgrade.php`

### Adding Preact Components

Components should be placed in `amd/src/components/`:

```tsx
// amd/src/components/MyComponent.tsx
import { h } from 'preact';

interface Props {
  title: string;
}

export default function MyComponent({ title }: Props) {
  return (
    <div className="p-4 rounded-lg bg-white shadow-md">
      <h2 className="text-xl font-bold">{title}</h2>
    </div>
  );
}
```

Import in entry point module (not excluded in `vite.config.ts`):

```typescript
// amd/src/mypage.ts
import { h, render } from 'preact';
import MyComponent from './components/MyComponent';

export const init = () => {
  render(<MyComponent title="Hello" />, document.getElementById('app'));
};
```

## Build System Details

### Vite Configuration

**Entry points:** Auto-discovered from `amd/src/*.ts` and `amd/src/*.tsx` (root level only)

**Excluded from entries:** `app`, `types`, `store` (defined in `vite.config.ts`)

**Output format:** AMD (for Moodle's RequireJS)

**Bundling:** Dependencies (Preact, Tailwind CSS) are bundled into each module

**Tailwind CSS:** v4 with modern `@import "tailwindcss"` syntax. Styles are injected via JavaScript.

**Source maps:** Generated for debugging

**Minification:** Enabled (esbuild)

### Adding New Entry Points

If you create a new root-level TypeScript file in `amd/src/`:
- It will automatically be discovered by Vite
- It will be compiled to `amd/build/{filename}.js`
- To exclude from entry points (if it's only imported by other modules), add to `excludeFromEntries` in `vite.config.ts`

## Integration with Parent Moodle Installation

This plugin is part of a Moodle 5.1 Docker environment:
- **Web root:** `moodle_app/public/` (Moodle 5.0+ structure)
- **Database:** MariaDB 11.4
- **URL:** http://localhost:8080
- **Admin credentials:** `admin` / `Admin123!`
- **Parent CLAUDE.md:** `/Users/bahatijustin/Dev/reb/elearning-app/CLAUDE.md`

## URLs and Pages

**Public:**
- Library Home: http://localhost:8080/local/reblibrary/
- Search: http://localhost:8080/local/reblibrary/search.php

**Admin (requires site config capability):**
- Dashboard: http://localhost:8080/local/reblibrary/admin/
- Resources: http://localhost:8080/local/reblibrary/admin/resources.php
- Categories: http://localhost:8080/local/reblibrary/admin/categories.php
- Education Structure: http://localhost:8080/local/reblibrary/admin/ed_structure.php

## Best Practices

1. **Always purge caches** after changes to PHP, templates, language strings, or compiled JavaScript
2. **Use Moodle APIs** - Never write raw SQL; use `$DB->get_records()`, `$DB->insert_record()`, etc.
3. **Validate all inputs** - Use `required_param()`, `optional_param()`, and external API validation
4. **Check capabilities** - Always verify user permissions with `require_capability()`
5. **Version increments** - Update `version.php` after database schema changes to trigger upgrades
6. **TypeScript types** - Define interfaces in `amd/src/types.ts` for type safety
7. **Frontend state** - Use Preact signals (`@preact/signals`) for reactive state management
8. **CSS classes** - Use Tailwind utility classes; avoid custom CSS files

## Useful Moodle APIs

- **Database:** `global $DB;` - DML API (`get_records()`, `insert_record()`, `update_record()`)
- **Output:** `global $OUTPUT;` - Rendering (`render_from_template()`, `header()`, `footer()`)
- **Page:** `global $PAGE;` - Page setup (`set_context()`, `set_url()`, `requires->js_call_amd()`)
- **User:** `global $USER;` - Current user object
- **Config:** `get_config('local_reblibrary', 'settingname')`
- **Capabilities:** `require_capability()`, `has_capability()`
- **Context:** `context_system::instance()`, `context_course::instance($courseid)`

## Troubleshooting

**Changes not showing up:**
- Purge caches: `docker compose exec php php /var/www/html/moodle_app/admin/cli/purge_caches.php`
- Rebuild frontend: `pnpm run build`

**Database changes not applied:**
- Increment version in `version.php`
- Run upgrade: `docker compose exec php php /var/www/html/moodle_app/admin/cli/upgrade.php`

**TypeScript module not found:**
- Ensure file is in `amd/src/` root (not subdirectory like `components/`)
- Check it's not excluded in `vite.config.ts`
- Rebuild: `pnpm run build`
- Purge caches

**Web service not working:**
- Check capability requirements in `db/services.php`
- Verify user has required capability
- Check browser console for AJAX errors
- Ensure external API class follows naming conventions (`{method}_parameters()`, `{method}_returns()`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/REB-ICTE) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
