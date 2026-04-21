## yoryor-last

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: YorYor Dating Application

A comprehensive Muslim dating and matchmaking platform built with Laravel 12, emphasizing cultural values, family involvement, and serious relationships. The platform modernizes traditional matchmaking while respecting Islamic cultural and religious values.

**Tech Stack:** Laravel 12, PHP 8.2+, Tailwind CSS 4.0, Laravel Reverb (WebSocket), Laravel Sanctum (API), PostgreSQL

**Core Purpose:** Islamic/Cultural Dating & Matchmaking with family involvement and serious commitment to marriage.

## Essential Commands

### Development Server
```bash
# Start all services (Laravel server + queue worker + logs + Vite)
composer dev

# Individual services
php artisan serve                    # Laravel server (port 8000)
php artisan reverb:start            # WebSocket server (port 8080)
php artisan queue:listen --tries=1   # Queue worker
php artisan pail --timeout=0        # Real-time logs
npm run dev                          # Vite dev server
```

### Database
```bash
# Run migrations
php artisan migrate

# Fresh database with seeders
php artisan migrate:fresh --seed

# Create new migration
php artisan make:migration create_table_name

# Rollback last migration
php artisan migrate:rollback
```

### Testing
```bash
# Run all tests (Pest)
php artisan test

# Run specific test file
php artisan test tests/Feature/AuthTest.php

# Run tests with coverage
php artisan test --coverage

# Code formatting (Laravel Pint)
./vendor/bin/pint
```

### Cache & Optimization
```bash
# Clear all caches
php artisan optimize:clear

# Production optimization
php artisan optimize
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

### Monitoring Tools
```bash
# Access Laravel Telescope (development debugging)
# http://localhost:8000/telescope

# Access Laravel Pulse (performance monitoring)
# http://localhost:8000/pulse

# Access Laravel Horizon (queue monitoring)
php artisan horizon
# http://localhost:8000/horizon

# Access API documentation (Swagger)
# http://localhost:8000/api/documentation
```

## Architecture Overview

### Layered Architecture Pattern

**YorYor follows a strict layered architecture:**

1. **Presentation Layer**: Blade views + API Resources (JSON:API format)
2. **Application Layer**: Controllers (API + Web) orchestrate requests
3. **Service Layer**: Business logic in dedicated service classes (25+ services)
4. **Domain Layer**: Eloquent models (55+ models) with relationships
5. **Data Access Layer**: Database, Cache, Queue

### Key Architectural Principles

**Service Layer Pattern:**
- **All business logic lives in services**, not controllers
- Controllers validate input, call services, return responses
- Services are injected via dependency injection
- Example: `AuthService`, `MediaUploadService`, `VerificationService`

**Example Service Usage:**
```php
// Controller only orchestrates
public function __construct(
    private AuthService $authService,
    private NotificationService $notificationService
) {}

