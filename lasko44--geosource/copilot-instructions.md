## geosource

> This document defines the architectural decisions and coding standards for this project. All code must follow these guidelines.

# Project Coding Standards

This document defines the architectural decisions and coding standards for this project. All code must follow these guidelines.

---

# Architectural Decision Records (ADR)

## Core Architecture

### ADR-001: Skinny Controllers, Fat Services

### Decision
Controllers contain **zero business logic**. All logic lives in service classes.


### Rationale
1. **Testability**: Services can be unit tested without HTTP layer. Controllers need feature tests.
2. **Reusability**: Services can be called from controllers, jobs, commands, or other services.
3. **Readability**: A 50-line controller is easier to understand than a 500-line one.
4. **Single Responsibility**: Each class has one reason to change.
5. **Debugging**: When something breaks, you know where to look based on the type of issue.

### Rules

**Controller responsibilities:**
- Receive HTTP request
- Call service method(s)
- Return HTTP response

**Service responsibilities:**
- Business logic
- Database operations
- External API calls
- Complex calculations

### Example
```php
// ❌ WRONG - Logic in controller
public function store(Request $request) {
    // 50+ lines of validation, authorization, business logic...
}

// ✅ CORRECT - Delegate to service
public function store(StoreScanRequest $request, ScanService $service): RedirectResponse
{
    $scan = $service->executeScan($request->user(), $request->validated());
    return redirect()->route('scans.show', $scan);
}
```

---

### ADR-002: Form Requests for Validation

### Decision
All validation logic lives in **Form Request** classes, not controllers.

### Rationale
1. **Reusability**: Same validation can be used by multiple endpoints.
2. **Separation of concerns**: Controllers don't need to know validation rules.
3. **Authorization co-location**: `authorize()` method keeps auth logic with related validation.
4. **Cleaner controllers**: No `$request->validate([...])` blocks.
5. **Custom error messages**: Centralized in one place.
6. **Data transformation**: Helper methods like `getTier()` encapsulate input transformation.

### Rules
```php
class StoreScanRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Scan::class);
    }

    public function rules(): array
    {
        return [
            'url' => 'required|url',
            'tier' => 'nullable|in:basic,pro,full',
        ];
    }

    public function getTier(): string
    {
        return $this->input('tier', 'basic');
    }
}
```

---

### ADR-003: Policy Classes for Authorization

### Decision
Use **Policy classes** for all model-based authorization, not inline checks.


### Rationale
1. **DRY**: Authorization logic written once, used everywhere.
2. **Testability**: Policies are easily unit tested.
3.**Consistency**: All authorization follows the same pattern.
4.**Discoverability**: All rules for a model are in one file.

### Rules
```php
// Controller - use $this->authorize()
public function show(Scan $scan): Response
{
    $this->authorize('view', $scan);
    // ...
}

// Policy - all authorization logic here
class ScanPolicy
{
    public function view(User $user, Scan $scan): bool
    {
        return $scan->user_id === $user->id
            || $user->allTeams()->contains('id', $scan->team_id);
    }
}
```

---

### ADR-004: Eloquent ORM Only

### Decision
All database queries must use **Eloquent ORM**. Raw `DB::` statements are prohibited.

### Rationale
1. **Consistency**: One way to access the database. No mixing of patterns.
2. **Security**: Eloquent handles parameter binding automatically, reducing SQL injection risk.
3. **Maintainability**: Eloquent queries are more readable and self-documenting.
4. **Model features**: Mass assignment protection, events, observers, and accessors/mutators only work with Eloquent.
5. **Relationships**: Eager loading, lazy loading, and relationship methods require Eloquent.
6. **Testability**: Eloquent models can be easily mocked and factories used for testing.
7. **Refactoring**: Changing table structure only requires updating the model, not hunting for raw queries.

### Prohibited
```php
DB::select('...');
DB::insert('...');
DB::statement('...');
DB::table('...');
DB::raw('...');
```

### Allowed
```php
User::where('status', 'active')->get();
Scan::create(['url' => $url]);
$user->scans()->where('status', 'pending')->get();
```

### Exceptions
Raw expressions (`selectRaw`, `whereRaw`, `orderByRaw`) are allowed for database-specific features like pgvector. Must include a comment explaining why.

---

## Security

### ADR-005: Return 404 Instead of 403

### Decision
Return **404 Not Found** instead of **403 Forbidden** when authorization fails. Log the actual 403 internally.

### Context
When a user attempts to access a resource they're not authorized to view (e.g., another user's scan), we need to decide what HTTP status code to return.

### Rationale
Returning 403 Forbidden leaks information and enables **resource enumeration attacks**:
```
GET /scans/12345 → 403  (attacker learns: scan 12345 EXISTS)
GET /scans/99999 → 404  (attacker learns: scan 99999 doesn't exist)
```

By returning 404 for both cases, attackers can't distinguish "exists but forbidden" from "doesn't exist."

### Benefits
1. **Prevents enumeration**: Attackers can't probe for valid resource IDs.
2. **Security monitoring**: Logged 403s can trigger alerts for suspicious activity.
3. **Same user experience**: Legitimate users see "not found" which is accurate from their perspective.

