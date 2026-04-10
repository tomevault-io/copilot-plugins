## gotime-site

> <laravel-boost-guidelines>

<laravel-boost-guidelines>
=== .ai/livewire-rules rules ===

# Livewire Rules

## Component Generation

- Always use `php artisan livewire:make` to create components
- Use the namespace when specified in the prompt (e.g., "admin product index" →
  `admin::resource.index`)
- Namespaces are configured in `config/livewire.php` under
  `component_namespaces`
- Livewire handles all file structure and code generation automatically

```bash

# With namespace

php artisan livewire:make admin::resource.index

# No namespace

php artisan livewire:make resource.index
```

```bash

# example of multiple components for a resource

pa livewire:make admin::resource.form
pa livewire:make admin::resource.index
pa livewire:form WidgetFormObject
```

### Blade References

```html +code-blade
<livewire:admin::resource.index />
<livewire:resource.index />
```

## Naming Conventions

Component names follow the pattern: `{namespace}::{resource}.{type}`

**Examples:**

- `product.index` - Public product listing
- `admin::course.index` - Admin course listing  
- `admin::course.manager` - Admin course management hub
- `quiz.editor` - Quiz editing workspace

=== .ai/naykel-laravel rules ===

# Naykel Laravel & PHP Guidelines (Reference)

## Core Laravel Principle

**Follow Laravel conventions first.** If Laravel has a documented way to do
something, use it. Only deviate when you have a clear justification.

## PHP Standards

- Use short nullable notation: `?string` not `string|null`
- Always specify `void` return types when methods return nothing
- Use typed properties over docblocks
- Prefer early returns over nested if/else
- Use constructor property promotion when all properties can be promoted
- Avoid `else` statements when possible
- Use string interpolation over concatenation

## Migrations

- Do not write down methods in migrations, only up methods

### Column Type Guidelines

- `price`, `stock`, `quantity`, `unit_price`, `total`, `amount` → type depends
  on usage:
  - **Product prices / fixed values** → `bigint unsigned`
  - **Ledger / transactions** → signed (`bigint` or `integer`)

### Price and Amount Fields

- `price` and `amount` fields should be nullable WITHOUT a default value
  - Reasoning: A record without a price/amount shouldn't exist in the database
  - Use `NULL` to indicate draft/incomplete records if needed
  - Avoid default `0` as it creates ambiguity between "free" and "not set"
- `unit_price` is signed (`bigint` or `integer`) to allow negative values for
  credits/adjustments on order line items

=== .ai/working-with-nathan rules ===

Here’s a tightened, AI-friendly version that’s very direct and enforces the “stop vs do” rules clearly:

# Working with Nathan

## When to Act vs When to Ask

**Do it without asking when:**

- The task is explicitly requested.
- The approach is documented in skills or guidelines.
- You’re following a known pattern from the codebase.

**Stop and ask before proceeding when:**

- Modifying config files not mentioned in the request.
- Inventing command flags, options, or behavior.
- Multiple valid approaches exist and I haven't specified which.
- You're unsure if your interpretation matches my intent.
- Something feels off or conflicts with existing patterns.

**Red flags = STOP immediately:**

- “I’ll also…”
- “While I’m at it…”
- “To make this work, I need to first…”
- “This requires…”
- Using commands or options not present in existing code or docs.

*Rule of thumb:* If in doubt, pause and check.

## Code Philosophy

- **Explicit over clever** — Show what’s happening rather than hiding it.
- **Start small, expand later** — Validate the approach before building out.
- **Config = WHAT, Code = HOW** — Config defines structure; code defines implementation.

## Frustrations to Avoid

- Overcomplicating simple things.
- Suggesting solutions I've already rejected.
- Hiding behavior or trying to "save me work."
- Making me second-guess solid decisions.
- Ignoring instructions or being indirect.

## Skill Philosophy

- **Self-contained skills** — Each skill must have all information needed to execute its task.
- **No external assumptions** — Skills should never rely on context outside themselves.

## Proactive Verification

Always compare your current understanding or implementation with what I have explicitly asked:

- **Highlight differences clearly** - If something doesn't match, show what differs and why
- **Point out missing steps** - If you notice a relevant gap, suggest adding it
- **Record discoveries** - Integrate new findings into context automatically
- **Present in list format** - For clarity, use lists for suggestions and findings