public function register(Request $request) {
    $validated = $request->validate([...]);
    $result = $this->authService->register($validated);
    $this->notificationService->sendWelcomeEmail($result['user']);
    return response()->json($result);
}
```

**Real-Time with Laravel Reverb:**
- WebSocket server runs on port 8080 (separate from Laravel)
- Broadcasting uses private channels: `private-chat.{chatId}`, `private-user.{userId}`
- Echo client configured in `resources/js/echo.js`
- Events must implement `ShouldBroadcast` interface

### Critical Architecture Rules

1. **Never put business logic in controllers** - always use services
2. **Never query models directly in views** - pass data from controllers
3. **Always use transactions for multi-model operations** - wrap in `DB::beginTransaction()`
4. **Route organization**:
   - `routes/api.php` - Mobile/API endpoints (Sanctum auth)
   - `routes/web.php` - Landing pages (public)
   - `routes/user.php` - Authenticated user routes (web middleware)
   - `routes/admin.php` - Admin dashboard routes (admin middleware)

## Database Architecture

### 70+ Tables Organized by Domain

**Database:** PostgreSQL (default connection configured)

**Core Pattern: User → Profile → Extended Profiles**
- `users` (authentication, basic info - password varchar(255) for future-proof hashing)
- `profiles` (core profile data with soft deletes)
- Extended profiles: `user_cultural_profiles`, `user_career_profiles`, `user_physical_profiles`, `user_family_preferences`, `user_location_preferences`

**Communication: Chat → Messages → Reads**
- `chats` (conversations)
- `chat_users` (pivot: participants with indexes)
- `messages` (chat messages)
- `message_reads` (read receipts)
- `calls` (video/voice calls linked to messages)

**Matching: Likes → Matches**
- `likes` (one-way interest)
- `dislikes` (pass/reject)
- `matches` (mutual likes - created automatically, with soft deletes)

**Reporting System:**
- `user_reports` (comprehensive reporting with severity, priority, evidence)
- `report_evidence` (file attachments for reports)
- `report_categories` (report categorization)

**Key Model Names:**
- `UserMatch` model → `matches` table (renamed from `Match` to avoid PHP 8+ reserved keyword)
- `UserReport` model → `user_reports` table (consolidated from duplicate tables)
- All pivot tables have proper indexes for reverse lookups

**Key Relationships:**
```php
User hasOne Profile
User hasOne UserPreference
User hasMany UserPhoto
User belongsToMany Chat (via chat_users)
User hasMany UserMatch (self-referencing via user_id/matched_user_id)
Chat hasMany Message
Message belongsTo Call
UserReport belongsTo User (reporter_id)
UserReport belongsTo User (reported_user_id)
UserReport hasMany ReportEvidence
```

**Soft Deletes Enabled:**
- `profiles` - Data retention for deleted profiles
- `matches` - Track match history even after deletion
- `user_blocks` - Audit trail for blocking history
- `user_photos` - Photo history preservation

### Migration Order Matters
- Migrations run chronologically by filename timestamp (2025_09_24_* to 2025_09_27_*)
- Foreign keys added last: `2025_09_24_999999_add_foreign_key_constraints.php`
- Never modify existing migrations after running in production
- Removed duplicate migrations: `create_user_reports_table` (old basic version), `create_pulse_tables` (empty placeholder)

## Authentication & Authorization

### Multi-Factor Authentication Flow
1. User enters email/phone + password
2. `AuthController::authenticate()` generates OTP via `OtpService`
3. OTP sent via email/SMS
4. User verifies OTP
5. If 2FA enabled: verify Google Authenticator code (`TwoFactorAuthService`)
6. Generate Sanctum token
7. Return user + token

### Authorization Layers
1. **Middleware**: `AdminMiddleware`, custom `Authenticate` (redirects to `/start` not `/login`)
2. **Policies**: Gate checks in controllers (`$this->authorize('update', $profile)`)
3. **RBAC**: `role_user`, `permission_role` pivot tables
4. **Scopes**: Model scopes filter queries (`User::active()`, `User::online()`)

### Custom Middleware
- `ApiRateLimit`: Dynamic rate limiting per endpoint type (15+ action types)
- `ChatRateLimit`: Separate limits for chat operations (create, send, read, edit, delete)
- `SecurityHeaders`: Inject security headers (CSP, HSTS, X-Frame-Options)
- `UpdateLastActive`: Track user activity for presence
- `PerformanceMonitor`: Log slow requests (dev only, disabled in production)
- `LanguageMiddleware`: Locale detection from session/header
- `SetLocale`: Set app locale for translations (supports: en, uz, ru)

## Real-Time Features

### WebSocket Architecture
```
Laravel App (port 8000) → Broadcasts Event → Reverb (port 8080) → Pushes to Clients
```

### Broadcasting Pattern
```php
// 1. Create event
class NewMessageEvent implements ShouldBroadcast {
    public function broadcastOn(): array {
        return [new PrivateChannel("chat.{$this->chatId}")];
    }
}

// 2. Dispatch event
event(new NewMessageEvent($message));

// 3. Listen in JavaScript (resources/js/messages.js)
Echo.private(`chat.${chatId}`)
    .listen('NewMessageEvent', (e) => {
        // Update UI
    });