### Implementation
```php
// bootstrap/app.php
$exceptions->render(function (AuthorizationException $e, $request) {
    Log::warning('Authorization denied (returned as 404)', [
        'user_id' => $request->user()?->id,
        'ip' => $request->ip(),
        'url' => $request->fullUrl(),
        'method' => $request->method(),
    ]);

    throw new NotFoundHttpException('Not Found');
});
```

### Monitoring
Set up alerts for patterns like:
- Multiple 403s from same IP in short time
- 403s targeting sequential resource IDs
- 403s from authenticated users on resources they shouldn't know about

---

## Code Quality

### ADR-006: Type Hints and Return Types Everywhere

### Decision
All method parameters and return types must have explicit type hints. **Every function must declare a return type.**

### Rationale
1. **Self-documenting**: The signature tells you what the method expects and returns.
2. **IDE support**: Autocomplete, refactoring, and error detection.
3. **Runtime safety**: PHP enforces types, catching bugs early.
4. **Static analysis**: Tools like PHPStan can verify correctness.
5. **Contract clarity**: Return types make the method's contract explicit.

### Rules
```php
// ✅ CORRECT
public function executeScan(User $user, string $url, ?Team $team = null): Scan
public function sendNotification(User $user): void
public function findScan(string $uuid): ?Scan

// ❌ WRONG - Missing return type
public function executeScan(User $user, string $url)
```

---

### ADR-007: Class Documentation

### Decision
Every class must have a docblock explaining its purpose.

### Rationale
1. **Discoverability**: Developers can quickly understand what a class does without reading all its code.
2. **Onboarding**: New team members understand the codebase faster.
3. **IDE integration**: Docblocks appear in hover tooltips and autocomplete.
4. **Intentional design**: Writing a description forces you to clarify the class's single responsibility.

### Rules
```php
/**
 * Handles website scan execution and lifecycle management.
 */
class ScanService
{
    // ...
}

/**
 * Exports a scan report as a PDF document.
 */
class ExportPdfController extends Controller
{
    // ...
}
```

### Guidelines
- Keep descriptions concise (1-3 sentences)
- Focus on **what** the class does, not **how**
- For controllers, describe the action it performs
- For services, describe the domain it manages

---

### ADR-008: Array Access via Arr Helper

### Decision
Always use `Illuminate\Support\Arr::get()` for array access instead of direct bracket notation.

### Rationale
1. **Null safety**: `Arr::get($data, 'key')` returns null instead of throwing an error.
2. **Default values**: `Arr::get($data, 'key', 'default')` is cleaner than `$data['key'] ?? 'default'`.
3. **Nested access**: `Arr::get($data, 'user.profile.name')` handles missing intermediate keys.
4. **Consistency**: One pattern across the codebase.

### Rules
```php
use Illuminate\Support\Arr;

// ✅ CORRECT
$name = Arr::get($response, 'data.user.name', 'Unknown');

// ❌ WRONG
$name = $response['data']['user']['name'];
$name = $data['user']['name'] ?? 'Default';
```

---

## Controller Rules

### ADR-009: Method Injection in Controllers

### Decision
Use **method injection** in controllers, not constructor injection.

### Rationale
1. **Only inject what you need**: Each method declares its own dependencies. The `index` method might not need `TokenService`, so why inject it for every action?
2. **Explicit dependencies**: Reading a method signature tells you exactly what it needs.
3. **Testing simplicity**: Mock only what the specific method uses.
4. **Controller lifecycle**: Controllers are instantiated per-request. Constructor injection offers no performance benefit and may instantiate unused services.
5. **Laravel convention**: Taylor Otwell (Laravel creator) recommends method injection for controllers.

### Rules
```php
// ✅ CORRECT - Method injection
public function show(Scan $scan, ScanService $scanService): Response
{
    return $scanService->getScanData($scan);
}

// ❌ WRONG - Constructor injection
public function __construct(private ScanService $scanService) {}
```

---

### ADR-010: Invokable Controllers for Single Actions

### Decision
Use native Laravel **invokable controllers** for single-action endpoints rather than the `lorisleiva/laravel-actions` package.

### Context
When refactoring controllers to follow single-responsibility principle, we needed a pattern for non-CRUD actions like `ExportPdf`, `ScanCancel`, `CheckCooldown`, etc.

### Rationale
1. **Separation of concerns**: Controllers handle HTTP, services handle business logic, jobs handle async work.
2. **YAGNI**: We don't have use cases requiring the same action as both HTTP endpoint and queued job.
3. **No external dependencies**: Reduces maintenance burden and potential security vulnerabilities.
4. **Team familiarity**: Standard Laravel patterns are easier for new developers to understand.

### Rules
- Resource controllers: 7 standard actions only (index, create, store, show, edit, update, destroy)
- Non-CRUD actions: Use invokable controllers with `__invoke()`
- Naming: `{Action}{Model}Controller` (e.g., `ExportPdfController`)

### Example
```php
Route::post('/scans/{scan}/cancel', ScanCancelController::class)->name('scans.cancel');

class ScanCancelController extends Controller
{
    public function __invoke(Scan $scan, ScanService $service): RedirectResponse
    {
        $service->cancel($scan);
        return redirect()->route('scans.index');
    }
}
```