**Example:**

Differences found:

1. You asked for X, but the code does Y
2. Missing: Z step in the workflow

Suggestion: Add step Z here...

**Do not act unless instructed**, except to highlight differences, missing info, or updates.

=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to ensure the best experience when building Laravel applications.

## Foundational Context

This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.3.28
- laravel/framework (LARAVEL) - v12
- laravel/prompts (PROMPTS) - v0
- livewire/livewire (LIVEWIRE) - v4
- laravel/boost (BOOST) - v2
- laravel/mcp (MCP) - v0
- laravel/pail (PAIL) - v1
- laravel/pint (PINT) - v1
- laravel/sail (SAIL) - v1
- pestphp/pest (PEST) - v4
- phpunit/phpunit (PHPUNIT) - v12

## Skills Activation

This project has domain-specific skills available. You MUST activate the relevant skill whenever you work in that domain—don't wait until you're stuck.

- `livewire-development` — Develops reactive Livewire 4 components. Activates when creating, updating, or modifying Livewire components; working with wire:model, wire:click, wire:loading, or any wire: directives; adding real-time updates, loading states, or reactivity; debugging component behavior; writing Livewire tests; or when the user mentions Livewire, component, counter, or reactive UI.
- `pest-testing` — Tests applications using the Pest 4 PHP framework. Activates when writing tests, creating unit or feature tests, adding assertions, testing Livewire components, browser testing, debugging test failures, working with datasets or mocking; or when the user mentions test, spec, TDD, expects, assertion, coverage, or needs to verify functionality works.
- `database-design` — Creates database design documents that serve as source of truth for tables, columns, and relationships. Uses these documents to generate migrations, models, and factories. Activates when creating schemas, tables, models, migrations, or factories.
- `livewire-editor-component` — Build Livewire Editor components for editing complex content with multiple related parts. Coordinates several forms that work together with a single save action. Activates when creating focused editing workspaces, multi-part content editors, or cohesive data entry tasks.
- `livewire-file-uploads` — Adds file upload capability to Livewire form components. Handles temporary storage, validation, and persisting uploads.
- `livewire-form-component` — Build Livewire Form components for creating and editing records. Displays structured input fields, handles user input and submission. Activates when creating data entry forms, edit interfaces, or input screens.
- `livewire-form-object` — Build Form Objects that centralize form data, validation, and state management. Handles field definitions, validation rules, error handling, and data persistence separate from UI components. Activates when creating structured form data containers, implementing validation logic, or managing form state and submission.
- `livewire-index-component` — Build Livewire Index components that display and manage resource collections. Handles listing, searching, filtering, sorting, pagination, and actions on records. Activates when creating resource listing pages, data tables, or browse interfaces.
- `livewire-resource-config` — Define resource configuration in config/resources.php for Livewire components. Controls index and form behaviour including columns, buttons, search, sorting, and basic routing. Use this before generating or modifying Livewire components.
- `markdown-formatting` — Ensures markdown files follow project conventions including Torchlight code tags and markdown-lint rules. Activates when creating or editing markdown files.

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

- Laravel Boost is an MCP server that comes with powerful tools designed specifically for this application. Use them.

## Artisan

- Use the `list-artisan-commands` tool when you need to call an Artisan command to double-check the available parameters.

## URLs

- Whenever you share a project URL with the user, you should use the `get-absolute-url` tool to ensure you're using the correct scheme, domain/IP, and port.

## Tinker / Debugging

- You should use the `tinker` tool when you need to execute PHP to debug code or query Eloquent models directly.
- Use the `database-query` tool when you only need to read from the database.
- Use the `database-schema` tool to inspect table structure before writing migrations or models.

## Reading Browser Logs With the `browser-logs` Tool

- You can read browser logs, errors, and exceptions using the `browser-logs` tool from Boost.
- Only recent browser logs will be useful - ignore old logs.

## Searching Documentation (Critically Important)

- Boost comes with a powerful `search-docs` tool you should use before trying other approaches when working with Laravel or Laravel ecosystem packages. This tool automatically passes a list of installed packages and their versions to the remote Boost API, so it returns only version-specific documentation for the user's circumstance. You should pass an array of packages to filter on if you know you need docs for particular packages.
- Search the documentation before making code changes to ensure we are taking the correct approach.
- Use multiple, broad, simple, topic-based queries at once. For example: `['rate limiting', 'routing rate limiting', 'routing']`. The most relevant results will be returned first.
- Do not add package names to queries; package information is already shared. For example, use `test resource table`, not `filament 4 test resource table`.

