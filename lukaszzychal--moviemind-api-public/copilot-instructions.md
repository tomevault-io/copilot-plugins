## coding-standards

> - Rules are tools, not goals in themselves


# Coding Standards

## Philosophy: Pragmatic Approach

- Rules are tools, not goals in themselves
- Readability > Conciseness
- Simple code is better than "perfect" code that follows all rules
- Don't force rules

## KISS (Keep It Simple, Stupid)

- **Simplicity over complexity** - choose simpler solutions
- **Don't overdo abstraction** - sometimes simple code is better
- **Avoid premature optimization** - working code first, then optimization
- **Readability > Performance** (in most cases)
- **The simplest solution that works** is the best

## SOLID (apply pragmatically)

- **SRP**: One class = one responsibility
- **OCP**: Open for extension, closed for modification
- **LSP**: Subclasses must be substitutable
- **ISP**: Specific interfaces, not general ones
- **DIP**: Depend on abstractions, not concrete implementations

## DRY (Don't Repeat Yourself)

- Refactor duplication when it occurs in 3+ places
- Don't overdo abstraction - sometimes duplication is more readable
- Refactor only when it makes practical sense

## CUPID

- **C**omposable - easy to compose
- **U**nix philosophy - does one thing well
- **P**redictable - predictable
- **I**diomatic - follows Laravel conventions
- **D**omain-based - domain-based

## Architecture Choice

### Principle: Start Simple, Scale When Needed

**Start with a simple solution, add complexity only when justified.**

### Simple Solutions (Transaction Script)