```

### Channel Authentication
- Private channels require authentication: `routes/channels.php`
- User must be chat participant to join `private-chat.{chatId}`
- Presence channels track online users

## API Design

### JSON:API Standard
- All API responses use JSON:API format via API Resources
- Structure: `{ type, id, attributes, relationships, included }`
- Resources in `app/Http/Resources/`: `UserResource`, `MessageResource`, `MatchResource` (uses `UserMatch` model)

### API Versioning
- All endpoints prefixed: `/api/v1/`
- Future versions: `/api/v2/` (maintain backwards compatibility)

### Rate Limiting Strategy
Different limits per action type (enforced at middleware level):
- `auth_action`: 10/min (login, register, check email)
- `like_action`: 100/hour (likes, dislikes, matches)
- `message_action`: 500/hour (send messages)
- `call_action`: 50/hour (initiate, join, end calls)
- `panic_activation`: 5/day (emergency button)
- `profile_update`: 30/hour (profile modifications)
- `block_action`: 20/hour (block/unblock users)
- `report_action`: 10/hour (report users)
- `verification_submit`: 3/day (verification requests)
- `password_change`: 5/hour
- `email_change`: 3/day
- `account_deletion`: 1/day
- `data_export`: 2/week
- `location_update`: 100/hour
- `story_action`: 20/day (create/delete stories)

Applied via middleware: `->middleware('api.rate.limit:action_type')`

**Chat-specific rate limits** (via `ChatRateLimit` middleware):
- `create_chat`: 50/hour
- `send_message`: 500/hour
- `mark_read`: 1000/hour
- `edit_message`: 100/hour
- `delete_message`: 100/hour

## Service Layer Details

### Core Service Categories

**Authentication Services:**
- `AuthService`: Registration, login, logout
- `OtpService`: OTP generation/verification
- `TwoFactorAuthService`: 2FA with Google Authenticator
- `ValidationService`: Business validation rules

**Media Services:**
- `MediaUploadService`: File upload to Cloudflare R2
- `ImageProcessingService`: Resize, crop, thumbnail generation
- Always process images in background for large files

**Communication Services:**
- `NotificationService`: Push notifications via Expo
- `PresenceService`: Online status, typing indicators
- `CallMessageService`: Call-related chat messages

**Video Services:**
- `VideoSDKService`: Primary video calling (VideoSDK.live)
- `AgoraService`: Backup video calling (Agora RTC)
- `AgoraTokenBuilder`: Token generation for Agora

**Advanced Services:**
- `VerificationService`: Identity verification (5 types: identity, photo, employment, education, income)
- `PanicButtonService`: Emergency panic button with GPS, contact alerts, admin notifications
- `MatchmakerService`: Professional matchmaker features (consultations, introductions, reviews)
- `FamilyApprovalService`: Family involvement features (family members, approval workflows)
- `UsageLimitsService`: Subscription limit enforcement (likes, messages, profile views)
- `PaymentManager`: Payment processing (transactions, subscriptions, refunds)
- `PrayerTimeService`: Islamic prayer time notifications and preferences
- `EnhancedReportingService`: Advanced user reporting with evidence attachments
- `PrivacyService`: Privacy controls and data protection
- `CacheService`: Cache management and optimization
- `ErrorHandlingService`: Centralized error tracking and logging

### Service Transaction Pattern
Services must use transactions for multi-model operations:
```php
public function createMatch($userId, $likedUserId) {
    DB::beginTransaction();
    try {
        $match = Match::create([...]);
        $chat = Chat::create([...]);
        $chat->users()->attach([$userId, $likedUserId]);
        event(new NewMatchEvent($match));
        DB::commit();
        return $match;
    } catch (\Exception $e) {
        DB::rollBack();
        throw $e;
    }
}
```

## Frontend Architecture

### JavaScript Architecture

**Modules (resources/js/):**
- `app.js`: Entry point, Alpine initialization
- `echo.js`: WebSocket client configuration
- `auth.js`: Authentication flows
- `messages.js`: Chat real-time updates
- `video-call.js`: Video calling (VideoSDK integration)
- `videosdk.js`: VideoSDK wrapper
- `theme.js`: Dark/light mode
- `country-data.js`: Country selection data
- `registration-store.js`: Multi-step registration state

**Alpine.js Integration:**
- Used for lightweight interactivity (dropdowns, modals, tabs)
- Example: `x-show`, `x-transition` for UI animations

## Third-Party Integrations

### VideoSDK.live (Primary Video Calling)
- Generate token: `VideoSDKService::getToken()`
- Create meeting: `VideoSDKService::createMeeting()`
- Join meeting: Client-side via `resources/js/videosdk.js`
- Meeting ID stored in `calls` table

### Agora RTC (Backup Video Calling)
- Similar pattern to VideoSDK
- Token builder: `AgoraService::generateToken()`
- Only used if VideoSDK fails

### Cloudflare R2 (Media Storage)
- S3-compatible API
- Config: `config/filesystems.php` disk `r2`
- Upload via `MediaUploadService::upload()`
- Signed URLs for private media

### Expo Push Notifications
- Token storage: `device_tokens` table
- Send via `ExpoPushService::send()`
- Register token: `DeviceTokenController::store()`

## Critical Configuration

### Environment Variables (Priority Order)

**Must Configure for Basic Operation:**
```env
APP_KEY=                           # Generate: php artisan key:generate
APP_URL=http://localhost            # Production: https://yourdomain.com
DB_CONNECTION=mysql                # Use MySQL in production (SQLite for dev)
BROADCAST_CONNECTION=reverb        # WebSocket broadcasting
QUEUE_CONNECTION=database          # Use Redis in production

# Laravel Reverb (WebSocket)
REVERB_APP_ID=yoryor-app
REVERB_APP_KEY=yoryor-key-123456
REVERB_APP_SECRET=yoryor-secret-123456
REVERB_HOST=localhost              # Production: your domain
REVERB_PORT=8080
REVERB_SCHEME=http                 # Production: https

# Vite (for WebSocket client)
VITE_REVERB_APP_KEY="${REVERB_APP_KEY}"
VITE_REVERB_HOST="${REVERB_HOST}"
VITE_REVERB_PORT="${REVERB_PORT}"
VITE_REVERB_SCHEME="${REVERB_SCHEME}"
```

**Must Configure for Full Functionality:**
```env
# Video Calling (Primary)
VIDEOSDK_API_KEY=
VIDEOSDK_SECRET_KEY=
VIDEOSDK_API_ENDPOINT=https://api.videosdk.live/v2

