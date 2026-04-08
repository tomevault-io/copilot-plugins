## contenta-content-management-mobile

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Contenta is a Laravel 12 + Vue 3 (Inertia.js) application for content-centric platforms (CMS/knowledge base/publishing) using domain-driven architecture with opinionated separation under `app/Domains/*`.

**IMPORTANT:** The project root directory is `/home/jkavuncuoglu/Projects/contenta/contenta`. All commands and paths should be executed from this directory.

### Path Aliases

- **`@/`** = Project root directory (`/home/jkavuncuoglu/Projects/contenta/contenta`)
- **`@contenta/`** = Project root directory (alternate alias)

When referencing files or creating documentation:
- Use `@/FILENAME.md` for root-level documentation files
- **CRITICAL:** All `.md` documentation files created by Claude **MUST** be placed in the project root (`@/`)
- Never create `.md` files inside `src/`, `app/`, or other subdirectories unless explicitly requested
- Examples:
  - ✅ `@/PHASE2_CHECKPOINT.md`
  - ✅ `@/PROGRESS_REPORT.md`
  - ✅ `@/TASK_RENAME_PAGEBUILDER_TO_PAGES.md`
  - ❌ `src/PHASE2_CHECKPOINT.md` (wrong - never do this)
  - ❌ `app/PROGRESS.md` (wrong - never do this)

## Laravel Sail Usage

**CRITICAL**: This project uses Laravel Sail (Docker) for all development operations.

### Sail Command Rules

**ALWAYS use Sail for:**
- ✅ All `artisan` commands: `./vendor/bin/sail artisan <command>`
- ✅ All `composer` commands: `./vendor/bin/sail composer <command>`
- ✅ All `php` commands: `./vendor/bin/sail php <command>`
- ✅ Running tests: `./vendor/bin/sail artisan test` or `./vendor/bin/sail pest`
- ✅ Database operations: Sail manages MySQL container
- ✅ Queue workers: Sail manages Redis container

**NEVER use direct commands without Sail** unless explicitly instructed:
- ❌ `php artisan ...` → Use `./vendor/bin/sail artisan ...`
- ❌ `composer ...` → Use `./vendor/bin/sail composer ...`
- ❌ `./vendor/bin/pest` → Use `./vendor/bin/sail pest`

### Starting Sail

Before running any commands, ensure Sail is running:
```bash
./vendor/bin/sail up -d
```

Check if Sail is running:
```bash
./vendor/bin/sail ps
```

Stop Sail:
```bash
./vendor/bin/sail down
```

## Commands

### Development
```bash
# Start Sail services first
./vendor/bin/sail up -d

# Start development server (PHP server + queue + logs + Vite concurrently)
./vendor/bin/sail composer dev

# Start development with SSR
./vendor/bin/sail composer dev:ssr

# Frontend only (can run without Sail)
npm run dev
```

### Building
```bash
# Build frontend assets (can run without Sail)
npm run build

# Build with SSR support (can run without Sail)
npm run build:ssr
```

### Testing
```bash
# ALWAYS use Sail for PHP tests
./vendor/bin/sail artisan test

# Or using Pest directly via Sail
./vendor/bin/sail pest

# Run specific test file
./vendor/bin/sail pest tests/Unit/ShortcodeParser/TokenizerTest.php

# Run E2E tests (can run without Sail if Playwright is installed)
npm run test:e2e
```

### Code Quality
```bash
# PHP formatting (Laravel Pint) - use Sail
./vendor/bin/sail pint

# Frontend linting and formatting (can run without Sail)
npm run lint
npm run format
npm run format:check
```

### Database
```bash
# ALWAYS use Sail for database operations
./vendor/bin/sail artisan migrate
./vendor/bin/sail artisan migrate:fresh
./vendor/bin/sail artisan migrate:rollback

# Database seeding
./vendor/bin/sail artisan db:seed

# Clear caches and optimize
./vendor/bin/sail artisan optimize:clear
./vendor/bin/sail composer dump-autoload
```

## Architecture

### Domain-Driven Structure
The application uses domain separation under `app/Domains/`:

