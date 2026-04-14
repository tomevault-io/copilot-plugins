## altrompy

> Use when appropriate:

<laravel-boost-guidelines>
    # Laravel Boost Guidelines
    
    Follow strictly.
    
    ---
    
    ## Foundational Context
    
    Stack (strict versions):
    - PHP 8.4+
      - Laravel 12
      - Livewire 4
      - Flux UI (Free/Pro as installed)
      - Tailwind CSS v4
      - Pest v4
      - Laravel AI SDK (laravel/ai)
    
    You are expert in these versions.
    
    ---
    
    ## Skills Activation (MANDATORY)
    
    Activate immediately when applicable:
    
    - fluxui-development → Flux UI components
      - livewire-development → Livewire 4 / wire: directives
      - pest-testing → Writing or modifying tests
      - tailwindcss-development → Styling/UI work
      - developing-with-ai-sdk → Any AI functionality
      - developing-with-fortify → Authentication flows
    
    Do not delay activation.
    
    ---
    
    # search-docs (MANDATORY BEFORE CODING)
    
    Use `search-docs` before implementing features involving:
    Laravel core, Laravel 12 structure, Livewire, Pest, Tailwind v4, Fortify, or AI SDK.
    
    Guidelines:
    - Use broad queries: `['rate limiting', 'routing']`
      - Do NOT include package names
      - Use `"exact phrase"` when needed
      - Prefer multiple related queries
    
    Do not rely on training data for version-specific behavior.
    
    ---
    
    # Laravel 12 Structure (CRITICAL)
    
    - Middleware → `bootstrap/app.php`
      - Providers → `bootstrap/providers.php`
      - No `app/Http/Kernel.php`
      - No `app/Console/Kernel.php`
      - Console commands auto-register
      - Modified migration columns must re-declare ALL attributes
      - Prefer `casts()` method over `$casts`
    
    Follow this structure exactly.
    
    ---
    
    # Boost Tools
    
    Use when appropriate:
    
    - search-docs (first step for framework work)
      - list-artisan-commands
      - get-absolute-url
      - tinker (write)
      - database-query (read-only)
      - database-schema
      - browser-logs (recent only)
    
    ---
    
    # Conventions
    
    - Follow existing structure and naming.
      - Check sibling files before creating new ones.
      - **Verification**: Don't use scripts/tinker if tests already verify behavior.
      - **Frontend**: Suggest `npm run build` or `npm run dev` if UI changes aren't visible.
      - Do not create new base folders without approval.
      - Do not change dependencies without approval.
      - Create documentation only if requested.
      - Be concise.
    
    ---
    
    # PHP Rules
    
    - Curly braces always.
      - Use constructor property promotion.
      - No empty constructors unless private.
      - Explicit return types.
      - Proper parameter type hints.
      - Enum keys TitleCase.
      - Prefer PHPDoc over inline comments.
      - Use array shape types when useful.
    
    ---
    
    # Laravel Rules
    
    - Use `artisan make:` with `--no-interaction`.
      - Prefer Eloquent relationships over raw queries.
      - Avoid `DB::` unless necessary.
      - Prevent N+1 via eager loading.
      - Use Form Requests for validation.
      - Use gates/policies for authorization.
      - Use named routes + `route()`.
      - Queue slow operations.
      - Use `config()` (never `env()` outside config).
      - Create factories and seeders with models.
    
    ---
    
    # Testing (MANDATORY)
    
    Every change must be programmatically tested.
    
    - Use Pest.
      - Command: `php artisan test --compact`
      - Prefer feature tests.
      - Run minimal necessary tests.
      - Ensure affected tests pass.
      - Do not delete tests without approval.
    
    ---
    
    # Livewire
    
    - State lives on the server.
      - Validate and authorize inside actions.
      - Single root element.
      - Activate livewire-development.
    
    ---
    
    # Flux UI
    
    - Use `<flux:*>` components when available.
      - Activate fluxui-development.
    
    ---
    
    # Tailwind v4
    
    - Follow project conventions.
      - Use search-docs for v4 features.
      - Activate tailwindcss-development.
      - Use `@theme` config (not `@tailwind` directives).
    
    ---
    
    # Laravel AI SDK
    
    - All AI functionality must use `laravel/ai`.
      - Use search-docs before implementing.
      - Activate developing-with-ai-sdk.
    
    ---
    
    # Fortify
    
    - Use search-docs before implementing auth features.
      - Activate developing-with-fortify.
    
    ---
    
    # Formatting
    
    Before finalizing:
    `vendor/bin/pint --dirty --format agent`
</laravel-boost-guidelines>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jdecode)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jdecode)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
