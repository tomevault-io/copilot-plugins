## os-provisioning

> > These rules **override** any conflicting guidance from Laravel Boost or other rules.

# Project Overrides – Laravel Modules + Livewire (PostgreSQL)

> These rules **override** any conflicting guidance from Laravel Boost or other rules.  
> Place this file as `.cursor/rules/00-project-overrides.mdc` to load first.

## Scope
- Framework: Laravel (modules architecture), **Livewire-first UI**, PostgreSQL.
- Code Gen Targets: migrations, models, controllers, routes, lang usage.
- Naming & Tables: **singular table names** (`contact`, not `contacts`).

## Database & Migrations
- Migrations live **globally** in `database/migrations/` and **extend `BaseMigration`**.
- **Do NOT** re-add columns already provided by `BaseMigration` (e.g., `id`, `timestamps`, `softDeletes`). Only declare extra columns.
- PostgreSQL: use `$table->json(...)` for structured data (maps to JSONB on PG).
- Foreign keys: **index every FK** and default to `onDelete('restrict')` unless explicitly allowed otherwise.
- Use check constraints or string+validation for enums (no DB `ENUM`).
- Foreign key column names must follow `tablename_id` format (e.g., `contact_point_id` for FK to `contact_point` table). No shortcuts or abbreviations allowed.


## Modules & Models
- Follow ProvBase-style namespaces. Eloquent models go in `Modules/<Module>/Entities`.
- Only add relationships **if the related model exists**. Otherwise, omit (do not leave commented stubs).
- Provide a **static** `html_list(): array` helper on models used as relation sources; return `id => label` pairs for selects.
- Add appropriate `$fillable` and `$casts` (`json` → `array`).

## Controllers
- Controllers extend **`BaseController`** and should use existing generic view helpers.
- **Do not** use `pluck()` for relation options; use `$RelatedModel::html_list()`.
- Validate requests server-side; keep controller text strings out of code (use lang keys).

## Routes
- Use **module-local** route files: `Modules/<Module>/Routes/web.php` and `Routes/api.php`.
- Prefer `BaseRoute::resource('<singular>', '<Name>Controller')` for web routes;  
  use `Route::apiResource('<singular>', <Name>Controller::class)` for API.
- Route names should be namespaced: `<module>.<singular>` (web) and `<module>.<singular>.api` (api).
- IMPORTANT: Don't forget to add API routes!

## Internationalization (lang)
- **No hard-coded UI text** in controllers, views, or components.
- Use global lang files in both **`lang/en`** and **`lang/de`**; provide at least:
  - `dt_header.php` – titles/headers
  - `helper.php` – common messages (created/updated/deleted)
  - `view.php` – labels/placeholders/buttons
- Access via global keys (e.g., `__('dt_header.contact.index')`).

### Translation Pattern for Generic Index Views
- **Index table headers**: Model `index_header` uses simple field names → `trans('dt_header.'.$field)`
- **Form fields**: Controller uses nested keys → `trans('dt_header.ci_channel.key')`
- **Provide both**: Global field translations AND nested module translations

### Menu Translation Pattern
- **Menu items**: Use simple strings in config (e.g., `'Channels'`) → `BaseViewController::translate_view()` automatically looks for `view.Menu_Channels`
- **No `trans()` needed**: Global system handles menu translation via `view.Menu_{string}` pattern

## UI & Livewire
- **Livewire-first**; avoid Vue/JS unless explicitly requested.
- Livewire components should live alongside module views or under `App/Livewire` per your convention.
- Use pagination/search server-side. Keep Alpine usage minimal and conventional (no custom vendor scripts).

## Conventions Recap (short)
- Singular tables; every FK indexed; `onDelete('restrict')` by default.
- Global migrations extend `BaseMigration`; JSON → `$table->json(...)`.
- Models in `Modules/<Module>/Entities`; only real relationships.
- Controllers extend `BaseController`; use `html_list()` (no `pluck()`).
- Routes per module; BaseRoute/`apiResource` + namespaced route names.
- All UI text in lang files (`en` + `de`), at least `dt_header`, `helper`, `view`.

## Code Style
- **Use Laravel Pint**, but ONLY on files changed in your current branch (not on legacy code).
- Run Pint on changed files using: `./vendor/bin/pint --dirty`
- For formatting all files in a branch before PR: `./vendor/bin/pint $(git diff --name-only dev...HEAD -- '*.php')`
- This keeps changes focused and reviewable while avoiding touching legacy code.
- Follow existing code style conventions in the codebase for any code not formatted by Pint.
- Maintain consistent indentation and spacing with surrounding code.

---
> Source: [cablelabs/os-provisioning](https://github.com/cablelabs/os-provisioning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
