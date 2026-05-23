## laravel

> Project rules for PHP, Laravel, Inertia, Vite, Npm, React, Typescript, and Tailwind


# Context

This is a Laravel app using PHP 8+, with strict types `declare(strict_types=1);`, Composer, Laravel 12, InertiaJS 2, Vite, npm, React 19 with TypeScript, Tailwind 4, and radix-ui.

No other libraries or packages should be added unless the user approves.
The developer has the option of using Sail, Herd, or artisan serve to run their app.
Make use of `config('app.url)` or the URL helper to generate URLs to provide to the user.

# How to behave

- Think through and create a step by step plan before acting so you have a todo list to work through
    - The first step is always to evaluate the existing codebase to gather existing conventions to use in future steps
    - The final step is always to validate the modified code using `composer lint` and `composer test`
- Work through the plan step by step - creating failing tests, then the functionality to ensure the tests pass
- Follow all rules and Laravel 11+ conventions closely
- New code MUST strictly follow the same style as existing code and Laravel conventions - naming, structure, patterns, formatting, architecture
  **Directory structure adherence**: Always follow Laravel's standard directory structure (`app/Models/`, `app/Http/Controllers/`, `app/Http/Middleware/`, etc.) without creating non-standard directories unless explicitly approved.
  **Naming conventions**: Use Laravel's established naming patterns - `PascalCase` for controllers and models, `snake_case` for database tables and columns, `kebab-case` for URLs, and descriptive migration names with `artisan make:migration`.

# PHP Standards

- Use PHP 8+ features if they match the current code conventions: match(), union types, nullsafe operators, named arguments, attributes/annotations, constructor property promotion,
- Always enforce strict typing with declare(strict_types=1), scalar type hinting, return types, parameter types, and property types
- Use backed enums for fixed values
- Use short, focused, well-name, and testable methods
- Follow PSR-12 and PSR-4 coding standards.
- Prefer value objects over raw arrays when appropriate.
- Avoid over-engineering — keep things simple and pragmatic.
- Never leave TODOs or FIXMEs without clear context or a linked issue.
- Never leave comments within code blocks, only on methods.

# PHPDoc

- Write PHPDocs when needed based on this guide

Below is an example of a valid Laravel documentation block. Note that the @param attribute is followed by two spaces, the argument type, two more spaces, and finally the variable name:

```php
/**
 * Register a binding with the container.
 *
 * @param  string|array  $abstract
 * @param  \Closure|string|null  $concrete
 * @param  bool  $shared
 * @return void
 *
 * @throws \Exception
 */

public function bind($abstract, $concrete = null, $shared = false)
{
    // ...
}
```

When the `@param` or `@return` attributes are redundant due to the use of native types, they can be removed:

```php
/*
* Execute the job.
*/
public function handle(AudioProcessor $processor): void
{
    //
}
```

However, when the native type is generic, please specify the generic type through the use of the @param or @return attributes:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file'),
    ];
}
```

# Creating new Laravel files

- Always use artisan commands to generate Laravel files: `artisan make:xyz`

# Laravel tests

- Always write Pest tests for new or updated functionality
- Tests must be written using Pest functions and assertions
- You MUST NOT write PHPUnit tests
- When asserting status codes on a response, use the specific method like `assertOk`, `assertForbidden`, `assertNotFound` etc, instead of using `assertStatus(403)` or similar
- When creating models for tests, you **MUST** use the factories for the models. Make use of custom states available
- Run the tests with `composer test` after finalizing functionality
- Run `composer lint` after changes

<example-pest-test path="tests/Unit/ExampleTest.php">
```php
<?php
test('that true is true', function () {
    expect(true)->toBeTrue();
});
```
</example-pest-test>

<example-pest-test path="tests/Feature/ExampleTest.php">

```php
<?php

it('returns a successful response', function () {
    $response = $this->get('/');
    $response->assertOk();
});
```

</example-pest-test>

---

# Laravel Coding Guidelines

## Backend Architecture & Laravel Patterns

### Laravel Framework Standards

- Implement proper type hints for all method parameters and return types (`Response`, `RedirectResponse`, etc.)
- Use Laravel's built-in validation, authentication, and authorization systems
- Commands created in `app\Console\Commands\` are automatically registered and available to use
- Never use `env()` directly in code, use `config('app.name')` for example
    - Use new environment variables for any new sensitive or configurable options, then add to a config file for use with `config('')`
- Implement proper error handling with custom exception classes when needed
- Listeners auto-listen for the events if they are type-hinted correctly
- Scheduled commands live in `routes/console.php`
- Use `DB::transaction()` in actions with multiple database operations
- Use `list<string>` for indexed arrays and detailed array shapes in docblocks:

```php
/**
 * @var list<string>
 */
protected $fillable = ['name', 'email'];

/**
 * @return array{
 *     status: 'App\Enums\DatabaseStatus',
 *     metadata: 'array',
 *     created_at: 'datetime',
 * }
 */
protected function casts(): array
{
    return [
        'status' => DatabaseStatus::class,
        'metadata' => 'array',
        'created_at' => 'datetime',
    ];
}
```

- Use dedicated classes for complex data structures rather than arrays:

```php
class DnsResult
{
    public function __construct(
        public readonly string $type,
        public readonly string $domain,
        public readonly string $ip
    ) {}