# Media Storage (Cloudflare R2)
CLOUDFLARE_R2_ACCESS_KEY_ID=
CLOUDFLARE_R2_SECRET_ACCESS_KEY=
CLOUDFLARE_R2_DEFAULT_REGION=auto
CLOUDFLARE_R2_BUCKET=
CLOUDFLARE_R2_URL=https://your-account.r2.cloudflarestorage.com
CLOUDFLARE_R2_ENDPOINT=https://your-account.r2.cloudflarestorage.com
CLOUDFLARE_R2_USE_PATH_STYLE_ENDPOINT=true
```

**Optional but Recommended:**
```env
# Backup Video Calling
AGORA_APP_ID=
AGORA_APP_CERTIFICATE=

# Social Authentication
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT_URI=

# Email (Production)
MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_FROM_ADDRESS=noreply@yoryor.com
MAIL_FROM_NAME="${APP_NAME}"

# Cache & Queue (Production - Use Redis)
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
CACHE_STORE=redis
QUEUE_CONNECTION=redis
```

### Localization Configuration
The app supports **3 languages**: English (en), Uzbek (uz), Russian (ru)

```php
// config/app.php
'locale' => env('APP_LOCALE', 'en'),
'available_locales' => ['en', 'uz', 'ru'],
```

Language files location: `resources/lang/{locale}/`

### Queue Configuration
- Queue driver: `database` (development), `redis` (production recommended)
- Jobs:
  - `SendEmergencyNotificationJob`: Panic button emergency alerts
  - `ProcessVerificationDocumentsJob`: Async document verification processing
- Run worker: `php artisan queue:listen --tries=1`
- Monitor with Horizon: `php artisan horizon` (requires Redis)
- View queue jobs: Check `jobs` table in database
- Failed jobs: `failed_jobs` table, retry with `php artisan queue:retry all`

### Storage Configuration
**Local Development:**
- Disk: `local` (storage/app/)
- Public access: `php artisan storage:link`

**Production:**
- Disk: `r2` (Cloudflare R2)
- Config: `config/filesystems.php`
- Automatic signed URLs for private media
- Thumbnails generated on upload

## Common Patterns

### Creating New API Endpoint
1. Add route: `routes/api.php` in appropriate section
2. Create controller method: `app/Http/Controllers/Api/V1/`
3. Create/use service: `app/Services/`
4. Add validation rules
5. Create API Resource if returning data: `app/Http/Resources/`
6. Add rate limiting: `->middleware('api.rate.limit:action_type')`
7. Update Swagger docs if needed

### Adding New Migration
1. Create migration: `php artisan make:migration create_table_name`
2. Define schema in `up()` method
3. Add indexes on foreign keys and query fields
4. Add foreign key constraints
5. Run migration: `php artisan migrate`
6. Create model if new table: `php artisan make:model ModelName`
7. Define relationships in model

### Broadcasting Real-Time Events
1. Create event: `php artisan make:event EventName`
2. Implement `ShouldBroadcast`
3. Define `broadcastOn()` returning channels
4. Dispatch: `event(new EventName($data))`
5. Listen in JavaScript: `Echo.private('channel').listen('EventName', callback)`
6. Ensure Reverb server running

## File Location Reference

**Never create files in these locations without understanding impact:**
- `bootstrap/cache/` - Auto-generated cache files
- `storage/framework/` - Framework storage (sessions, cache, compiled views)
- `public/build/` - Vite compiled assets (auto-generated)

**Configuration files:**
- `config/app.php` - App settings, locales
- `config/services.php` - Third-party API credentials
- `config/filesystems.php` - Storage disks (local, r2)
- `config/broadcasting.php` - Reverb configuration
- `bootstrap/app.php` - Middleware, routing configuration

**Critical startup files:**
- `bootstrap/app.php` - Application bootstrap, middleware aliases
- `routes/channels.php` - WebSocket channel authorization
- `app/Providers/AppServiceProvider.php` - Service container bindings

## Debugging

### Laravel Telescope
- Access: `http://localhost:8000/telescope`
- View requests, queries, events, jobs, logs
- Only enabled in local environment
- Clear entries: `php artisan telescope:clear`

### Laravel Tinker
```bash
php artisan tinker

# Query users
User::with('profile')->first()

# Test service
$authService = app(AuthService::class)
$authService->register([...])

# Fire event
event(new NewMessageEvent($message))
```

### Common Issues

**WebSocket not connecting:**
- Verify Reverb running: `php artisan reverb:start`
- Check port 8080 not in use
- Verify `.env` has correct `REVERB_*` vars
- Check `resources/js/echo.js` configuration

**Queue jobs not processing:**
- Start queue worker: `php artisan queue:listen --tries=1`
- Check `jobs` table for pending jobs
- View failed jobs: `php artisan queue:failed`
- Retry failed: `php artisan queue:retry all`

**Images not uploading/displaying:**
- Verify Cloudflare R2 credentials in `.env`
- Check storage link: `php artisan storage:link`
- Verify `public` disk accessible
- Check file permissions: `storage/` and `public/` should be writable
- Image processing requires `GD` or `Imagick` PHP extension