- **ContentManagement/**: Posts, Pages, Categories, Tags, Comments
- **Media/**: File uploads and media management
- **Security/**: API tokens, two-factor auth, roles/permissions
- **Settings/**: User profile, site configuration
- **PageBuilder/**: Page builder (being replaced with markdown editor)

Each domain encapsulates:
- Models/Aggregates
- Http/Controllers (UI + Admin endpoints)
- Services (application layer)
- Policies, Jobs, Events
- **Permissions** (database seeder for domain-specific permissions)

### Permissions & Authorization

**CRITICAL:** Every new domain MUST implement permissions using Spatie Laravel Permission.

#### Required Steps for New Domains:

1. **Create Permissions Seeder:**
   ```bash
   ./vendor/bin/sail artisan make:seeder {DomainName}PermissionsSeeder
   ```

2. **Define Domain Permissions:**
   Follow the pattern: `{action} {domain resource}`

   Examples:
   - `view social accounts`
   - `create social posts`
   - `edit social accounts`
   - `delete social posts`
   - `publish social posts`

3. **Create Domain-Specific Role:**
   Each domain should have a manager role (e.g., "Social Media Manager")

4. **Seeder Template:**
   ```php
   public function run(): void
   {
       app()[\Spatie\Permission\PermissionRegistrar::class]->forgetCachedPermissions();

       $permissions = [
           'view {domain} {resource}',
           'create {domain} {resource}',
           'edit {domain} {resource}',
           'delete {domain} {resource}',
           // Add domain-specific permissions
       ];

       foreach ($permissions as $permission) {
           Permission::firstOrCreate(['name' => $permission, 'guard_name' => 'web']);
       }

       $role = Role::firstOrCreate(['name' => '{Domain} Manager', 'guard_name' => 'web']);
       $role->syncPermissions($permissions);

       // Grant to Admin role
       $admin = Role::where('name', 'Admin')->where('guard_name', 'web')->first();
       if ($admin) {
           $admin->givePermissionTo($permissions);
       }
   }
   ```

5. **Protect Controllers:**
   Add permission middleware in controller constructor:
   ```php
   public function __construct()
   {
       $this->middleware('permission:view {domain} {resource}')->only(['index', 'show']);
       $this->middleware('permission:create {domain} {resource}')->only(['create', 'store']);
       $this->middleware('permission:edit {domain} {resource}')->only(['edit', 'update']);
       $this->middleware('permission:delete {domain} {resource}')->only(['destroy']);
   }
   ```

6. **Run the Seeder:**
   ```bash
   ./vendor/bin/sail artisan db:seed --class={DomainName}PermissionsSeeder
   ```

#### Permission Naming Convention:
- Format: `{verb} {domain-name} {resource-name}`
- Use lowercase with spaces
- Be specific and descriptive
- Group by resource when possible

#### Example Domain Permissions:

**Social Media Domain:**
- Account Management: `view/connect/edit/disconnect social accounts`, `refresh social tokens`
- Post Management: `view/create/edit/delete/publish social posts`
- Analytics: `view/sync social analytics`

**Content Management Domain:**
- Posts: `view/create/edit/delete/publish posts`
- Pages: `view/create/edit/delete/publish pages`
- Comments: `view/moderate/delete comments`

### Service Bindings
Key services registered in `App\Providers\AppServiceProvider`:
- `TwoFactorAuthenticationServiceInterface` → `TwoFactorAuthenticationService`
- `PagesServiceContract` → `PagesService`
- `MediaServiceContract` → `MediaService`
- `CommentsServiceContract` → `CommentsService`
- `ShortcodeParserServiceContract` → `ShortcodeParserService`
- `MarkdownRenderServiceContract` → `MarkdownRenderService`

### Frontend Structure
```
resources/js/
├── pages/          # Inertia page components
├── layouts/        # Application layouts
├── components/     # Reusable Vue components
├── stores/         # Application state
├── types/          # TypeScript definitions
└── lib/            # Utilities and helpers
```

## Key Technologies

### Backend
- Laravel 12 (PHP 8.4)
- Laravel Fortify (authentication)
- Laravel Sanctum (API tokens)
- Spatie packages (permissions, media, activity log)
- Horizon (queue management)
- **Markdown Parser**: Custom shortcode parser for content rendering

### Frontend
- Vue 3 + TypeScript
- Inertia.js (SPA-like experience)
- Tailwind CSS v4
- Reka UI components
- Vite 7 (build tool)

### Testing
- Pest (PHP testing)
- Playwright (E2E testing)

## Route Structure

- `/` - Public pages
- `/admin/dashboard` - Main authenticated dashboard (default after login)
- `/admin/*` - Admin content management
  - `/admin/posts` - Posts management
  - `/admin/page-builder` - PageBuilder (drag & drop page editor)
  - `/admin/categories` - Categories management
  - `/admin/tags` - Tags management
  - `/admin/comments` - Comments moderation
  - `/admin/media` - Media library
- `/user/settings/*` - User profile and settings management
  - `/user/settings/profile` - Profile management
  - `/user/settings/password` - Password management
  - `/user/settings/security/two-factor` - 2FA management
  - `/user/settings/api-tokens` - API token management

## Development Workflow

### Running Tests
Always run tests via Sail before committing changes:
```bash
./vendor/bin/sail artisan test
npm run test:e2e  # if E2E tests exist
```

### Code Style
The project enforces code style through:
- Laravel Pint for PHP (configured in CI): `./vendor/bin/sail pint`
- ESLint + Prettier for TypeScript/Vue: `npm run lint && npm run format`
- GitHub Actions workflows for automatic checks

### Making Changes
1. Ensure Sail is running: `./vendor/bin/sail up -d`
2. Follow existing domain patterns when adding features
3. Use service interfaces for domain logic abstraction
4. Write tests for new functionality
5. Ensure code passes linting before committing

## Important Notes

### Environment Setup
- Uses MySQL via Sail Docker container (do NOT use SQLite in development)
- Queue connection: database (Redis available via Sail)
- Session driver: database for persistence
- Redis available via Sail for caching/queues

### Security Features
- API token management with granular abilities
- Two-factor authentication (TOTP + recovery codes)
- Role-based permissions via Spatie
- Activity logging for audit trails

### Testing Status
- **Unit Tests**: All passing (370 tests, 1588 assertions)
- **Domain Unit Tests**: All passing
- **Feature Tests**: Some tests pending full implementation of:
  - Two-factor authentication flows (base implementation exists, full flow testing in progress)
  - Password reset email notifications (requires mail configuration)
  - User registration flows
  - Some admin controller endpoints

### CI/CD
GitHub Actions workflows configured for:
- PHP code quality (Laravel Pint)
- JavaScript/TypeScript code quality (ESLint + Prettier)
- Automated testing with HTML coverage reports

### Media Handling
Uses Spatie Media Library with collections:
- `featured_images`
- `gallery`
- `attachments`

Run `./vendor/bin/sail artisan storage:link` for local file serving.

## Markdown Editor (New Feature)

### Shortcode Syntax
The project now uses a custom markdown parser with shortcode syntax for page/post content.

See `SHORTCUT_SYNTAX_SPEC.md` for full syntax documentation.

Example:
```markdown
[#hero title="Welcome" subtitle="Get Started"]{
Learn more about our platform
}[/#hero]

[#features title="Our Features" columns="3"]{
[#feature-item title="Fast"]{Lightning speed}[/#feature-item]
[#feature-item title="Secure"]{Bank-level security}[/#feature-item]
}[/#features]
```

### Migration Command
Migrate legacy page builder pages to markdown:
```bash
./vendor/bin/sail artisan page:migrate-md
```

See `MIGRATION_EXAMPLE.md` for examples.

## Service Container Usage

When adding new services:
1. Create interface in domain's `Services/` directory
2. Implement concrete class
3. Register binding in `AppServiceProvider`
4. Type-hint interface in controllers

## Database Conventions

- Uses Laravel's standard migration structure
- Activity logging tracks all model changes
- Soft deletes where appropriate
- Uses UUIDs for API tokens via Sanctum

## Troubleshooting

### Database Connection Issues
If you see "Connection refused" or MySQL errors:
```bash
# Check if Sail is running
./vendor/bin/sail ps

# Start Sail if not running
./vendor/bin/sail up -d

# Check MySQL container logs
./vendor/bin/sail logs mysql
```

### Permission Issues
If you encounter permission errors:
```bash
# Fix permissions (Sail runs as sail user, UID 1000)
sudo chown -R $USER:$USER .
```

### Cache Issues
```bash
./vendor/bin/sail artisan optimize:clear
./vendor/bin/sail composer dump-autoload
```

# Contanta Project Rules

## Coding and Generation Rules

1. **Architecture**
    - Use **Domain-Driven Design (DDD)** strictly.
    - Each backend feature must belong to a domain in `app/Domains/{DomainName}`.
    - Cross-domain communication should occur only via **service contracts** or **facades**.
    - Keep controllers thin — delegate logic to Services.

2. **Laravel Backend Rules**
    - Use **Form Requests** for input validation.
    - Use **Service Contracts and Providers** for dependency binding.
    - **Unit tests must reside inside the domain** under
        - `app/Domains/{DomainName}/tests/UnitTests`
        - `app/Domains/{DomainName}/tests/FeatureTests`
    -
        - Examples:
            - `MenuBuilderServiceTest.php`
            - `MenuModelTest.php`
            - `MenuBuilderControllerTest.php`
            - `MenuBuilderAdminControllerTest.php`
            - `MenuBuilderServiceContractTest.php`
            - `MenuBuilderFacadeTest.php`
            - `MenuBuilderServiceProviderTest.php`
            - `MenuBuilderRoutesTest.php`
            - `MenuBuilderFormRequestTest.php`
            - `MenuBuilderInputTest.php`
            - `MenuBuilderConstantTest.php`
            - `MenuBuilderSubdomainTests.php`
            - etc.
    - Controllers must return **data or Inertia-compatible responses**, but the frontend itself is managed separately.

3. **Testing Requirements**
    - **All backend tests must be encapsulated within the domain** to make domains plug-and-play.
    - Backend: **PHPUnit / Pest** for services, models, and controllers.
    - Each generated backend file must include a corresponding test stub.
    - Domain test folder structure:
      ```
      `app/Domains/{DomainName}/tests/UnitTests`
      `app/Domains/{DomainName}/tests/FeatureTests`
      ```

4. **Inertia + Routing**
    - Routes belong in `{Domain}/Http/routes.php`.
    - Register routes via ServiceProvider.
    - The domain should focus on backend logic; frontend components are handled separately.

5. **Service Layer**
    - Service classes must implement `{Domain}ServiceContract.php`.
    - Bind interfaces to implementations in `{Domain}ServiceProvider.php`.
    - Business logic should always reside in the Service, not Controller.

6. **AI Generation Behavior**
    - When generating code or scaffolds:
        - Follow the **DDD backend structure** exactly.
        - Include controllers, models, services, contracts, facades, providers, and **backend unit tests inside the domain**.
        - Do **not** generate Vue components or frontend code inside the domain.

7. **Code Quality**
    - Follow **PSR-12** and **Laravel Style Guide**.
    - Use docblocks for all public methods.
    - Use clear and descriptive naming for all classes and variables.

8. **Domain Encapsulation Goal**
    - Each backend domain must be **fully self-contained**, including:
        - Models
        - Controllers
        - Services & Contracts
        - Routes
        - Unit tests (PHP)
    - Domains should be **plug-and-play**, requiring minimal external changes if moved to another project.

# Contanta Project Context

## Overview
Contanta is an AI-powered content management and automation platform built with **Laravel 12**, **Vue 3**, and **TailwindCSS**, using **Domain-Driven Design (DDD)** principles.  
The system is modular, scalable, and designed around distinct *domains* and *subdomains*, each encapsulating its own logic, routes, models, and services.

## Project Architecture

### Folder Structure
app/
└── Domains/
├── {DomainName}/
│ ├── Http/
│ │ ├── Controllers/
│ │ │ ├── {Domain}Controller.php
│ │ │ └── {Domain}AdminController.php
│ │ ├── Requests/
│ │ │ └── {FormRequestName}.php
│ │ └── Inputs/
│ │ └── {InputName}.php
│ ├── Constants/
│ │ └── {ConstantFile}.php
│ ├── Models/
│ │ └── {ModelName}.php
│ ├── Services/
│ │ ├── {Domain}Service.php
│ │ ├── {Domain}ServiceContract.php
│ │ └── ...
│ ├── routes.php # Inertia routes
│ ├── {Domain}.php # Facade
│ ├── {Domain}ServiceProvider.php
│ └── {Subdomain}/
│ └── ... (same structure)


### Backend Technologies
- **Laravel 12**
- **Inertia.js (Vue 3 + Laravel Bridge)**
- **PHPUnit / Pest** for unit and feature tests
- **Laravel Service Container** and **Contracts** for dependency inversion

### Frontend Technologies
- **Vue 3 (Composition API)**
- **Inertia.js** for SPA routing
- **TailwindCSS** for design
- **Vitest + Vue Test Utils** for unit tests

### AI & Automation
- Contanta integrates AI-assisted scaffolding commands (e.g., `ai:make domain MenuBuilder`) that auto-generate DDD-compliant code and associated unit tests.

---

## Design & Development Principles

1. **Follow Domain-Driven Design (DDD):**
    - Each domain encapsulates its own logic, models, routes, and services.
    - Cross-domain communication should occur only via service contracts or facades.
    - Keep controllers thin — delegate logic to Services.

2. **Laravel + Inertia Best Practices:**
    - Keep route definitions inside each domain’s `routes.php`.
    - Use Form Requests for validation.
    - Use Inertia responses for all view rendering, passing props from controller to Vue components.

3. **Vue 3 Best Practices:**
    - Use Composition API (`<script setup>`).
    - Keep state in Pinia stores or composables, not in single components.
    - Components should be domain-scoped and reusable.
    - Apply Tailwind utility classes for consistency.

4. **Testing:**
    - Write **unit tests** for Laravel models, services, and controllers.
    - Write **unit and component tests** for Vue components using Vitest.
    - Each generated component or service must include a test stub.

---

## Example Domain: MenuBuilder
A typical `MenuBuilder` domain would contain:
- `MenuBuilderController.php` (Inertia routes)
- `MenuBuilderService.php` (business logic)
- `MenuBuilderServiceContract.php` (interface)
- `Menu.php` (Eloquent model)
- Vue components under `resources/js/Pages/MenuBuilder/`
- Tests under `tests/Unit/Domains/MenuBuilder/`

---

## AI Expectations
Claude should:
- Always scaffold new features respecting DDD domain structure.
- Automatically generate Laravel and Vue test stubs.
- Use Laravel-style naming conventions (PascalCase for classes, snake_case for database).
- Follow PSR-12 and Tailwind styling conventions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jkavuncuoglu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jkavuncuoglu)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