    public static function fromEnvironment(Environment $environment): self
    {
        return new self(
            type: $environment->type,
            domain: $environment->domain,
            ip: $environment->ip
        );
    }
}
```

### Controller Structure

- Keep controllers thin and focused on HTTP concerns only. Almost always return `Inertia` responses. Delegate to `Services` when needed.
- Use Form Request classes for complex validation logic (`ProfileUpdateRequest`)
- Return proper response types (`Inertia\Response` for renders, `RedirectResponse` for redirects)
- Group related functionality in subdirectories (`Settings/`, `Auth/`) within `App/Http/Controllers`
- Use resource controllers and route model binding where appropriate
- Delegate to a queued job for long-running tasks

### Model Best Practices

- Use modern Laravel model patterns with proper docblocks (`@use HasFactory<\Database\Factories\UserFactory>`)
- Implement proper `$fillable` and `$hidden` arrays for mass assignment protection
- Begin queries from a Model or QueryBuilder, not `DB::`
- Use the modern `casts()` method for attribute casting
- Use Eloquent relationships properly with correct type hints. Prefer relationship methods over raw queries or manual joins.
- Each Model must have a {Model}Factory
- Use eager loading to prevent N+1 queries

### Database & Migrations

- Create new migrations with `artisan make:migration` using a descriptive migration name that indicates the change
- Implement proper foreign key constraints and indexes
    - Use `->constrained()->cascadeOnDelete()` or `->nullOnDelete()` patterns for foreign key relationships
- Keep migrations atomic and reversible
- Omit `down()` in new migrations
- Use database factories for testing data
- Use eager loading to prevent N+1 queries

### Inertia.js Integration

- Implement proper data sharing through `HandleInertiaRequests` middleware
- Enable and configure Server-Side Rendering (SSR) for better SEO and performance
- Make use of Inertia's new deferred props, polling, prefetching, load when visible, and props merging, available in Inertia JS v2

### React/TypeScript Standards

- Use TypeScript for all frontend code with proper type definitions
- Use `export default function Something(){...}`
- When importing local components, use `import SomeComponent from '@/Components/SomeComponent'` instead of relative paths and always leave off the file extension
- Implement component composition patterns with Radix UI primitives
- Use modern React patterns (hooks, function components, proper state management)
- Follow accessibility best practices with ARIA attributes
- Implement proper error boundaries and loading states

### Styling & UI Components

- Use Tailwind CSS v4 syntax and properties
- Implement design system with reusable components using shadcn/ui patterns
- Implement proper responsive design with mobile-first approach

### Testing Requirements

- You must use Pest PHP for expressive testing syntax. PHPUnit tests have been banned
- Implement feature tests for all user-facing functionality
- Write unit tests for complex business logic
- Always use factories for test data generation. Make use of 'factory states' when possible.
    - Use `fake()` helper instead of `$this->faker`.
- If xdebug or pcov is installed (`php -m | grep -i xdebug`) you must maintain high test coverage with meaningful assertions. You can test the code and validate coverage percentage above 85% using this command `XDEBUG_MODE=coverage vendor/bin/pest --coverage --min=85`

### Code Quality & Formatting

- Use `npm run format:check` to verify prettier formatting
- Use `npm run lint` to fix eslint issues
- Use proper import organization with `prettier-plugin-organize-imports`
- Implement proper error handling and logging

### Security & Best Practices

- Use Laravel's built-in authentication with proper middleware
- Implement CSRF protection on all forms
- Use rate limiting for sensitive endpoints

### Data Validation & Sanitization

- Always add FormRequest classes extending `Illuminate\Foundation\Http\FormRequest` in `app/Http/Requests/` with Laravel's validation rules in an array, like this example:
  <form-request-validation-array-example>

    ```php
    /**
    * Get the validation rules that apply to the request.
    *
    * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
    */
    public function rules(): array
    {
      return [
          'email' => ['required', 'string', 'email'],
          'password' => ['required', 'string'],
      ];
    }
    ```

</form-request-validation-array-example>

## Service Providuers Class Structure

- Do not create new service providers unless approved by the user to be created and added to `bootstrap/providers.php`

## HTTP Requests

Use Laravel's HTTP client with retry logic, connection exception handling, and custom exception throwing:

```php
protected function client(): PendingRequest
{
    return Http::withToken($this->apiKey)
        ->baseUrl("{$this->host}/api")
        ->asJson()
        ->acceptJson()
        ->retry($this->maxAttempts, when: fn (Throwable $e) => $this->retryWhen($e))
        ->throw(fn (Response $response) => throw new CustomRequestException($response));
}

protected function retryWhen(Throwable $exception): bool
{
    return $exception instanceof ConnectionException;
}
```

### Service Method Patterns

Keep service methods focused and use descriptive names:

```php
public function createCustomer(Organization $organization): array
{
    $response = $this->client()
        ->post('/customers', [
            'external_id' => $organization->identifier,
            'name' => $organization->name,
            'email' => $organization->billing_email,
        ]);

    return $response->json();
}
```

---
> Source: [appdotbuilder/damkar-morowali-utara](https://github.com/appdotbuilder/damkar-morowali-utara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
