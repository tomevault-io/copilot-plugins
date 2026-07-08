## mealo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
composer run dev                     # Start Laravel server, queue, Pail logs, and Vite concurrently

# PHP tests (SQLite in-memory, RefreshDatabase on every test)
composer run test                    # All tests (clears config cache first)
composer run test:feature            # Feature suite only (parallel)
composer run test:integration        # Integration suite only (parallel)
composer run test:unit               # Unit suite only (parallel)
composer run test:types              # PHPStan level 10 (covers app/ + tests/)
php artisan test tests/Feature/PlannedMeal/PlannedMealStoreTest.php
php artisan test --filter "stores a planned meal"

# Frontend checks
pnpm types        # TypeScript type check
pnpm lint         # ESLint with auto-fix
pnpm format       # Prettier

# Code quality
composer run lint                    # Laravel Pint (PHP auto-formatter)

# Code generation — run after changes
php artisan wayfinder:generate       # Regenerate resources/js/actions/ after route changes
php artisan typescript:transform     # Regenerate TS types after #[TypeScript] Data class changes
```

## Stack

- **Backend**: Laravel 12, PHP 8.2+, Inertia.js, Pest
- **Frontend**: React 19, TypeScript, Tailwind CSS v4, Radix UI, DaisyUI, pnpm
- **Auth**: Laravel Fortify
- **Permissions**: Spatie Laravel Permission (team-scoped roles)
- **Data/DTOs**: Spatie Laravel Data + TypeScript Transformer
- **Routes → TS**: Laravel Wayfinder
- **AI**: OpenAI PHP client — recipe generation uses `gpt-4o-mini`, meal plan generation uses `gemini-3-flash-preview` (both via `app('openai.client')`)

---

## Backend

### Request/Response pipeline

```
HTTP Request
  → Controller (thin: resolves deps, calls action, returns Inertia/redirect)
    → RequestData DTO (Spatie Data, validates via rules())
    → Action::execute(User, ...args) or Action::__invoke(...)
      → Model / DB / Service
    → ResourceData DTO (Spatie Data, passed to Inertia::render())
```

Controllers use the `HasAuthenticatedUser` trait — always call `$this->authenticatedUser()` instead of `auth()->user()`.

### Directory structure

```
app/
  Actions/
    PlannedMeal/   # PlannedMealStoreAction, PlannedMealDestroyAction, PlannedMealWeekQueryAction,
                   # PlannedMealUpdateAction, PlannedMealGeneratePlanAction
    Recipes/       # RecipeStoreAction, RecipeUpdateAction, RecipeDestroyAction,
                   # RecipeAIGenerationAction, RecipeImageAIGenerationAction,
                   # RecipeUploadImageAction, RecipeImageDeleteAction,
                   # RecipeSyncIngredientsAction, RecipeSyncTagsAction,
                   # RecipeSyncMealTimesAction, RecipeSyncStepsAction,
                   # RecipeFiltersAction, RecipeSearchAction, ...
    Workspace/     # WorkspaceStoreAction, WorkspaceUpdateAction, WorkspaceDeleteAction,
                   # WorkspaceGetCurrentAction, WorkspaceInvitationStoreAction,
                   # WorkspaceInvitationAcceptAction, WorkspaceInvitationDeclineAction,
                   # WorkspaceMemberDeleteAction, WorkspaceMemberRoleUpdateAction
  Data/
    Requests/      # Incoming DTOs with rules() for validation
    Resources/     # Outgoing DTOs sent to Inertia (many marked #[TypeScript])
  Enums/
    Unit           # Ingredient units (ml, g, tsp, piece, etc.) — used by AI prompts
  Http/Controllers/
  Models/
  Observers/       # RecipeObserver (deletes image + planned meals on delete)
                   # PlannedMealObserver (syncs shopping lists)
  Policies/        # Bound via #[UsePolicy] attribute on models
  Services/
    AIMealPlanningService   # Generates weekly meal plans via AI function calling
    ShoppingListService
```

### Models & domain

**Key relationships:**

```
User ──< Recipe (user_id, UUID PK)
          ├── BelongsToMany MealTime  (via recipe_meal_time)
          ├── BelongsToMany Ingredient (via recipe_ingredient, pivot: quantity, unit)
          ├── HasMany Step
          └── BelongsToMany Tag (via recipe_tag)

User >─< Workspace (via workspace_users, pivot: joined_at)
Workspace ──< PlannedMeal (workspace_id + user_id + recipe_id + meal_time_id + planned_date)
PlannedMeal ──< ShoppingListPlannedMealIngredient