---

### ADR-011: No Private Methods in Controllers

### Decision
Controllers must not contain `private` or `protected` methods.

### Context
Private controller methods indicate business logic that should live elsewhere.

### Rationale
1. **Signals misplaced logic**: If you need a helper method, it belongs in a service.
2. **Untestable**: Private methods can't be unit tested directly.
3. **Not reusable**: Other controllers can't use private methods.
4. **Controller bloat**: Private methods are a slippery slope to 500-line controllers.

### Rules
| Logic Type | Location |
|------------|----------|
| Business logic | Service |
| Data transformation | Form Request or Service |
| Authorization | Policy |
| Query building | Service or Model scope |

---

## Service Rules

### ADR-012: Services Use Constructor Injection

### Decision
While controllers use method injection, **services use constructor injection**.

### Rationale
1. **Different lifecycle**: Services are often singletons or have consistent dependencies.
2. **All methods need same dependencies**: Unlike controllers, service methods typically share dependencies.
3. **Cleaner method signatures**: Service methods focus on business parameters, not infrastructure.

### Rules
```php
class ScanService
{
    public function __construct(
        protected SubscriptionService $subscriptionService,
        protected TokenService $tokenService,
    ) {}

    public function executeScan(User $user, string $url): Scan
    {
        // Uses $this->subscriptionService and $this->tokenService
    }
}
```

---

### ADR-013: Singleton Registration for Stateless Services

### Decision
Register only **truly stateless** services as singletons. Services with mutable state must NOT be singletons.

### Context
The application has RAG services that benefit from singleton registration, and GEO scoring services that must NOT be singletons due to mutable state.

### Rationale for Singletons
1. **Resource Efficiency**: `EmbeddingService` creates HTTP clients for OpenAI API calls. Multiple instances waste memory and duplicate connection pools.
2. **Cache Sharing**: `EmbeddingService` caches embeddings to avoid redundant API calls. A singleton ensures all code paths share the same cache.
3. **Shared Dependencies**: Without singletons, `EmbeddingService` would be instantiated multiple times.
4. **Configuration Consistency**: Services read config values at construction. Singletons guarantee consistent configuration.

### Registered as Singletons
- `EmbeddingService` - HTTP client reuse, embedding cache
- `ChunkingService` - Stateless text processing
- `VectorStore` - Stateless database queries
- `RAGService` - Orchestrates vector search

### NOT Registered (mutable state)
- `GeoScorer` - Has `forTier()` that mutates `$scorers` array
- `EnhancedGeoScorer` - Wraps GeoScorer, same mutation issue

### Rule
If a service has methods that modify internal state, it must NOT be a singleton. State changes would leak between requests.

---

## Project Organization

### ADR-014: Feature-Based Directory Structure

### Decision
Organize controllers and requests by **feature/domain**, not by type.

### Rationale
1. **Cohesion**: Related code lives together.
2. **Discoverability**: Find all scan-related controllers in one folder.
3. **Scalability**: Adding features doesn't bloat a single directory.
4. **Bounded contexts**: Mirrors domain-driven design principles.

### Structure
```
app/Http/Controllers/
├── Scans/
│   ├── ScanController.php
│   ├── BulkScanController.php
│   └── ExportPdfController.php
├── GA4/
│   ├── GA4ConnectionController.php
│   └── GA4CallbackController.php
└── Teams/
    └── TeamController.php
```

---

# Architecture Overview

```
HTTP Request
     │
     ▼
┌─────────────────┐
│  Form Request   │  Validation, authorization, data transformation
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Controller    │  Receive request → call service → return response
│                 │  NO business logic, NO private methods
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    Service      │  Business logic, database ops, external APIs
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    Policy       │  Model-level authorization (can user do X to Y?)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│     Model       │  Eloquent relationships, scopes, accessors
└─────────────────┘
```

---

# Quick Reference

## Do
- Put all business logic in services
- Put all validation in Form Requests
- Put all authorization in Policies
- Use Eloquent for all database queries
- Return 404 for authorization failures (log 403 internally)
- Add return types to all methods
- Add docblocks to all classes
- Use `Arr::get()` for array access
- Use method injection in controllers
- Use constructor injection in services

## Don't
- Business logic in controllers
- Validation in controllers (`$request->validate()`)
- Private/protected methods in controllers
- Constructor injection in controllers
- Raw DB queries (`DB::select`, `DB::table`, etc.)
- Return 403 for authorization failures (leaks info)
- Missing return types
- Missing class docblocks
- Direct array access (`$data['key']`)
- Register mutable services as singletons

## Naming Conventions
| Type | Pattern | Example |
|------|---------|---------|
| Resource Controller | `{Model}Controller` | `ScanController` |
| Invokable Controller | `{Action}{Model}Controller` | `ExportPdfController` |
| Service | `{Feature}Service` | `ScanService` |
| Form Request | `{Action}{Model}Request` | `StoreScanRequest` |
| Policy | `{Model}Policy` | `ScanPolicy` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lasko44) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