**Database migration errors:**
- Foreign key constraints: Run `2025_09_24_999999_add_foreign_key_constraints.php` last
- Check migration order (chronological by timestamp)
- Never modify existing migrations in production
- Use `migrate:fresh` only in development (destroys data)

**Multilingual features not working:**
- Check session has locale: `session()->get('locale')`
- Verify translation files exist: `resources/lang/{locale}/`
- Use `__('key')` or `@lang('key')` in Blade
- Set locale: `App::setLocale($locale)`
- Route: `/locale/{locale}` to switch languages

## Documentation

Comprehensive documentation available in `/docs/`:

**Main Documentation Hub:** `/docs/README.md` - Complete navigation and overview

**API Documentation** (`/docs/api/`):
- `ENDPOINTS.md` - Complete API reference with 100+ endpoints
- `AUTHENTICATION.md` - Auth flows, 2FA, OTP, token management
- `WEBSOCKETS.md` - Laravel Reverb WebSocket setup and events
- `MOBILE_INTEGRATION.md` - React Native/Expo integration guide

**Web Documentation** (`/docs/web/`):
- `FRONTEND_ARCHITECTURE.md` - Frontend structure and patterns
- `THEMING.md` - Dark/light mode, icons, design tokens

**Development** (`/docs/development/`):
- `GETTING_STARTED.md` - Installation, setup, workflow
- `ARCHITECTURE.md` - System architecture, design patterns
- `DATABASE.md` - Complete schema for 70+ tables
- `SERVICES.md` - All 25+ services documentation
- `TESTING.md` - Pest PHP testing guide

**Deployment** (`/docs/deployment/`):
- `PRODUCTION.md` - Complete production deployment guide
- `SECURITY.md` - Security hardening and infrastructure

**Features** (`/docs/features/`):
- `OVERVIEW.md` - All 30+ features overview
- `PROFILES.md` - Multi-section profile system
- `MATCHING.md` - AI-powered matching algorithm
- `CHAT.md` - Real-time messaging system
- `VIDEO_CALLING.md` - VideoSDK integration
- `SAFETY.md` - Safety and privacy features

**Maintenance** (`/docs/maintenance/`):
- `CODE_QUALITY_ISSUES.md` - Code quality and refactoring
- `SECURITY_AUDIT.md` - Security audit findings
- `PERFORMANCE_IMPROVEMENTS.md` - Performance optimizations

## Project-Specific Conventions

### Naming Conventions
- Models: Singular PascalCase (`User`, `UserPhoto`, `UserMatch`, `UserReport`)
  - Avoid PHP reserved keywords: Use `UserMatch` not `Match`, `UserEnum` not `Enum`
- Controllers: Plural + Controller (`UsersController`, `ChatsController`)
- Services: Singular + Service (`AuthService`, `MediaUploadService`)
- Migrations: snake_case with action (`create_users_table`, `add_foreign_keys`)
- Events: PascalCase + Event (`NewMessageEvent`, `CallInitiatedEvent`)
- Table Names: Explicit `protected $table` when model name differs from table (e.g., `UserMatch` → `matches`)

### Model Relationships
- Always define inverse relationships
- Use explicit relationship methods: `belongsTo()`, `hasMany()`, `belongsToMany()`
- Specify foreign keys explicitly if not following convention
- Use `withTimestamps()` on pivot tables that have timestamps

### API Response Format
- Success: `{ type, id, attributes, relationships }`
- Error: `{ status: 'error', message: '...', errors: {...} }`
- Always return appropriate HTTP status codes

### Route Naming
- API routes: `api.v1.resource.action` (e.g., `api.v1.users.show`)
- Web routes: `resource.action` (e.g., `profile.edit`)
- Use route names, not hardcoded URLs
- Named routes accessed: `route('name', ['param' => $value])`

## Key Business Rules

### Profile Completion Requirements
- Minimum 2 photos required for account activation
- Basic profile must be completed before profile visibility
- `registration_completed` flag tracks onboarding status
- Profile completion percentage calculated from filled sections

### Matching Logic
- **Like System**: User A likes User B → stored in `likes` table
- **Match Creation**: If User B also likes User A → automatic `match` record created
- **Chat Creation**: Match automatically creates a private chat
- **Filters Applied**: Gender preference, age range, distance, cultural compatibility

### Privacy & Safety
- **Profile Visibility Options**:
  - `public`: Visible to all active users
  - `matches_only`: Only visible to matched users
  - `private`: Hidden from discovery (searchable only by username/ID)
- **Photo Privacy**: Separate control from profile visibility
- **Blocking**: Blocked users cannot see profile, send messages, or appear in discovery
- **Reporting**: Evidence-based reporting system with admin review workflow
- **Panic Button**: GPS location sent to emergency contacts + admin alert

### Subscription System (3 Tiers)
- **Free**: Basic features with usage limits
  - 50 likes/month
  - Limited messages
  - Basic search
- **Premium**: Enhanced features
  - Unlimited likes
  - See who liked you
  - Advanced filters
  - Read receipts