Workspace ──< WorkspaceInvitation (expires_at, token)
```

**Conventions:**

- `Recipe` uses `HasUuids` (UUID primary key); all other models use auto-increment int IDs.
- Policies are bound with `#[UsePolicy(XxxPolicy::class)]` on the model, not in `AuthServiceProvider`.
- Observers are registered with `#[ObservedBy([XxxObserver::class])]` on the model.
- All datetime columns cast to `immutable_datetime`.

A `PlannedMeal` stores **both** `user_id` (who planned it) and `workspace_id` (the shared context). Storing a duplicate (same user + workspace + recipe + meal_time + date) increments `serving_size` instead of creating a new row.

### Workspaces & permissions

The active workspace is stored in session (`current_workspace_id`). `WorkspaceGetCurrentAction` reads it and falls back to the user's personal default workspace.

**Always call `setPermissionsTeamId($workspace->id)` before any Spatie permission check** — roles are team-scoped. Every new user gets a personal workspace via `Workspace::createPersonalWorkspace()`. The `Workspace::booted()` hook auto-assigns the `owner` role to the creator.

| Permission                        | owner | editor | viewer |
|-----------------------------------|-------|--------|--------|
| `workspace.view`                  | ✓     | ✓      | ✓      |
| `workspace.edit`                  | ✓     |        |        |
| `workspace.manage`                | ✓     |        |        |
| `workspace.planned-meal.store`    | ✓     | ✓      |        |
| `workspace.planned-meal.update`   | ✓     | ✓      |        |
| `workspace.planned-meal.view`     | ✓     | ✓      |        |
| `workspace.planned-meal.destroy`  | ✓     | ✓      |        |
| `planning.edit`                   | ✓     | ✓      |        |

### Spatie Laravel Data (DTOs)

- **Request DTOs** (`app/Data/Requests/`): extend `Data`, validate in `static rules(): array`.
- **Resource DTOs** (`app/Data/Resources/`): extend `Data`, provide `static fromModel(Model): self`. Pass to Inertia with `::collect()` for collections.
- `#[TypeScript]` triggers TS type generation. `#[Optional]` marks nullable props. `#[LiteralTypeScriptType('File|null')]` overrides the inferred type (used for `UploadedFile`).
- After modifying any `#[TypeScript]` class, run `php artisan typescript:transform`.

### AI features

`RecipeAIGenerationAction` calls `gpt-4o-mini` with function calling to generate a complete `RecipeStoreRequestData` from a text prompt. The function schema enforces valid `Unit` enum values and correct `MealTime` IDs.

`AIMealPlanningService` / `PlannedMealGeneratePlanAction` calls `gemini-3-flash-preview` with function calling. It samples up to 25 random recipes, groups them by meal time (breakfast/lunch/dinner/snack), and asks the AI to produce a plan for N days. The action deletes existing planned meals for the period inside a DB transaction before inserting the new ones.

### File storage

Recipe images use the `recipe_images` disk (`storage/app/private/recipe_images/`, local driver, served via `RecipeController::image()`). Image URLs include `?v={timestamp}` for cache-busting. Use `Storage::fake('recipe_images')` in tests.

---

## Frontend

### Structure

```
resources/js/
  actions/          # Auto-generated Wayfinder route helpers (DO NOT EDIT)
  routes/           # Domain re-exports of Wayfinder actions (import recipesRoute from '@/routes/recipes')
  pages/            # Inertia entry points — thin, each delegates to a feature adapter + view
  types/
    index.d.ts      # Manual types (SharedData, PaginatedCollection, PlannedMeal, Workspace, …)
    generated.d.ts  # Auto-generated from #[TypeScript] PHP classes (DO NOT EDIT)
    enum.ts         # TypeScript enums mirroring PHP enums
  app/
    features/       # Domain modules: recipes/, planned-meals/, workspaces/, shopping-lists/, admin/
    components/     # Shared UI components; ui/ sub-folder holds primitives (Button, Input, Dialog, …)
    hooks/          # Shared hooks (usePermissions, useAppForm, useWeekSelector, …)
    stores/         # Shared Zustand stores
    layouts/        # AppLayout, AuthLayout, AdminLayout, SettingsLayout
    lib/            # i18n setup (i18n.ts)
    data/           # Frontend data types/schemas grouped by domain
    locales/        # i18n translation files
```

### Feature module pattern

Every domain feature follows this structure:

