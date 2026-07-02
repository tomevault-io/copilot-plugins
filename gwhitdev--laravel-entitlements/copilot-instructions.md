## laravel-entitlements

> This file tells an AI assistant how to gate a feature correctly using this package. Read it before writing any entitlement check, middleware, seeder, or plan mapping.

# Agent Instructions — Entitlements for Laravel

This file tells an AI assistant how to gate a feature correctly using this package. Read it before writing any entitlement check, middleware, seeder, or plan mapping.

---

## What this package does (and doesn't do)

**One job:** given a user and a feature key, answer "is this user entitled to it?"

It is **not** a feature-flag toggle system (that's Pennant). It is **not** a billing engine (that's Cashier). It sits between them: Cashier tells you who pays, Entitlements tells you what paying unlocks.

---

## Core concepts

### Feature key
A `snake_case` string that identifies a capability — e.g. `advanced_reporting`. Defined in your feature enum. Always a string at the boundary; backed enum cases are accepted as sugar and normalised internally.

### Resolution cascade (first match wins)
1. **Admin override** — if `admin_override` is enabled in config and `$user->isEntitlementAdmin()` returns `true` (or `$user->is_admin` is truthy as a fallback), the user is entitled to everything.
2. **Per-user grant** — a row in `user_features` for this user + key grants access regardless of plan.
3. **Membership gate** — if the user has no active subscription (`PlanResolver::isActive()` returns `false`), resolution stops here and returns `false`.
4. **Plan mapping** — the user's plan identifier (e.g. Stripe price id) is looked up in `plan_features` (or `config/entitlements.php` if using the config store); if the key is in that plan's set, access is granted.

Results are **memoized per request**, so calling `hasFeature()` multiple times is safe.

---

## Checking entitlement

Prefer the trait method on the User model. Use the facade when `$user` is resolved manually. All three forms are equivalent.

```php
// Trait (most common)
$user->hasFeature('advanced_reporting');        // bool
$user->features();                              // string[] — every key the user holds
$user->grantFeature('advanced_reporting');      // direct grant (trials, comps, etc.)

// Facade
use Entitlements\Facades\Tessera;
Tessera::has($user, 'advanced_reporting');      // bool
Tessera::entitlements($user);                   // string[]

// Route middleware — aborts 403 if the authed user isn't entitled
Route::get('/reports', ReportController::class)
    ->middleware('feature:advanced_reporting');

// Blade
@feature('advanced_reporting')
    <a href="/reports">Advanced reporting</a>
@endfeature
```

**Accepted key forms** — both are valid everywhere:
```php
$user->hasFeature('advanced_reporting');        // string key
$user->hasFeature(Feature::AdvancedReporting);  // backed enum case
```

---

## Defining features

Features live in a backed enum at `app/Enums/Feature.php`. Scaffold a new case with:

```bash
php artisan entitlements:make AdvancedReporting
```

The enum case value is the string key used everywhere else:

```php
enum Feature: string
{
    case AdvancedReporting = 'advanced_reporting';
}
```

If a feature should only be available when another is also in the plan, declare it as a dependency on the enum case attribute. `entitlements:lint` will catch mappings that violate these:

```php
use Entitlements\Attributes\DependsOn;

#[DependsOn('basic_reporting')]
case AdvancedReporting = 'advanced_reporting';
```

---

## Mapping plans to features

### Config store (code-managed, best for simple setups)

In `config/entitlements.php`:

```php
'plan_store' => 'config',
'plans' => [
    'price_pro_monthly'    => ['advanced_reporting', 'team_seats'],
    'price_starter_monthly' => ['basic_reporting'],
],
```

Keys are opaque plan identifiers — for Stripe these are price ids (e.g. `price_pro_monthly`). They are stored verbatim; never assume a specific format.

### Database store (runtime-editable, default)

```php
use Entitlements\Models\PlanFeature;

PlanFeature::grant('price_pro_monthly', 'advanced_reporting');
PlanFeature::grant('price_pro_monthly', Feature::TeamSeats);   // enum case also accepted
```

`grant()` is idempotent — safe to call in seeders.

---

## Granting features directly to a user

Direct grants sit above the plan in the cascade — they survive plan downgrades and don't require an active subscription:

```php
$user->grantFeature('advanced_reporting');
// or
UserFeature::grant($user, 'advanced_reporting');
```

Never write to `user_features` via mass assignment (`UserFeature::create([...])`) — the model is fully guarded by design.

---

## Swappable seams

All three seams are bound from `config/entitlements.php` and resolved through the container. Swap any of them without touching the check API.

| Seam | Contract | Default | When to swap |
|------|----------|---------|--------------|
| Billing | `PlanResolver` | `StripePlanResolver` | Using Paddle, Lemon Squeezy, or a custom billing system |
| Feature catalog | `FeatureCatalog` | `EnumFeatureCatalog` | Using config array (`ConfigFeatureCatalog`) or a DB-backed catalog |
| Gate | `FeatureGate` | `CascadingFeatureGate` | Overriding resolution logic entirely |

To swap, point the config key at your implementation class:

```php
'resolver' => \App\Entitlements\PaddlePlanResolver::class,
```

Your class must implement the corresponding contract. `PlanResolver` requires `planIdentifier(Authenticatable): ?string` and `isActive(Authenticatable): bool`.

---

## Admin override

Off by default (fail-closed). When enabled, an entitled admin bypasses the cascade entirely:

```php
// config/entitlements.php
'admin_override' => true,
```

Prefer defining `isEntitlementAdmin()` on your User model over relying on a raw `is_admin` column — make admin status an explicit, non-mass-assignable decision:

```php
public function isEntitlementAdmin(): bool
{
    return $this->role === 'super_admin';
}
```

---

## Pennant bridge (optional)

If the codebase uses Laravel Pennant and you want existing `Feature::` check sites to keep working, call this once in `AppServiceProvider::boot()`:

```php
use Entitlements\Bridge\PennantBridge;

PennantBridge::register();
```

This iterates the catalog and registers every feature key with Pennant, delegating resolution to the entitlement cascade. After calling it:

```php
Feature::for($user)->active('advanced_reporting'); // same as $user->hasFeature(...)
```

**Do not** use `Feature::define()` manually for feature keys that are already in the catalog — that would create two conflicting definitions. `PennantBridge::register()` handles all of them in one call.

Pennant's own storage and caching are bypassed — resolution always hits the cascade, so per-user grants and plan changes are reflected without any Pennant cache flush.

---

## Common mistakes

**Don't check the plan directly** — never read `$user->subscription()->stripe_price` and branch on it. Always go through `hasFeature()`. The cascade handles grants, admin overrides, and inactive subscriptions for you.

**Don't write to `user_features` via `create()`** — use `UserFeature::grant()` or `$user->grantFeature()`. The model is fully guarded.

**Don't use string literals for keys in more than one place** — define them on the enum and reference `Feature::AdvancedReporting->value` or pass the enum case directly.

**Don't call `flush()` manually in normal request code** — the gate is scoped to the request container, so memoization resets automatically. Only call `flush()` in long-lived processes (Octane, queues) after a grant or plan change within the same process.

**Run the linter after changing plan mappings:**

```bash
php artisan entitlements:lint
```

This catches plans that include a feature without including its declared prerequisite.

---
> Source: [gwhitdev/laravel-entitlements](https://github.com/gwhitdev/laravel-entitlements) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