- **Premium Plus**: All features
  - Matchmaker access
  - Priority support
  - Verification fast-track
  - Profile boost

### Verification Badges (5 Types)
1. **Identity**: Government ID verification
2. **Photo**: Selfie verification matching profile photos
3. **Employment**: Work verification via employer
4. **Education**: Degree/certificate verification
5. **Income**: Income bracket verification

### Family Involvement Features
- Family members can be added to account
- Family approval workflow for matches
- Shared profile access (controlled permissions)
- Communication channels with potential match families

## Security Principles

### Password Requirements
- Minimum 8 characters
- Bcrypt hashing with 12 rounds
- Optional 2FA with Google Authenticator (TOTP)
- Account lockout after 5 failed attempts

### Data Protection
- Sensitive data encrypted at rest
- HTTPS enforced in production
- Signed URLs for private media
- CSRF protection on all forms
- XSS prevention via Blade escaping
- SQL injection prevention via Eloquent ORM

### GDPR Compliance
- **Right to Access**: Users can export their data via `DataExportRequest`
- **Right to Deletion**: Account deletion with 30-day grace period
- **Right to Rectification**: Users can update all personal data
- **Consent Tracking**: Explicit consent for data usage
- **Privacy Policy**: Available at `/privacy`
- **Terms of Service**: Available at `/terms`

===

<laravel-boost-guidelines>
=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to enhance the user's satisfaction building Laravel applications.

## Foundational Context
This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.4.14
- laravel/framework (LARAVEL) - v12
- laravel/horizon (HORIZON) - v5
- laravel/prompts (PROMPTS) - v0
- laravel/pulse (PULSE) - v1
- laravel/reverb (REVERB) - v1
- laravel/sanctum (SANCTUM) - v4
- laravel/socialite (SOCIALITE) - v5
- laravel/telescope (TELESCOPE) - v5
- laravel/mcp (MCP) - v0
- laravel/pint (PINT) - v1
- laravel/sail (SAIL) - v1
- pestphp/pest (PEST) - v3
- phpunit/phpunit (PHPUNIT) - v11
- alpinejs (ALPINEJS) - v3
- tailwindcss (TAILWINDCSS) - v4
- laravel-echo (ECHO) - v2

## Conventions
- You must follow all existing code conventions used in this application. When creating or editing a file, check sibling files for the correct structure, approach, naming.
- Use descriptive names for variables and methods. For example, `isRegisteredForDiscounts`, not `discount()`.
- Check for existing components to reuse before writing a new one.

## Verification Scripts
- Do not create verification scripts or tinker when tests cover that functionality and prove it works. Unit and feature tests are more important.

## Application Structure & Architecture
- Stick to existing directory structure - don't create new base folders without approval.
- Do not change the application's dependencies without approval.

## Frontend Bundling
- If the user doesn't see a frontend change reflected in the UI, it could mean they need to run `npm run build`, `npm run dev`, or `composer run dev`. Ask them.

## Replies
- Be concise in your explanations - focus on what's important rather than explaining obvious details.

## Documentation Files
- You must only create documentation files if explicitly requested by the user.


=== boost rules ===

## Laravel Boost
- Laravel Boost is an MCP server that comes with powerful tools designed specifically for this application. Use them.

## Artisan
- Use the `list-artisan-commands` tool when you need to call an Artisan command to double check the available parameters.

## URLs
- Whenever you share a project URL with the user you should use the `get-absolute-url` tool to ensure you're using the correct scheme, domain / IP, and port.

## Tinker / Debugging
- You should use the `tinker` tool when you need to execute PHP to debug code or query Eloquent models directly.
- Use the `database-query` tool when you only need to read from the database.

## Reading Browser Logs With the `browser-logs` Tool
- You can read browser logs, errors, and exceptions using the `browser-logs` tool from Boost.
- Only recent browser logs will be useful - ignore old logs.

## Searching Documentation (Critically Important)
- Boost comes with a powerful `search-docs` tool you should use before any other approaches. This tool automatically passes a list of installed packages and their versions to the remote Boost API, so it returns only version-specific documentation specific for the user's circumstance. You should pass an array of packages to filter on if you know you need docs for particular packages.
- The 'search-docs' tool is perfect for all Laravel related packages, including Laravel, Inertia, Filament, Tailwind, Pest, Nova, Nightwatch, etc.
- You must use this tool to search for Laravel-ecosystem documentation before falling back to other approaches.
- Search the documentation before making code changes to ensure we are taking the correct approach.
- Use multiple, broad, simple, topic based queries to start. For example: `['rate limiting', 'routing rate limiting', 'routing']`.
- Do not add package names to queries - package information is already shared. For example, use `test resource table`, not `filament 4 test resource table`.

### Available Search Syntax
- You can and should pass multiple queries at once. The most relevant results will be returned first.

