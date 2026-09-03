## coding-standards

> **Last Updated:** 2026-02-06

# BooztFrog — Coding Standards

**Version:** 1.0
**Last Updated:** 2026-02-06

This document defines the coding standards for the BooztFrog project. All code must follow these guidelines. Claude Code should reference this file when writing or reviewing code.

---

## 1. General Principles

- Write clean, readable, self-documenting code
- Prefer composition over inheritance
- Keep functions/methods short and single-purpose (max ~30 lines)
- Follow the Single Responsibility Principle — one class/component does one thing
- No commented-out code in commits
- No `console.log` or `dd()` in committed code (use proper logging)
- DRY (Don't Repeat Yourself) — extract shared logic into utilities, services, or hooks
- Fail early — validate inputs at the boundary, not deep inside logic
- All money values stored as integers (cents). Never use floats for currency.
- All primary keys are UUIDs. No auto-incrementing integers.

---

## 2. TypeScript / Next.js Standards

### 2.1 TypeScript

- **Strict mode enabled** — no exceptions
- **No `any` type** unless absolutely unavoidable (and must include a `// TODO: type this` comment)
- Use `interface` for object shapes and API responses
- Use `type` for unions, intersections, and utility types
- Export types/interfaces from dedicated `.types.ts` files per feature
- Use `as const` for constant objects and enums
- Prefer `unknown` over `any` when the type is genuinely unknown

```typescript
// ✅ Good
interface Business {
  id: string;
  name: string;
  slug: string;
  createdAt: string;
}

type PlatformType = 'google' | 'tripadvisor' | 'trustpilot' | 'facebook' | 'yelp' | 'custom';

// ❌ Bad
const business: any = await fetchBusiness(id);
```

### 2.2 Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Components | PascalCase | `ProductCard.tsx` |
| Hooks | camelCase with `use` prefix | `useAuth.ts` |
| Utilities | camelCase | `formatCurrency.ts` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_DEVICES_FREE_PLAN` |
| Types/Interfaces | PascalCase | `BusinessResponse` |
| API functions | camelCase verb-first | `fetchProducts()`, `createBusiness()` |
| Event handlers | camelCase with `handle` prefix | `handleSubmit`, `handleDelete` |
| Boolean variables | camelCase with `is`/`has`/`can` prefix | `isLoading`, `hasSubscription` |
| Files (non-components) | kebab-case | `format-currency.ts` |
| Translation keys | dot.notation.camelCase | `dashboard.analytics.totalScans` |

### 2.3 Component Structure

```typescript
// 1. Imports (external → internal → types → styles)
import { useState } from 'react';
import { useTranslations } from 'next-intl';
import { Button } from '@/components/ui/button';
import { useBusiness } from '@/hooks/useBusiness';
import type { Business } from '@/types/business';

// 2. Types (if component-specific and small)
interface BusinessCardProps {
  business: Business;
  onEdit: (id: string) => void;
}

// 3. Component
export function BusinessCard({ business, onEdit }: BusinessCardProps) {
  const t = useTranslations('dashboard.businesses');
  const [isExpanded, setIsExpanded] = useState(false);

  // 4. Handlers
  const handleEdit = () => {
    onEdit(business.id);
  };

  // 5. Render
  return (
    <div className="rounded-lg border p-4">
      <h3 className="text-lg font-semibold">{business.name}</h3>
      <Button onClick={handleEdit}>{t('edit')}</Button>
    </div>
  );
}
```

### 2.4 React / Next.js Rules

- **Server components by default.** Only add `'use client'` when you need interactivity, hooks, or browser APIs.
- **Colocate page-specific components** next to their page file. Move to `components/` only when reused.
- **Never call fetch or axios directly in components.** All API calls go through `lib/api.ts`.
- **Use React Query (TanStack Query)** for all server state in client components.
- **Use React Hook Form + Zod** for all forms.
- **All user-facing strings must use `useTranslations()`** — zero hardcoded text, including aria labels, placeholder text, and error messages.
- **Prefer named exports** over default exports (except for Next.js page/layout files which require default).
- **No prop drilling beyond 2 levels.** Use context, composition, or React Query instead.

### 2.5 Tailwind CSS Rules

- **No custom CSS files** unless absolutely necessary (e.g., third-party library overrides)
- **Use the design system colors** defined in `tailwind.config.ts` — never hardcode hex values in classNames
- **Responsive: mobile-first.** Start with mobile styles, add `sm:`, `md:`, `lg:` breakpoints
- **Max line length for className:** If a className string exceeds ~100 chars, use `cn()` utility with logical grouping:

```typescript
// ✅ Good — grouped logically
<div className={cn(
  'flex items-center gap-4 rounded-lg border p-4',
  'hover:border-primary hover:shadow-sm',
  'transition-all duration-200',
  isActive && 'border-primary bg-primary/5'
)} />

// ❌ Bad — one enormous string
<div className="flex items-center gap-4 rounded-lg border p-4 hover:border-primary hover:shadow-sm transition-all duration-200" />
```

### 2.6 API Client (`lib/api.ts`)

```typescript
// All API calls are centralized here with proper typing
// Every endpoint gets its own typed function

// ✅ Good
export async function fetchBusinesses(): Promise<Business[]> {
  const { data } = await api.get<ApiResponse<Business[]>>('/businesses');
  return data.data;
}

export async function createBusiness(payload: CreateBusinessPayload): Promise<Business> {
  const { data } = await api.post<ApiResponse<Business>>('/businesses', payload);
  return data.data;
}

// ❌ Bad — untyped, called directly in component
const res = await fetch(`${API_URL}/businesses`);
const data = await res.json();
```

### 2.7 Error Handling

- API errors must be caught and displayed to the user via toast notifications
- Use error boundaries for unexpected component errors
- Form validation errors must map to specific fields
- Network errors should show a retry option

---

## 3. PHP / Laravel Standards

### 3.1 General PHP

- Follow **PSR-12** coding standard
- Use **PHP 8.5** features: typed properties, enums, readonly, match expressions, named arguments, property hooks
- Use strict types: `declare(strict_types=1);` at the top of every file
- Use `final` on classes that shouldn't be extended
- Type all method parameters and return types — no untyped methods

### 3.2 Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Controllers | PascalCase, singular, suffix | `BusinessController` |
| Models | PascalCase, singular | `Business` |
| Form Requests | PascalCase, action prefix | `CreateBusinessRequest` |
| API Resources | PascalCase, suffix | `BusinessResource` |
| Services | PascalCase, suffix | `AnalyticsService` |
| Jobs | PascalCase, verb prefix | `LogScan`, `ProcessOrder` |
| Events | PascalCase, past tense | `OrderPlaced`, `DeviceScanned` |
| Policies | PascalCase, suffix | `BusinessPolicy` |
| Migrations | snake_case, descriptive | `create_businesses_table` |
| Config keys | snake_case | `booztfrog.max_free_devices` |
| Route names | dot.notation | `api.businesses.store` |
| Database columns | snake_case | `created_at`, `stripe_customer_id` |

### 3.3 Controller Structure

Controllers must be thin. They handle HTTP concerns only: receive request, call service, return response.

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Dashboard;

use App\Http\Controllers\Controller;
use App\Http\Requests\CreateBusinessRequest;
use App\Http\Requests\UpdateBusinessRequest;
use App\Http\Resources\BusinessResource;
use App\Models\Business;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Resources\Json\AnonymousResourceCollection;

final class BusinessController extends Controller
{
    // ✅ Good — thin controller
    public function index(): AnonymousResourceCollection
    {
        $businesses = auth()->user()
            ->businesses()
            ->withCount('devices', 'locations')
            ->latest()
            ->paginate(20);

        return BusinessResource::collection($businesses);
    }

    public function store(CreateBusinessRequest $request): JsonResponse
    {
        $business = auth()->user()
            ->businesses()
            ->create($request->validated());

        return BusinessResource::make($business)
            ->response()
            ->setStatusCode(201);
    }

    public function update(UpdateBusinessRequest $request, Business $business): BusinessResource
    {
        $this->authorize('update', $business);
        $business->update($request->validated());

        return BusinessResource::make($business->fresh());
    }

    public function destroy(Business $business): JsonResponse
    {
        $this->authorize('delete', $business);
        $business->delete();

        return response()->json(null, 204);
    }
}
```

### 3.4 Model Standards

```php
<?php

declare(strict_types=1);

namespace App\Models;

use App\Traits\HasUuid;
use App\Traits\HasTranslations;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;

final class Business extends Model
{
    use HasFactory, HasUuid, HasTranslations;

    // Always explicit — never use $guarded = []
    protected $fillable = [
        'name',
        'slug',
        'logo_url',
        'website',
        'phone',
        'address',
        'city',
        'country',
        'timezone',
        'language',
    ];

    protected $casts = [
        // JSONB columns cast to array
    ];

    // Relationships grouped together
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function locations(): HasMany
    {
        return $this->hasMany(Location::class);
    }

    public function devices(): HasMany
    {
        return $this->hasMany(Device::class);
    }

    // Scopes
    public function scopeActive($query)
    {
        return $query->where('is_active', true);
    }
}
```

### 3.5 Form Requests

Every create/update endpoint must use a Form Request. Never validate in controllers.

```php
<?php

declare(strict_types=1);

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

final class CreateBusinessRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true; // Auth handled by Sanctum middleware
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'slug' => ['required', 'string', 'max:255', Rule::unique('businesses')],
            'website' => ['nullable', 'url', 'max:500'],
            'phone' => ['nullable', 'string', 'max:50'],
            'country' => ['nullable', 'string', 'size:2'],
            'language' => ['nullable', 'string', 'max:5'],
        ];
    }
}
```

### 3.6 API Resources

All API responses must use Resources. Never return Eloquent models directly.

```php
<?php

declare(strict_types=1);

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

final class BusinessResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        $locale = $request->header('Accept-Language', 'en');

        return [
            'id' => $this->id,
            'name' => $this->name,
            'slug' => $this->slug,
            'logoUrl' => $this->logo_url,
            'website' => $this->website,
            'phone' => $this->phone,
            'address' => $this->address,
            'city' => $this->city,
            'country' => $this->country,
            'timezone' => $this->timezone,
            'language' => $this->language,
            // Include counts when loaded
            'devicesCount' => $this->whenCounted('devices'),
            'locationsCount' => $this->whenCounted('locations'),
            // Include relations when loaded
            'locations' => LocationResource::collection($this->whenLoaded('locations')),
            'devices' => DeviceResource::collection($this->whenLoaded('devices')),
            'createdAt' => $this->created_at->toISOString(),
            'updatedAt' => $this->updated_at->toISOString(),
        ];
    }
}
```

### 3.7 Services

Complex business logic goes in services. Controllers call services, services call models.

```php
<?php

declare(strict_types=1);

namespace App\Services;

final class DeviceService
{
    public function generateCode(): string
    {
        // 6-char alphanumeric, no ambiguous chars (0, O, l, 1, I)
        $chars = 'abcdefghjkmnpqrstuvwxyz23456789';
        do {
            $code = '';
            for ($i = 0; $i < 6; $i++) {
                $code .= $chars[random_int(0, strlen($chars) - 1)];
            }
        } while (Device::where('code', $code)->exists());

        return $code;
    }

    public function provisionDevices(Order $order): void
    {
        DB::transaction(function () use ($order) {
            foreach ($order->items as $item) {
                for ($i = 0; $i < $item->quantity; $i++) {
                    Device::create([
                        'order_item_id' => $item->id,
                        'code' => $this->generateCode(),
                        'device_type' => 'both',
                        'is_active' => true,
                    ]);
                }
            }
        });
    }
}
```

### 3.8 Jobs (Queued Work)

All async operations use Laravel Jobs with proper error handling.

```php
<?php

declare(strict_types=1);

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

final class LogScan implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;
    public int $backoff = 5;

    public function __construct(
        private readonly string $deviceId,
        private readonly string $ipAddress,
        private readonly string $userAgent,
        private readonly ?string $platformSelected,
    ) {}

    public function handle(GeoIpService $geoIp): void
    {
        $location = $geoIp->lookup($this->ipAddress);
        $parsed = $this->parseUserAgent($this->userAgent);

        Scan::create([
            'device_id' => $this->deviceId,
            'ip_address' => $this->ipAddress,
            'user_agent' => $this->userAgent,
            'device_type' => $parsed['deviceType'],
            'os' => $parsed['os'],
            'browser' => $parsed['browser'],
            'country' => $location?->country,
            'city' => $location?->city,
            'platform_selected' => $this->platformSelected,
        ]);
    }
}
```

### 3.9 Authorization (Policies)

```php
<?php

declare(strict_types=1);

namespace App\Policies;

use App\Models\Business;
use App\Models\User;

final class BusinessPolicy
{
    public function view(User $user, Business $business): bool
    {
        return $user->id === $business->user_id;
    }

    public function update(User $user, Business $business): bool
    {
        return $user->id === $business->user_id;
    }

    public function delete(User $user, Business $business): bool
    {
        return $user->id === $business->user_id;
    }
}
```

---

## 4. Database Standards

### 4.1 Migrations

- One migration per table or change
- Descriptive names: `create_businesses_table`, `add_timezone_to_businesses_table`
- Always include `down()` method for rollbacks
- Add indexes for all foreign keys and frequently queried columns
- Add composite indexes for common query patterns

```php
// ✅ Good
Schema::create('scans', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->uuid('device_id');
    $table->timestamp('scanned_at')->useCurrent();
    $table->inet('ip_address')->nullable();
    $table->string('country', 2)->nullable();

    $table->foreign('device_id')->references('id')->on('devices')->cascadeOnDelete();
    $table->index('device_id');
    $table->index('scanned_at');
    $table->index(['device_id', 'scanned_at']); // Composite for analytics queries
});
```

### 4.2 Query Patterns

```php
// ✅ Good — Eloquent for simple CRUD
$business = Business::where('user_id', $user->id)->findOrFail($id);

// ✅ Good — Query Builder for complex analytics
$stats = DB::table('scan_aggregates')
    ->where('business_id', $businessId)
    ->whereBetween('date', [$startDate, $endDate])
    ->selectRaw('SUM(total_scans) as scans, SUM(unique_visitors) as visitors')
    ->first();

// ✅ Good — Eager loading to prevent N+1
$businesses = Business::with(['locations', 'devices'])->where('user_id', $userId)->get();

// ❌ Bad — N+1 query problem
$businesses = Business::where('user_id', $userId)->get();
foreach ($businesses as $business) {
    echo $business->devices->count(); // Separate query per business!
}
```

### 4.3 Transactions

Use transactions for any operation that writes to multiple tables:

```php
DB::transaction(function () use ($order, $user) {
    $order->update(['status' => 'paid']);
    $this->deviceService->provisionDevices($order);
    $this->notificationService->sendOrderConfirmation($user, $order);
});
```

---

## 5. API Response Standards

### 5.1 Response Format

All API responses follow a consistent structure:

```json
// Single resource
{
  "data": {
    "id": "uuid",
    "name": "My Business",
    "createdAt": "2026-01-15T10:30:00.000Z"
  }
}

// Collection
{
  "data": [
    { "id": "uuid", "name": "Business 1" },
    { "id": "uuid", "name": "Business 2" }
  ],
  "meta": {
    "currentPage": 1,
    "lastPage": 5,
    "perPage": 20,
    "total": 100
  }
}

// Error
{
  "message": "The given data was invalid.",
  "errors": {
    "name": ["The name field is required."],
    "slug": ["The slug has already been taken."]
  }
}
```

### 5.2 HTTP Status Codes

| Code | Usage |
|------|-------|
| 200 | Successful GET, PUT |
| 201 | Successful POST (resource created) |
| 204 | Successful DELETE (no content) |
| 400 | Bad request (malformed input) |
| 401 | Unauthenticated |
| 403 | Forbidden (authenticated but not authorized) |
| 404 | Resource not found |
| 422 | Validation errors |
| 429 | Rate limited |
| 500 | Server error |

### 5.3 JSON Key Format

- API responses use **camelCase** keys (`createdAt`, `logoUrl`, `devicesCount`)
- Database columns use **snake_case** (`created_at`, `logo_url`)
- API Resources handle the transformation

---

## 6. i18n Standards

### 6.1 Translation File Structure

```json
// messages/en.json
{
  "common": {
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete",
    "edit": "Edit",
    "loading": "Loading...",
    "error": "Something went wrong",
    "success": "Success"
  },
  "nav": {
    "home": "Home",
    "products": "Products",
    "pricing": "Pricing",
    "login": "Log in",
    "signup": "Sign up"
  },
  "dashboard": {
    "sidebar": {
      "overview": "Overview",
      "analytics": "Analytics",
      "devices": "Devices"
    },
    "analytics": {
      "totalScans": "Total Scans",
      "uniqueVisitors": "Unique Visitors"
    }
  }
}
```

### 6.2 Translation Rules

- **Every user-facing string** must be in the translation file — no exceptions
- Use **dot notation** namespacing: `dashboard.analytics.totalScans`
- Use **ICU message syntax** for plurals and interpolation:
  ```json
  "devicesCount": "You have {count, plural, =0 {no devices} one {# device} other {# devices}}"
  ```
- Keep keys descriptive and hierarchical
- **English is the source of truth** — all other languages translate from `en.json`
- Never translate brand names: "BooztFrog", "Tap. Scan. Boozt."

### 6.3 Backend Translations

- API error messages and validation messages are translated via Laravel's `lang/` directory
- Translatable database content (product names, plan names) uses JSONB with locale keys
- Always fall back to `en` if the requested locale is missing:

```php
// HasTranslations trait
public function getTranslation(string $field, string $locale = 'en'): string
{
    $translations = $this->{$field};
    return $translations[$locale] ?? $translations['en'] ?? '';
}
```

---

## 7. Git Standards

### 7.1 Branch Naming

```
feature/add-analytics-dashboard
fix/stripe-webhook-retry
refactor/device-service-cleanup
chore/update-dependencies
```

### 7.2 Commit Messages

Use conventional commits:

```
feat: add scan analytics endpoint with date range filtering
fix: resolve race condition in device code generation
refactor: extract payment logic into StripeService
chore: update next-intl to v4
docs: add API endpoint documentation
test: add feature tests for business CRUD
```

### 7.3 Pull Request Rules

- One feature per PR
- All tests must pass
- No TypeScript errors or PHP static analysis issues
- Include description of what changed and why

---

## 8. Security Standards

- **Never expose internal IDs** in error messages
- **Never log sensitive data** (passwords, tokens, full credit card numbers)
- **Always validate and sanitize input** via Form Requests (Laravel) and Zod (Next.js)
- **Use parameterized queries** — never concatenate user input into SQL
- **Scope all data access** by authenticated user — verify ownership in Policies
- **Rate limit all endpoints** — stricter limits on auth endpoints (login, register)
- **CORS**: Only allow `booztfrog.com` origin on the API
- **Stripe webhooks**: Always verify webhook signatures
- **Device codes**: Non-sequential, non-guessable — prevent enumeration
- **File uploads**: Validate file type and size, store in S3 (never on local disk in production)
- **Environment variables**: Never commit `.env` files. Use `.env.example` as template.

---

## 9. Performance Standards

### Frontend
- **Lighthouse score target:** 90+ on all metrics
- **Use Next.js Image component** for all images (automatic optimization)
- **Lazy load** below-the-fold components with `dynamic()` imports
- **Minimize client-side JavaScript** — prefer server components
- **Cache API responses** with React Query stale times (30s for dashboard data, 5min for products)

### Backend
- **N+1 prevention:** Always eager load relations with `with()` or `withCount()`
- **Redis caching:** Cache device lookups, product catalog, subscription plans
- **Database indexes:** Every foreign key and every column used in WHERE/ORDER BY
- **Queue heavy work:** Emails, manufacturer API calls, GeoIP lookups, analytics aggregation
- **NFC redirect target:** < 100ms response time

---

## 10. Testing Standards

### Backend (PHPUnit)

- **Feature tests** for every API endpoint (happy path + error cases)
- **Unit tests** for services with complex logic
- **Use factories** for test data — never hardcode IDs or values
- **Test authorization** — verify users can't access other users' data
- **Test validation** — verify all required fields and constraints

```php
public function test_user_can_only_view_own_businesses(): void
{
    $user = User::factory()->create();
    $otherUser = User::factory()->create();
    $business = Business::factory()->for($otherUser)->create();

    $this->actingAs($user)
        ->getJson("/api/businesses/{$business->id}")
        ->assertForbidden();
}
```

### Frontend (Vitest + Playwright)

- **Unit tests** for utility functions and hooks
- **Component tests** for complex interactive components
- **E2E tests** (Playwright) for critical user flows: register → login → create business → view dashboard
- **Test i18n** — verify key pages render in at least 2 locales

---
> Source: [DonCrypto78/booztfrog.com](https://github.com/DonCrypto78/booztfrog.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