### Available Search Syntax

1. Simple Word Searches with auto-stemming - query=authentication - finds 'authenticate' and 'auth'.
2. Multiple Words (AND Logic) - query=rate limit - finds knowledge containing both "rate" AND "limit".
3. Quoted Phrases (Exact Position) - query="infinite scroll" - words must be adjacent and in that order.
4. Mixed Queries - query=middleware "rate limit" - "middleware" AND exact phrase "rate limit".
5. Multiple Queries - queries=["authentication", "middleware"] - ANY of these terms.

=== php rules ===

# PHP

- Always use curly braces for control structures, even for single-line bodies.

## Constructors

- Use PHP 8 constructor property promotion in `__construct()`.
    - `public function __construct(public GitHub $github) { }`
- Do not allow empty `__construct()` methods with zero parameters unless the constructor is private.

## Type Declarations

- Always use explicit return type declarations for methods and functions.
- Use appropriate PHP type hints for method parameters.

<!-- Explicit Return Types and Method Params -->
```php
protected function isAccessible(User $user, ?string $path = null): bool
{
    ...
}
```

## Enums

- Typically, keys in an Enum should be TitleCase. For example: `FavoritePerson`, `BestLake`, `Monthly`.

## Comments

- Prefer PHPDoc blocks over inline comments. Never use comments within the code itself unless the logic is exceptionally complex.

## PHPDoc Blocks

- Add useful array shape type definitions when appropriate.

=== laravel/core rules ===

# Do Things the Laravel Way

- Use `php artisan make:` commands to create new files (i.e. migrations, controllers, models, etc.). You can list available Artisan commands using the `list-artisan-commands` tool.
- If you're creating a generic PHP class, use `php artisan make:class`.
- Pass `--no-interaction` to all Artisan commands to ensure they work without user input. You should also pass the correct `--options` to ensure correct behavior.

## Database

- Always use proper Eloquent relationship methods with return type hints. Prefer relationship methods over raw queries or manual joins.
- Use Eloquent models and relationships before suggesting raw database queries.
- Avoid `DB::`; prefer `Model::query()`. Generate code that leverages Laravel's ORM capabilities rather than bypassing them.
- Generate code that prevents N+1 query problems by using eager loading.
- Use Laravel's query builder for very complex database operations.

### Model Creation

- When creating new models, create useful factories and seeders for them too. Ask the user if they need any other things, using `list-artisan-commands` to check the available options to `php artisan make:model`.

### APIs & Eloquent Resources

- For APIs, default to using Eloquent API Resources and API versioning unless existing API routes do not, then you should follow existing application convention.

## Controllers & Validation

- Always create Form Request classes for validation rather than inline validation in controllers. Include both validation rules and custom error messages.
- Check sibling Form Requests to see if the application uses array or string based validation rules.

## Authentication & Authorization

- Use Laravel's built-in authentication and authorization features (gates, policies, Sanctum, etc.).

## URL Generation

- When generating links to other pages, prefer named routes and the `route()` function.

## Queues

- Use queued jobs for time-consuming operations with the `ShouldQueue` interface.

## Configuration

- Use environment variables only in configuration files - never use the `env()` function directly outside of config files. Always use `config('app.name')`, not `env('APP_NAME')`.

## Testing

- When creating models for tests, use the factories for the models. Check if the factory has custom states that can be used before manually setting up the model.
- Faker: Use methods such as `$this->faker->word()` or `fake()->randomDigit()`. Follow existing conventions whether to use `$this->faker` or `fake()`.
- When creating tests, make use of `php artisan make:test [options] {name}` to create a feature test, and pass `--unit` to create a unit test. Most tests should be feature tests.

## Vite Error

- If you receive an "Illuminate\Foundation\ViteException: Unable to locate file in Vite manifest" error, you can run `npm run build` or ask the user to run `npm run dev` or `composer run dev`.

=== laravel/v12 rules ===

# Laravel 12

- CRITICAL: ALWAYS use `search-docs` tool for version-specific Laravel documentation and updated code examples.
- Since Laravel 11, Laravel has a new streamlined file structure which this project uses.