1. Simple Word Searches with auto-stemming - query=authentication - finds 'authenticate' and 'auth'
2. Multiple Words (AND Logic) - query=rate limit - finds knowledge containing both "rate" AND "limit"
3. Quoted Phrases (Exact Position) - query="infinite scroll" - Words must be adjacent and in that order
4. Mixed Queries - query=middleware "rate limit" - "middleware" AND exact phrase "rate limit"
5. Multiple Queries - queries=["authentication", "middleware"] - ANY of these terms


=== php rules ===

## PHP

- Always use curly braces for control structures, even if it has one line.

### Constructors
- Use PHP 8 constructor property promotion in `__construct()`.
    - <code-snippet>public function __construct(public GitHub $github) { }</code-snippet>
- Do not allow empty `__construct()` methods with zero parameters.

### Type Declarations
- Always use explicit return type declarations for methods and functions.
- Use appropriate PHP type hints for method parameters.

<code-snippet name="Explicit Return Types and Method Params" lang="php">
protected function isAccessible(User $user, ?string $path = null): bool
{
    ...
}
</code-snippet>

## Comments
- Prefer PHPDoc blocks over comments. Never use comments within the code itself unless there is something _very_ complex going on.

## PHPDoc Blocks
- Add useful array shape type definitions for arrays when appropriate.

## Enums
- Typically, keys in an Enum should be TitleCase. For example: `FavoritePerson`, `BestLake`, `Monthly`.


=== laravel/core rules ===

## Do Things the Laravel Way

- Use `php artisan make:` commands to create new files (i.e. migrations, controllers, models, etc.). You can list available Artisan commands using the `list-artisan-commands` tool.
- If you're creating a generic PHP class, use `artisan make:class`.
- Pass `--no-interaction` to all Artisan commands to ensure they work without user input. You should also pass the correct `--options` to ensure correct behavior.

### Database
- Always use proper Eloquent relationship methods with return type hints. Prefer relationship methods over raw queries or manual joins.
- Use Eloquent models and relationships before suggesting raw database queries
- Avoid `DB::`; prefer `Model::query()`. Generate code that leverages Laravel's ORM capabilities rather than bypassing them.
- Generate code that prevents N+1 query problems by using eager loading.
- Use Laravel's query builder for very complex database operations.

### Model Creation
- When creating new models, create useful factories and seeders for them too. Ask the user if they need any other things, using `list-artisan-commands` to check the available options to `php artisan make:model`.

### APIs & Eloquent Resources
- For APIs, default to using Eloquent API Resources and API versioning unless existing API routes do not, then you should follow existing application convention.

### Controllers & Validation
- Always create Form Request classes for validation rather than inline validation in controllers. Include both validation rules and custom error messages.
- Check sibling Form Requests to see if the application uses array or string based validation rules.

### Queues
- Use queued jobs for time-consuming operations with the `ShouldQueue` interface.

### Authentication & Authorization
- Use Laravel's built-in authentication and authorization features (gates, policies, Sanctum, etc.).

### URL Generation
- When generating links to other pages, prefer named routes and the `route()` function.

### Configuration
- Use environment variables only in configuration files - never use the `env()` function directly outside of config files. Always use `config('app.name')`, not `env('APP_NAME')`.

### Testing
- When creating models for tests, use the factories for the models. Check if the factory has custom states that can be used before manually setting up the model.
- Faker: Use methods such as `$this->faker->word()` or `fake()->randomDigit()`. Follow existing conventions whether to use `$this->faker` or `fake()`.
- When creating tests, make use of `php artisan make:test [options] <name>` to create a feature test, and pass `--unit` to create a unit test. Most tests should be feature tests.

### Vite Error
- If you receive an "Illuminate\Foundation\ViteException: Unable to locate file in Vite manifest" error, you can run `npm run build` or ask the user to run `npm run dev` or `composer run dev`.


=== laravel/v12 rules ===

## Laravel 12

- Use the `search-docs` tool to get version specific documentation.
- Since Laravel 11, Laravel has a new streamlined file structure which this project uses.

### Laravel 12 Structure
- No middleware files in `app/Http/Middleware/`.
- `bootstrap/app.php` is the file to register middleware, exceptions, and routing files.
- `bootstrap/providers.php` contains application specific service providers.
- **No app\Console\Kernel.php** - use `bootstrap/app.php` or `routes/console.php` for console configuration.
- **Commands auto-register** - files in `app/Console/Commands/` are automatically available and do not require manual registration.

### Database
- When modifying a column, the migration must include all of the attributes that were previously defined on the column. Otherwise, they will be dropped and lost.
- Laravel 11 allows limiting eagerly loaded records natively, without external packages: `$query->latest()->limit(10);`.

### Models
- Casts can and likely should be set in a `casts()` method on a model rather than the `$casts` property. Follow existing conventions from other models.


=== pint/core rules ===

## Laravel Pint Code Formatter

- You must run `vendor/bin/pint --dirty` before finalizing changes to ensure your code matches the project's expected style.
- Do not run `vendor/bin/pint --test`, simply run `vendor/bin/pint` to fix any formatting issues.