```
app/features/recipes/
  inertia.adapter.tsx   # Extracts Inertia props → typed React context
  views/                # View components: index.recipes.view.tsx, create.recipes.view.tsx, …
  components/           # Feature-specific UI components
  hooks/                # Feature-specific hooks
  stores/               # Feature-specific Zustand stores
```

The **Inertia Adapter** extracts `usePage().props`, wraps them in a typed context via `createGenericContext<PageProps>()`, and exposes them via a domain hook (e.g. `useRecipesContextValue()`). The page file is just:

```tsx
export default function RecipesIndex() {
  return <RecipesInertiaAdapter><IndexRecipesView /></RecipesInertiaAdapter>;
}
```

Views consume context via `useRecipesContextValue()` and **never call `usePage()` directly**.

### Forms

Forms use **TanStack Form** via the custom `useAppForm` hook (`app/hooks/form-hook.ts`), which pre-wires all field components (`TextField`, `NumberField`, `SelectField`, `MultiSelectField`, `TextAreaField`, etc.) and `SubmitButton`. Always use `useAppForm` — never raw TanStack Form APIs.

### State, routing & i18n

- **State**: Zustand stores. Feature stores in `features/{domain}/stores/`, shared stores in `app/stores/`. The planned-meals page polls with `usePoll(3000, { only: ['plannedMeals'] })`.
- **Routing**: Import from `resources/js/routes/` (e.g. `import recipesRoute from '@/routes/recipes'`). Never hardcode URLs.
- **i18n**: `react-i18next` — use `useTranslation()` + `t('key')`. Translations in `app/locales/`.
- **Pagination**: Backend sends `Inertia::scroll()` responses; frontend uses Inertia's `<InfiniteScroll>` with `PaginatedCollection<T>` props.
- **Shared props** (`SharedData`): `auth.user`, `flash` (success/error/warning/message), `sidebarOpen` — available on every page via `usePage<SharedData>().props`.

---

## Testing

### Test types

All suites extend `Tests\TestCase` (SQLite in-memory, `RefreshDatabase`, seeds `RolesAndPermissionsSeeder` + `MealTimeSeeder` before every test).

| Suite | Layer | How to assert |
|---|---|---|
| `tests/Feature/` | HTTP | `actingAs($user)->post(route(...), $dto->transform())` → `assertSessionHas('success', ...)` |
| `tests/Integration/` | Action | `app(XxxAction::class)->execute(...)` → `assertDatabaseHas()` |
| `tests/Unit/` | Pure logic | No DB |

For workspace-scoped Feature tests, pass `->withSession(['current_workspace_id' => $workspace->id])`.

### Fixtures

Four context traits (`HasUserContext`, `HasWorkspaceContext`, `HasRecipeContext`, `HasPlannedMealContext`) build a full fixture graph in `setUp()`:

| Property | Description |
|---|---|
| `$this->user` | Primary test user (owner of `defaultWorkspace`) |
| `$this->otherUser` / `$this->thirdUser` | Unrelated users |
| `$this->editorUser` | Has `editor` role in `sharedWorkspace` |
| `$this->viewerUser` | Has `viewer` role in `sharedWorkspace` |
| `$this->inviteeUser` / `$this->otherInviteeUser` | Users with pending invitations |
| `$this->defaultWorkspace` | `$this->user`'s personal default workspace |
| `$this->sharedWorkspace` | Shared workspace owned by `$this->user` (editor + viewer members) |
| `$this->personalWorkspace` | Additional personal workspace for `$this->user` |
| `$this->recipe` / `$this->otherRecipe` | Recipes belonging to `$this->user` |
| `$this->editorRecipe` / `$this->viewerRecipe` | Recipes belonging to editor/viewer users |
| `$this->mealTime` | First `MealTime` from the seeded list |
| `$this->userPlannedMealStoreRequestData` | Valid store DTO for `$this->user` |
| `$this->viewerPlannedMealStoreRequestData` | Store DTO using viewer's recipe |
| `$this->userPlannedMealUpdateRequestData` | Valid update DTO |
| `$this->sharedWorkspaceInvitation` | Active invitation for `$this->inviteeUser` |
| `$this->sharedWorkspaceExpiredInvitation` | Expired invitation |

**Global Pest helpers** (`tests/Pest.php`):

- `createUserWithWorkspace(array $attributes = []): User` — creates user + personal workspace
- `recipeResourceFor(User $user, array $attributes = []): RecipeResource` — full recipe with all relations
- `createRecipeFor(User $user, array $attributes = []): Recipe` — minimal recipe, no relations

---
> Source: [lionelp-dev/Mealo](https://github.com/lionelp-dev/Mealo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