**Use when:**
- ✅ Small to medium project (MVP, prototypes)
- ✅ Simple business logic (CRUD, simple operations)
- ✅ Small team (1-3 people)
- ✅ Fast development and delivery
- ✅ Low risk of requirement changes
- ✅ Simple domain (doesn't require complex business logic)

**Examples:**
- Laravel Controllers with direct access to Models
- Simple Services with business logic
- Repository pattern (if needed)
- Minimal abstraction

**Advantages:**
- Fast development
- Easy to understand
- Low maintenance costs (initially)
- Fast iteration

### Advanced Architectures (DDD, Hexagonal, Clean Architecture)

**Use when:**
- ✅ Large, complex project
- ✅ Complex business logic (many rules, states)
- ✅ Multiple teams working in parallel
- ✅ Long project lifespan (5+ years)
- ✅ High risk of requirement changes
- ✅ Complex domain (domain experts, rich domain model)
- ✅ High testability and isolation required
- ✅ Need for multiple ports (API, CLI, Web, Events)

**Examples:**
- Domain-Driven Design (DDD) - Entities, Value Objects, Domain Services
- Hexagonal Architecture - Ports & Adapters
- Clean Architecture - layers with dependency rules
- CQRS + Event Sourcing (when needed)

**Advantages:**
- High testability
- Better separation of responsibilities
- Easier to add new features
- Framework independence
- Long-term maintainability

**Disadvantages:**
- More code (boilerplate)
- Slower initial development
- More abstraction (can be harder to understand)
- Higher initial costs

### Migration: Simple → Advanced

**When to refactor to more advanced architecture:**

1. ✅ Code becomes hard to maintain
2. ✅ Hard to add new features without changing existing ones
3. ✅ Tests are hard to write
4. ✅ Lots of business logic duplication
5. ✅ Team grows and needs better organization
6. ✅ Domain becomes more complex
7. ✅ Framework independence required

**When NOT to refactor:**
- ❌ "For the principle" - without a concrete problem
- ❌ When code works well and is easy to maintain
- ❌ When project is small and simple
- ❌ When there's no time/resources for refactoring

### Recommendations for MovieMind API

**Current state:**
- Simple architecture with Services, Repositories, Controllers
- Event-Driven for asynchronous operations (Jobs)
- Laravel conventions

**When to consider DDD/Hexagonal:**
- When business logic becomes very complex
- When need for many different ports (Web, CLI, Queue, Events)
- When team grows and needs better separation
- When need for independence from Laravel (e.g., shared kernel)

**Principle:**
**Start simple, refactor when concrete problems arise, not "just in case".**

## GRASP (General Responsibility Assignment Software Patterns)

GRASP patterns help assign responsibilities in object-oriented design:

- **Information Expert** - assign responsibility to class with information needed for execution
- **Creator** - assign creation responsibility to class that uses/contains object
- **Controller** - assign handling responsibility to class representing system/facade
- **Low Coupling** - minimize dependencies between classes
- **High Cohesion** - keep related functionality together
- **Polymorphism** - use polymorphism for behavioral variations
- **Pure Fabrication** - create classes that don't represent domain concepts (e.g., Repository)
- **Indirection** - use intermediate objects to loosen dependencies
- **Protected Variations** - protect against variations through encapsulation

## YAGNI (You Aren't Gonna Need It)

- **Don't add functionality until needed** - avoid premature abstraction
- **Solve today's problems, not tomorrow's** - focus on current requirements
- **Refactor when needed** - add complexity only when there's a concrete need

## Code Smells (fix when they hinder work)

### Structural Smells
- **God Class/Method** - class/method does too much → split into smaller classes/methods
- **Long Parameter List** - use DTO/Request object → create parameter object
- **Feature Envy** - method uses more data from another class → move method to that class
- **Data Clumps** - groups of data always together → use Value Object
- **Primitive Obsession** - using primitives instead of objects → use Value Objects
- **Duplicate Code** - same code in many places → extract to common method/class
- **Long Method** - method is too long → extract methods
- **Large Class** - class has too many responsibilities → split into smaller classes

### Behavioral Smells
- **Shotgun Surgery** - one change requires many small changes → consolidate related code
- **Divergent Change** - class changes for many reasons → split responsibilities
- **Parallel Inheritance Hierarchies** - similar inheritance hierarchies → combine or use composition
- **Lazy Class** - class doesn't do enough → embed or combine with another class
- **Speculative Generality** - code for future that isn't needed → remove (YAGNI)

### Dependency Smells
- **Dependency Inversion Violation** - high-level modules depend on low-level → use interfaces/abstractions
- **Tight Coupling** - classes depend too much on each other → introduce interfaces/abstractions
- **Circular Dependency** - classes depend on each other → break the cycle

## Design Patterns (use when appropriate)

### Creational Patterns
- **Factory** - create objects without specifying exact class
- **Builder** - construct complex objects step by step
- **Singleton** - ensure only one instance (use sparingly, prefer dependency injection)

### Structural Patterns
- **Repository** - abstraction of data access layer
- **Adapter** - make incompatible interfaces work together
- **Decorator** - add behavior to objects dynamically
- **Facade** - simplified interface to complex subsystem

### Behavioral Patterns
- **Strategy** - define family of algorithms, make them interchangeable
- **Observer** - notify many objects of state changes (Laravel Events)
- **Command** - encapsulate requests as objects (Laravel Jobs)
- **Template Method** - define algorithm skeleton, let subclasses fill details

## Architectural Patterns

- **Repository Pattern** - data access abstraction (already used in project)
- **Service Layer** - business logic layer (already used in project)
- **Event-Driven** - communication through events (Laravel Events/Listeners)
- **CQRS** - separate read and write models (when needed)
- **Hexagonal Architecture** - ports and adapters (when needed)
- **Clean Architecture** - dependency rule with layers (when needed)

## Refactoring and Optimization Rules

### When to refactor
1. ✅ **Code duplication** - same logic in 3+ places
2. ✅ **Dependency violations** - DIP, tight coupling
3. ✅ **Code smells** - God Class, Long Method, etc.
4. ✅ **Hard to test** - hard to write unit tests
5. ✅ **Hard to extend** - adding features requires many changes
6. ✅ **Performance issues** - identified bottlenecks (measure first!)

### Refactoring during feature development
- **Always refactor when adding features** - improve code quality incrementally
- **Fix code smells when you encounter them** - don't accumulate technical debt
- **Apply SOLID/GRASP when making changes** - use principles pragmatically
- **Extract common logic** - use Repository, Service, or Trait when appropriate

### Optimization Guidelines
- **Measure first** - profile before optimizing
- **Optimize bottlenecks** - focus on real performance problems
- **Don't optimize prematurely** - working code first, then optimization
- **Consider readability** - readable code is often fast enough

## Feature Flags

### Flag Types

The project uses two types of feature flags:

#### 1. Product Flags

**Characteristics:**
- Long-term enabling/disabling of features in production
- Categories: `core_ai`, `moderation`, `public_api`, `billing`, `i18n`, `performance`, `analytics`, `recommendations`, `operations`
- Can be `togglable: true` (managed by admin API)
- Default value depends on feature (often `true` for core features)

**Examples:**
- `ai_description_generation` - core_ai, default: true, togglable: true
- `tmdb_verification` - moderation, default: true, togglable: true
- `public_jobs_polling` - public_api, default: true, togglable: true

**When to use:**
- Features that can be enabled/disabled in production
- Features requiring gradual rollout
- Features that may require quick rollback

#### 2. Developer Flags

**Characteristics:**
- Temporary flags used during development
- Category: `experiments`
- Always `default: false` (disabled by default)
- Always `togglable: false` (cannot be toggled via API - security)
- Description contains words: "Experimental", "WIP", "Work in progress"

**Examples:**
- `generate_v2_pipeline` - experiments, default: false, togglable: false
- `description_style_packs` - experiments, default: false, togglable: false

**When to use:**
- Any new or risky feature disrupting stability
- Features in development (WIP)
- Experimental features tested before deployment
- Features that may cause problems in production

### Developer Flags Lifecycle

1. **Creation:**
   - Create developer flag when starting work on feature
   - Set `category: 'experiments'`, `default: false`, `togglable: false`
   - Add description indicating it's an experimental feature

2. **Testing:**
   - Test feature manually by enabling flag in dev/staging environment
   - DO NOT enable developer flags in production
   - Use developer flags to isolate new features

3. **Deployment:**
   - After deploying feature to production, **mandatorily remove developer flag**
   - Remove code related to flag (conditions `Feature::active()`)
   - Remove flag definition from `config/pennant.php`
   - Remove flag class from `app/Features/`
   - Update tests (remove tests related to flag)

### Naming Rules

- **Product flags:** descriptive feature names (e.g., `ai_description_generation`, `tmdb_verification`)
- **Developer flags:** names with experimental prefix or suffix `_v2`, `_experimental` (e.g., `generate_v2_pipeline`)

### Documentation

- Every flag must have `description` in `config/pennant.php`
- Developer flags must have information about temporary nature in description
- After removing developer flag, update documentation

### Configuration Example

```php
// Product flag
'ai_description_generation' => [
    'class' => ai_description_generation::class,
    'description' => 'Enables AI-generated movie/series descriptions.',
    'category' => 'core_ai',
    'default' => true,
    'togglable' => true,
],

// Developer flag
'generate_v2_pipeline' => [
    'class' => generate_v2_pipeline::class,
    'description' => 'Experimental v2 generation flow (WIP - remove after deployment).',
    'category' => 'experiments',
    'default' => false,
    'togglable' => false,
],
```

## Coding Standards

- **PSR-12** - formatting (enforced by Pint)
- **Laravel Conventions** - Laravel conventions
- **Type hints** - always use types
- **Strict types** - `declare(strict_types=1);` in PHP files
- **Return types** - always specify return type
- **Repository Pattern** - use Repository for data access, not direct Model queries in Jobs/Services
- **Dependency Injection** - prefer constructor injection, use service container

## Readable Method Names

**Principle:** Method names should be **self-describing** - reading the method name, you should immediately know what it does.

### ✅ GOOD method names

#### Controllers (RESTful, standard Laravel)
- `index()` - list resources
- `show()` - single resource
- `store()` - create new resource
- `update()` - update resource
- `destroy()` - delete resource
- `search()` - search with criteria
- `related()` - related resources
- `refresh()` - refresh data

#### Services (business actions, verbs)
- `retrieveMovie()` - retrieve movie
- `searchMovies()` - search movies
- `getSimilarMovies()` - get similar movies
- `createFromTmdb()` - create from TMDB data
- `verifyMovie()` - verify movie
- `formatMovieList()` - format movie list
- `findBySlugWithRelations()` - find by slug with relations
- `normalizeDescriptionId()` - normalize description ID

#### Actions (business operations, verbs)
- `handle()` - main action method (standard for Actions)
- `queueMovieGeneration()` - queue generation
- `getRelatedMovies()` - get related movies
- `buildDisambiguationOptions()` - build disambiguation options

#### Response Formatters (response formatting)
- `formatSuccess()` - format success
- `formatError()` - format error
- `formatNotFound()` - format "not found"
- `formatRelatedMovies()` - format related movies
- `formatGenerationQueued()` - format "generation queued"
- `formatDisambiguation()` - format disambiguation

#### Repositories (data operations)
- `findBySlug()` - find by slug
- `findBySlugWithRelations()` - find by slug with relations
- `searchMovies()` - search movies
- `create()` - create (from Model)
- `update()` - update (from Model)
- `delete()` - delete (from Model)

### ❌ BAD method names (avoid)

#### Too general / unclear
- ❌ `process()` - what does it process?
- ❌ `do()` - what does it do?
- ❌ `handle()` - in controller (use only in Actions)
- ❌ `execute()` - what does it execute?
- ❌ `run()` - what does it run?
- ❌ `get()` - what does it get? (use only in Repositories for simple operations)

#### Too technical / implementation-specific
- ❌ `getData()` - what data?
- ❌ `processArray()` - what does it do with array?
- ❌ `buildResponse()` - what type of response?
- ❌ `executeQuery()` - what query?

#### Too short / abbreviations
- ❌ `ret()` - what does it return?
- ❌ `proc()` - what does it process?
- ❌ `fmt()` - what does it format?
- ❌ `norm()` - what does it normalize?

#### Too long / redundant
- ❌ `getMovieFromDatabaseBySlug()` - "FromDatabase" is redundant (Repository already implies this)
- ❌ `formatJsonResponseForMovieList()` - "JsonResponse" is redundant (Response Formatter already implies this)
- ❌ `executeMovieSearchQueryInDatabase()` - too detailed

### Naming Rules

#### 1. Use verbs for actions
- ✅ `retrieve`, `search`, `create`, `update`, `delete`, `format`, `normalize`
- ❌ `movie`, `data`, `result`, `info`

#### 2. Be specific, not general
- ✅ `findBySlug()` instead of `find()`
- ✅ `formatMovieList()` instead of `format()`
- ✅ `getSimilarMovies()` instead of `getMovies()`

#### 3. Use class context
- In `MovieRepository`: `findBySlug()` (not `findMovieBySlug()` - class already says it's Movie)
- In `MovieResponseFormatter`: `formatSuccess()` (not `formatMovieSuccess()` - class already says it's Movie)
- In `MovieService`: `retrieve()` (not `retrieveMovie()` - class already says it's Movie)

#### 4. Avoid negation in names
- ✅ `isValid()` instead of `isNotValid()`
- ✅ `hasDescription()` instead of `doesNotHaveDescription()`
- If you need negation, use `!` in code, not in name

#### 5. Use standard suffixes/prefixes
- `get*()` - get data (Services, Repositories)
- `find*()` - search (Repositories)
- `create*()` - create (Services, Repositories)
- `update*()` - update (Services, Repositories)
- `delete*()` - delete (Services, Repositories)
- `format*()` - format (Response Formatters)
- `normalize*()` - normalize data
- `validate*()` - validate
- `build*()` - build data structures
- `handle*()` - handle (only in Actions)

#### 6. Method names should match their responsibility
- **Controller:** `index()`, `show()`, `search()` - HTTP operations
- **Service:** `retrieveMovie()`, `getSimilarMovies()` - business logic
- **Repository:** `findBySlug()`, `searchMovies()` - data access
- **Action:** `handle()` - main business operation
- **Response Formatter:** `formatSuccess()`, `formatError()` - response formatting

### Project Examples

#### ✅ GOOD examples from MovieController
```php
public function index(Request $request): JsonResponse
public function search(SearchMovieRequest $request): JsonResponse
public function show(Request $request, string $slug): JsonResponse
public function related(Request $request, string $slug): JsonResponse
public function refresh(string $slug): JsonResponse
private function handleDisambiguationSelection(string $originalSlug, string $selectedSlug): JsonResponse
private function cacheKey(string $slug, ?string $descriptionId = null): string
private function normalizeDescriptionId(mixed $descriptionId): null|string|false
```

#### ✅ GOOD examples from Services
```php
// MovieRetrievalService
public function retrieveMovie(string $slug, ?string $descriptionId): MovieRetrievalResult
private function handleMovieNotFound(string $slug, ?string $descriptionId): MovieRetrievalResult
private function attemptToFindOrCreateMovieFromTmdb(string $slug, ?string $descriptionId, array $validation): MovieRetrievalResult
private function findSelectedDescription(Movie $movie, ?string $descriptionId): MovieDescription|null|false

// MovieSearchService
public function search(array $criteria): SearchResult
private function searchLocal(array $criteria): SearchResult
private function searchExternal(array $criteria): SearchResult
```

#### ✅ GOOD examples from Response Formatters
```php
// MovieResponseFormatter
public function formatSuccess(Movie $movie, string $slug, ?MovieDescription $selectedDescription = null): JsonResponse
public function formatError(string $errorMessage, int $statusCode, ?array $additionalData = null): JsonResponse
public function formatNotFound(): JsonResponse
public function formatRelatedMovies(Movie $movie, array $result): JsonResponse
public function formatGenerationQueued(array $generationResult): JsonResponse
```

#### ✅ GOOD examples from Actions
```php
// QueueMovieGenerationAction
public function handle(string $slug, ?float $confidence = null, ?Movie $existingMovie = null, ...): array
private function buildExistingJobResponse(string $slug, array $existingJob, ...): array
private function normalizeLocale(?string $locale): ?string
private function normalizeContextTag(?string $contextTag): ?string
```

### Checklist before naming a method

**Before naming a method, ask yourself:**

- [ ] Does the name clearly describe **what** the method does?
- [ ] Does the name use a **verb** (for actions)?
- [ ] Is the name **specific**, not general?
- [ ] Is the name not **too long** (> 50 characters)?
- [ ] Is the name not **too short** (< 5 characters, unless standard like `get()`)?
- [ ] Does the name **fit the class context** (doesn't repeat class name)?
- [ ] Does the name **not use negation** (use `!` in code)?
- [ ] Does the name **match the responsibility** (Controller vs Service vs Repository)?

**If answer to any question is NO, change the name!**

## Readable Conditions / Guard Clauses

**Principle:** Instead of long `if` conditions with multiple conditions, extract logic to helper methods with readable names.

### ❌ BAD: Long conditions in if

```php
// ❌ BAD: Hard to understand what we're checking
if ($user->isActive() && $user->hasPermission('edit') && $movie->isPublished() && !$movie->isDeleted() && $movie->getOwnerId() === $user->getId()) {
    // edit logic
}

// ❌ BAD: Complex conditions with negations
if (!empty($criteria['q']) && ($typeFilter === 'all' || $typeFilter === 'collection') && $movie->hasRelationships()) {
    // logic
}

// ❌ BAD: Conditions with multiple checks
if ($descriptionId !== null && $descriptionId !== '' && preg_match('/^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i', $descriptionId)) {
    // logic
}
```

### ✅ GOOD: Extracted to helper methods

```php
// ✅ GOOD: Readable condition with helper method
if ($this->canEditMovie($user, $movie)) {
    // edit logic
}

private function canEditMovie(User $user, Movie $movie): bool
{
    return $user->isActive()
        && $user->hasPermission('edit')
        && $movie->isPublished()
        && !$movie->isDeleted()
        && $movie->getOwnerId() === $user->getId();
}

// ✅ GOOD: Readable method names
if ($this->shouldSearchCollection($criteria, $typeFilter, $movie)) {
    // logic
}

private function shouldSearchCollection(array $criteria, string $typeFilter, Movie $movie): bool
{
    return !empty($criteria['q'])
        && ($typeFilter === 'all' || $typeFilter === 'collection')
        && $movie->hasRelationships();
}

// ✅ GOOD: Helper method with readable name
if ($this->isValidDescriptionId($descriptionId)) {
    // logic
}

private function isValidDescriptionId(?string $descriptionId): bool
{
    if ($descriptionId === null || $descriptionId === '') {
        return false;
    }

    return preg_match('/^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i', $descriptionId) === 1;
}
```

### Condition Extraction Rules

#### 1. Extract conditions when there are 3+ conditions
- ✅ `if ($this->isSomething())` - readable
- ❌ `if ($a && $b && $c && $d)` - hard to understand

#### 2. Use verbs for checking methods
- ✅ `isValid()`, `canEdit()`, `shouldProcess()`, `hasPermission()`, `isReady()`
- ❌ `valid()`, `edit()`, `process()`, `permission()`

#### 3. Use prefixes for check types
- `is*()` - check state/status (`isValid()`, `isPublished()`, `isActive()`)
- `can*()` - check permissions/capabilities (`canEdit()`, `canDelete()`, `canAccess()`)
- `has*()` - check possession (`hasPermission()`, `hasRelationships()`, `hasDescription()`)
- `should*()` - check if should execute (`shouldProcess()`, `shouldCache()`, `shouldRetry()`)
- `needs*()` - check if needs (`needsRefresh()`, `needsUpdate()`)

#### 4. Avoid negation in method names
- ✅ `if (!$this->isValid())` - readable
- ❌ `if ($this->isNotValid())` - less readable
- ✅ `if ($this->isInvalid())` - acceptable (but positive is better)

#### 5. Group related conditions
- If conditions relate to the same concept, extract them together
- Example: all conditions about user permissions → `canEditMovie()`

### Project Examples

#### ✅ GOOD: Guard clauses in controller

```php
public function show(Request $request, string $slug): JsonResponse
{
    $descriptionId = $this->normalizeDescriptionId($request->query('description_id'));
    if ($descriptionId === false) {
        return $this->responseFormatter->formatError('Invalid description_id parameter', 422);
    }

    // Guard clause - early return
    if ($selectedSlug = $request->query('slug')) {
        return $this->handleDisambiguationSelection($slug, (string) $selectedSlug);
    }

    // Rest of logic...
}
```

#### ✅ GOOD: Extracted conditions in Service

```php
// MovieRetrievalService
private function handleMovieNotFound(string $slug, ?string $descriptionId): MovieRetrievalResult
{
    if (!$this->isGenerationEnabled()) {
        return MovieRetrievalResult::notFound();
    }

    $validation = SlugValidator::validateMovieSlug($slug);
    if (!$this->isValidSlug($validation)) {
        return MovieRetrievalResult::invalidSlug($slug, $validation);
    }

    return $this->attemptToFindOrCreateMovieFromTmdb($slug, $descriptionId, $validation);
}

private function isGenerationEnabled(): bool
{
    return Feature::active('ai_description_generation');
}

private function isValidSlug(array $validation): bool
{
    return $validation['valid'] === true;
}
```

#### ✅ GOOD: Readable conditions in helper method

```php
// MovieController
private function normalizeDescriptionId(mixed $descriptionId): null|string|false
{
    if ($this->isEmpty($descriptionId)) {
        return null;
    }

    $descriptionId = (string) $descriptionId;

    if (!$this->isValidUuid($descriptionId)) {
        return false;
    }

    return $descriptionId;
}

private function isEmpty(mixed $value): bool
{
    return $value === null || $value === '';
}

private function isValidUuid(string $uuid): bool
{
    return preg_match('/^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i', $uuid) === 1;
}
```

### Guard Clauses Pattern

**Guard Clauses** - early returns for error/invalid state conditions:

```php
// ❌ BAD: Nested if-else
public function process(Request $request): JsonResponse
{
    if ($request->has('data')) {
        $data = $request->input('data');
        if (!empty($data)) {
            if ($this->isValid($data)) {
                // main logic here (deeply nested)
                return $this->handle($data);
            } else {
                return $this->formatError('Invalid data');
            }
        } else {
            return $this->formatError('Empty data');
        }
    } else {
        return $this->formatError('Missing data');
    }
}

// ✅ GOOD: Guard clauses - early returns
public function process(Request $request): JsonResponse
{
    if (!$request->has('data')) {
        return $this->formatError('Missing data');
    }

    $data = $request->input('data');
    if (empty($data)) {
        return $this->formatError('Empty data');
    }

    if (!$this->isValid($data)) {
        return $this->formatError('Invalid data');
    }

    // Main logic at the end (without nesting)
    return $this->handle($data);
}
```

### Checklist before writing a condition

**Before writing an `if` condition, ask yourself:**

- [ ] Does the condition have **3+ conditions** connected with `&&` or `||`? → Extract to method
- [ ] Is the condition **hard to understand** at first glance? → Extract to method
- [ ] Does the condition **repeat** in several places? → Extract to method
- [ ] Does the condition contain **negations** (`!`) that hinder readability? → Extract to method
- [ ] Does the condition **mix different concepts** (permissions + state + validation)? → Split into methods
- [ ] Can you use **guard clause** (early return) instead of nested if-else? → Refactor

**If answer to any question is YES, extract condition to helper method!**

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