=== pest/core rules ===

## Pest

### Testing
- If you need to verify a feature is working, write or update a Unit / Feature test.

### Pest Tests
- All tests must be written using Pest. Use `php artisan make:test --pest <name>`.
- You must not remove any tests or test files from the tests directory without approval. These are not temporary or helper files - these are core to the application.
- Tests should test all of the happy paths, failure paths, and weird paths.
- Tests live in the `tests/Feature` and `tests/Unit` directories.
- Pest tests look and behave like this:
<code-snippet name="Basic Pest Test Example" lang="php">
it('is true', function () {
    expect(true)->toBeTrue();
});
</code-snippet>

### Running Tests
- Run the minimal number of tests using an appropriate filter before finalizing code edits.
- To run all tests: `php artisan test`.
- To run all tests in a file: `php artisan test tests/Feature/ExampleTest.php`.
- To filter on a particular test name: `php artisan test --filter=testName` (recommended after making a change to a related file).
- When the tests relating to your changes are passing, ask the user if they would like to run the entire test suite to ensure everything is still passing.

### Pest Assertions
- When asserting status codes on a response, use the specific method like `assertForbidden` and `assertNotFound` instead of using `assertStatus(403)` or similar, e.g.:
<code-snippet name="Pest Example Asserting postJson Response" lang="php">
it('returns all', function () {
    $response = $this->postJson('/api/docs', []);

    $response->assertSuccessful();
});
</code-snippet>

### Mocking
- Mocking can be very helpful when appropriate.
- When mocking, you can use the `Pest\Laravel\mock` Pest function, but always import it via `use function Pest\Laravel\mock;` before using it. Alternatively, you can use `$this->mock()` if existing tests do.
- You can also create partial mocks using the same import or self method.

### Datasets
- Use datasets in Pest to simplify tests which have a lot of duplicated data. This is often the case when testing validation rules, so consider going with this solution when writing tests for validation rules.

<code-snippet name="Pest Dataset Example" lang="php">
it('has emails', function (string $email) {
    expect($email)->not->toBeEmpty();
})->with([
    'james' => 'james@laravel.com',
    'taylor' => 'taylor@laravel.com',
]);
</code-snippet>


=== tailwindcss/core rules ===

## Tailwind Core

- Use Tailwind CSS classes to style HTML, check and use existing tailwind conventions within the project before writing your own.
- Offer to extract repeated patterns into components that match the project's conventions (i.e. Blade, JSX, Vue, etc..)
- Think through class placement, order, priority, and defaults - remove redundant classes, add classes to parent or child carefully to limit repetition, group elements logically
- You can use the `search-docs` tool to get exact examples from the official documentation when needed.

### Spacing
- When listing items, use gap utilities for spacing, don't use margins.

    <code-snippet name="Valid Flex Gap Spacing Example" lang="html">
        <div class="flex gap-8">
            <div>Superior</div>
            <div>Michigan</div>
            <div>Erie</div>
        </div>
    </code-snippet>


### Dark Mode
- If existing pages and components support dark mode, new pages and components must support dark mode in a similar way, typically using `dark:`.


=== tailwindcss/v4 rules ===

## Tailwind 4

- Always use Tailwind CSS v4 - do not use the deprecated utilities.
- `corePlugins` is not supported in Tailwind v4.
- In Tailwind v4, configuration is CSS-first using the `@theme` directive — no separate `tailwind.config.js` file is needed.
<code-snippet name="Extending Theme in CSS" lang="css">
@theme {
  --color-brand: oklch(0.72 0.11 178);
}
</code-snippet>

- In Tailwind v4, you import Tailwind using a regular CSS `@import` statement, not using the `@tailwind` directives used in v3:

<code-snippet name="Tailwind v4 Import Tailwind Diff" lang="diff">
   - @tailwind base;
   - @tailwind components;
   - @tailwind utilities;
   + @import "tailwindcss";
</code-snippet>


### Replaced Utilities
- Tailwind v4 removed deprecated utilities. Do not use the deprecated option - use the replacement.
- Opacity values are still numeric.

| Deprecated |	Replacement |
|------------+--------------|
| bg-opacity-* | bg-black/* |
| text-opacity-* | text-black/* |
| border-opacity-* | border-black/* |
| divide-opacity-* | divide-black/* |
| ring-opacity-* | ring-black/* |
| placeholder-opacity-* | placeholder-black/* |
| flex-shrink-* | shrink-* |
| flex-grow-* | grow-* |
| overflow-ellipsis | text-ellipsis |
| decoration-slice | box-decoration-slice |
| decoration-clone | box-decoration-clone |


=== tests rules ===

## Test Enforcement

- Every change must be programmatically tested. Write a new test or update an existing test, then run the affected tests to make sure they pass.
- Run the minimum number of tests needed to ensure code quality and speed. Use `php artisan test` with a specific filename or filter.
</laravel-boost-guidelines>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hitcklife) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
