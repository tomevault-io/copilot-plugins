## wrestling

> Wrestling Legacy - wrestler management platform built with Laravel 12.

# Wrestling Legacy

Wrestling Legacy - wrestler management platform built with Laravel 12.

## Tech Stack

- **Backend**: Laravel 12, PHP 8.3, MySQL 8.0
- **Frontend**: Vite 7, Tailwind CSS 4, Alpine.js 3
- **Auth**: Laravel Fortify (registration, login, password reset, 2FA)
- **Storage**: MinIO (S3-compatible) for file uploads
- **Infrastructure**: Docker (nginx, app, mysql, minio, redis, mailpit)

## Architecture

- **Action Classes**: Business logic lives in `app/Actions/` (e.g. `CreateWrestler`, `CreateGear`, `DeleteGear`)
- **Models use UUIDs** as primary keys, soft deletes, and `created_by`/`updated_by` audit fields
- **Models have no Eloquent relationships** — controllers use direct queries and joins
- **Many-to-many**: Gear and Skills are linked to Wrestlers via pivot tables (`wrestler_gear`, `wrestler_skill`)
- **Admin pages**: Top-level CRUD for Gear (`/gear`) and Skills (`/skills`) via `AdminGearController` and `AdminSkillsController`, views in `resources/views/admin/`
- **Wrestler assignment**: Wrestler-scoped controllers (`GearController`, `SkillsController`) handle attaching/detaching existing gear/skills to wrestlers
- **Blade Components**: Layouts in `resources/views/components/layouts/`, reusable partials use `_form.blade.php` convention

## Domain Model

```
Wrestler (table: wrestlers)
Gear (table: gear) — many-to-many with Wrestler via wrestler_gear
Skill (table: skills) — many-to-many with Wrestler via wrestler_skill
```

## Commands

```bash
# Docker
docker compose up -d                          # Start all services
docker compose exec app php artisan migrate   # Run migrations
docker compose exec app php artisan tinker    # REPL

# Testing
docker compose exec app composer test         # Runs config:clear && phpunit
# Tests use SQLite in-memory (:memory:), array cache/session/mail, sync queue

# Development (inside container)
composer dev    # Concurrent: serve + queue:listen + pail + vite dev

# Linting
docker compose exec app ./vendor/bin/pint    # Laravel Pint (code style)

# Assets
npm run build   # Production build
npm run dev     # Vite dev server with HMR
```

## Workflow

- **"Start the project"** means: run `docker compose up -d` then run `docker compose exec app npm run dev` in the background for asset watching/HMR.

## Key Conventions

- All routes except welcome require auth middleware
- No permission system — all authenticated users can access all wrestlers
- Database transactions are not used for simple creates
- File uploads use `Storage::disk()` (default disk — `minio` locally, `s3` on Laravel Cloud)
- Dark mode supported via Tailwind `dark:` variants
- Form partials named `_form.blade.php` shared between create/edit views

## IMPORTANT: Sound Notification After finishing responding to my request or running a command, AND when asking a question, run this command to notify me by sound:
afplay /System/Library/Sounds/Funk.aiff

Do NOT use osascript or display notification commands.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rickwiltsie)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rickwiltsie)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