## Laravel 12 Structure

- In Laravel 12, middleware are no longer registered in `app/Http/Kernel.php`.
- Middleware are configured declaratively in `bootstrap/app.php` using `Application::configure()->withMiddleware()`.
- `bootstrap/app.php` is the file to register middleware, exceptions, and routing files.
- `bootstrap/providers.php` contains application specific service providers.
- The `app\Console\Kernel.php` file no longer exists; use `bootstrap/app.php` or `routes/console.php` for console configuration.
- Console commands in `app/Console/Commands/` are automatically available and do not require manual registration.

## Database

- When modifying a column, the migration must include all of the attributes that were previously defined on the column. Otherwise, they will be dropped and lost.
- Laravel 12 allows limiting eagerly loaded records natively, without external packages: `$query->latest()->limit(10);`.

### Models

- Casts can and likely should be set in a `casts()` method on a model rather than the `$casts` property. Follow existing conventions from other models.

=== livewire/core rules ===

# Livewire

- Livewire allows you to build dynamic, reactive interfaces using only PHP — no JavaScript required.
- Instead of writing frontend code in JavaScript frameworks, you use Alpine.js to build the UI when client-side interactions are required.
- State lives on the server; the UI reflects it. Validate and authorize in actions (they're like HTTP requests).
- IMPORTANT: Activate `livewire-development` every time you're working with Livewire-related tasks.

=== boost/core rules ===

# Laravel Boost

- Laravel Boost is an MCP server that comes with powerful tools designed specifically for this application. Use them.

## Artisan

- Use the `list-artisan-commands` tool when you need to call an Artisan command to double-check the available parameters.

## URLs

- Whenever you share a project URL with the user, you should use the `get-absolute-url` tool to ensure you're using the correct scheme, domain/IP, and port.

## Tinker / Debugging

- You should use the `tinker` tool when you need to execute PHP to debug code or query Eloquent models directly.
- Use the `database-query` tool when you only need to read from the database.
- Use the `database-schema` tool to inspect table structure before writing migrations or models.

## Reading Browser Logs With the `browser-logs` Tool

- You can read browser logs, errors, and exceptions using the `browser-logs` tool from Boost.
- Only recent browser logs will be useful - ignore old logs.

## Searching Documentation (Critically Important)

- Boost comes with a powerful `search-docs` tool you should use before trying other approaches when working with Laravel or Laravel ecosystem packages. This tool automatically passes a list of installed packages and their versions to the remote Boost API, so it returns only version-specific documentation for the user's circumstance. You should pass an array of packages to filter on if you know you need docs for particular packages.
- Search the documentation before making code changes to ensure we are taking the correct approach.
- Use multiple, broad, simple, topic-based queries at once. For example: `['rate limiting', 'routing rate limiting', 'routing']`. The most relevant results will be returned first.
- Do not add package names to queries; package information is already shared. For example, use `test resource table`, not `filament 4 test resource table`.

### Available Search Syntax

1. Simple Word Searches with auto-stemming - query=authentication - finds 'authenticate' and 'auth'.
2. Multiple Words (AND Logic) - query=rate limit - finds knowledge containing both "rate" AND "limit".
3. Quoted Phrases (Exact Position) - query="infinite scroll" - words must be adjacent and in that order.
4. Mixed Queries - query=middleware "rate limit" - "middleware" AND exact phrase "rate limit".
5. Multiple Queries - queries=["authentication", "middleware"] - ANY of these terms.

=== pint/core rules ===

# Laravel Pint Code Formatter

- You must run `vendor/bin/pint --dirty --format agent` before finalizing changes to ensure your code matches the project's expected style.
- Do not run `vendor/bin/pint --test --format agent`, simply run `vendor/bin/pint --format agent` to fix any formatting issues.

=== pest/core rules ===

## Pest

- This project uses Pest for testing. Create tests: `php artisan make:test --pest {name}`.
- Run tests: `php artisan test --compact` or filter: `php artisan test --compact --filter=testName`.
- Do NOT delete tests without approval.
- CRITICAL: ALWAYS use `search-docs` tool for version-specific Pest documentation and updated code examples.
- IMPORTANT: Activate `pest-testing` every time you're working with a Pest or testing-related task.

</laravel-boost-guidelines>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/naykel76)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/naykel76)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
