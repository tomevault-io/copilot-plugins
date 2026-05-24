## site-wewingames

> WeWinGames is a comprehensive sports betting information and picks service built with Laravel 12 and Vue.js 3. The platform provides betting recommendations, game analysis, subscription-based access to premium picks, and a full suite of content management and user engagement features.

# WeWinGames - Sports Betting Platform

## Overview
WeWinGames is a comprehensive sports betting information and picks service built with Laravel 12 and Vue.js 3. The platform provides betting recommendations, game analysis, subscription-based access to premium picks, and a full suite of content management and user engagement features.

## Technology Stack

### Backend
- **Framework**: Laravel 12 (PHP 8.2+)
- **Database**: MySQL 8.0 / SQLite (local development)
- **Cache**: File/Redis with Cloudflare integration
- **Queue**: Laravel Queue with database driver
- **Authentication**: Laravel Breeze with Inertia.js
- **Billing**: Laravel Cashier (Stripe integration)
- **SSR**: Inertia.js v2

### Frontend
- **Framework**: Vue.js 3 with TypeScript
- **Build Tool**: Vite 6
- **CSS**: Bootstrap 5 (migrated from Tailwind CSS)
- **UI Components**: Bootstrap 5 components with custom admin theme
- **Rich Text**: TinyMCE and Tiptap editors
- **Charts**: Chart.js with vue-chartjs
- **3D Graphics**: Three.js
- **Icons**: Bootstrap Icons, Lucide Vue, Heroicons

### Third-Party Services
- **Payment**: Stripe (with dynamic product management)
- **Push Notifications**: OneSignal & Web Push API
- **Analytics**: Google Analytics & Tag Manager
- **Security**: Cloudflare Turnstile
- **Email**: SendGrid (via LoggedMailChannel)
- **Marketing**: SpringBig (member sync & tier segments)
- **Monitoring**: Laravel Telescope
- **Media**: Spatie Media Library
- **Permissions**: Spatie Laravel Permission
- **Activity Logging**: Spatie Activity Log

## Project Structure

```
.
├── app/
│   ├── Console/           # Artisan commands
│   ├── Events/            # Event classes
│   ├── Http/
│   │   ├── Controllers/   # All controllers including Admin/
│   │   ├── Middleware/    # Custom middleware
│   │   └── Requests/      # Form requests
│   ├── Models/            # Eloquent models (30+ models)
│   ├── Services/          # Business logic services
│   ├── Policies/          # Authorization policies
│   └── Mail/              # Mailable classes
├── resources/
│   ├── js/
│   │   ├── components/    # Reusable Vue components
│   │   ├── pages/         # Page components
│   │   ├── layouts/       # Layout components
│   │   └── composables/   # Vue composition utilities
│   └── css/               # Bootstrap-based styles
├── routes/                # Application routes
├── database/              # Migrations, seeders, factories
├── tests/                 # PHPUnit & Feature tests
└── docker/                # Docker configuration
```

## Key Features

### 1. Betting System
- **Bet Management**: Create, edit, track betting picks with performance metrics
- **Parlay Support**: Multi-bet parlays with combined odds
- **Golf Betting**: Each-way bets with place fractions and dead heat rules
- **CSV Import/Export**: Wizard-based bulk data management
- **Mass Edit**: Batch updates for golf positions
- **Premium Notes**: Subscriber-only betting insights
- **Profit Tracking**: Detailed P&L calculations

### 2. User & Subscription System
- **User Types**: Regular, Ambassador, Gifted, Admin
- **Subscription Tiers**: Bronze, Silver, Gold, Platinum
- **Billing Periods**: Daily, Weekly, Monthly, Yearly
- **Dynamic Stripe Products**: Database-driven product management
- **Discount Codes**: Percentage/fixed with usage limits
- **Affiliate System**: Track and manage affiliates
- **Impersonation**: Admin user switching
- **Quick Checkout**: Payment-first registration flow (feature flagged)

### 3. Content Management
- **Blog System**: Full-featured with SEO, categories, view tracking
- **CMS Pages**: Dynamic page creation and management
- **Landing Pages**: Marketing-focused pages
- **FAQ System**: Categorized Q&A management
- **Knowledgebase**: Article-based help system
- **Media Library**: Centralized file management
- **Testimonials**: Customer reviews with Google integration

### 4. Communication Features
- **Email System**: 
  - Template management
  - Full logging with SendGrid
  - Customizable transactional emails
- **Push Notifications**:
  - OneSignal integration (NEW)
  - Web Push API fallback
  - Tier-based targeting
  - Notification history
- **Support Tickets**: Guest-accessible support system

### 5. Career/Jobs System
- **Job Positions**: Manage job listings
- **Resume Submissions**: Application tracking system
- **Admin Review**: Application management interface

### 6. Admin Dashboard
Located at `/admin`, provides:
- **Statistics Dashboard**: MRR, user growth, betting activity
- **User Management**: Complete user administration
- **Bet Management**: Full CRUD with import/export
- **Content Editing**: Pages, posts, FAQs, testimonials
- **Subscription Dashboard**: Customer & revenue tracking
- **System Settings**: Configuration management
- **Activity Logs**: User action tracking
- **Cache Management**: Clear Laravel & Cloudflare cache

### 7. Security & Performance
- **Middleware**:
  - Admin security headers
  - Rate limiting
  - IP blacklisting
  - Spam prevention
- **Under Construction Mode**: Site-wide maintenance
- **Cloudflare Integration**: CDN & cache management
- **Session Security**: CSRF protection

### 8. Quick Checkout (Payment-First Registration)
A conversion-optimized registration flow that collects payment before account setup.

**Flow:**
1. Guest visits `/quick-checkout?plan=gold&period=monthly`
2. Enters email, name, phone + card via Stripe Elements
3. Payment processed → User created with `status=pending_setup`
4. Email sent with completion link → `/complete-registration?token=xxx`
5. User sets password + optional Discord → `status=active`, logged in

**Key Files:**
- `app/Services/QuickCheckoutService.php` - Business logic
- `app/Http/Controllers/QuickCheckoutController.php` - Endpoints
- `app/Http/Requests/QuickCheckoutRequest.php` - Validation
- `app/Http/Requests/CompleteRegistrationRequest.php` - Password validation
- `app/Mail/CompleteYourAccountMail.php` - Completion email
- `resources/js/pages/QuickCheckout.vue` - Checkout form
- `resources/js/pages/CompleteRegistration.vue` - Password setup

**Feature Flag:**
```env
QUICK_CHECKOUT_ENABLED=true  # Enable payment-first flow
```

When enabled, pricing CTAs route guests to `/quick-checkout` instead of `/register`.

**User Status Values:**
- `active` - Normal active user
- `disabled` - Account disabled
- `pending_setup` - Quick checkout user awaiting password setup

**Fallback:** Users who don't complete setup can use "Forgot Password" to access their account.

## Development Commands

### Quick Start
```bash
# Install dependencies
composer install && npm install

# Setup environment
cp .env.example .env
php artisan key:generate

# Database setup
php artisan migrate --seed

# Start development
composer dev
```

### Available Scripts
```bash
# Development
composer dev              # All services (recommended)
composer dev:ssr          # With SSR
npm run dev              # Vite only
npm run build            # Production build

# Code Quality
composer format          # Format PHP (Pint)
npm run format          # Format JS/TS (Prettier)
npm run lint            # ESLint
npm run typecheck       # TypeScript check

# Testing
php artisan test         # Run all tests
php artisan test --filter TestName
php artisan test --coverage

# Database
php artisan migrate:fresh --seed  # Reset database
php artisan db:seed --class=ProductionSeeder  # Production data only
```

## Database Schema

### Core Tables
- `users` - User accounts with ambassador/gifted fields
- `bets` - Betting picks and predictions
- `games` - Sporting events
- `teams` - Sports teams with aliases
- `sports` - Sport categories
- `operators` - Betting operators/bookmakers
- `leagues` - Team leagues
- `subscriptions` - Laravel Cashier subscriptions
- `stripe_products` - Dynamic Stripe configuration
- `discount_codes` - Coupon management
- `discount_redemptions` - Usage tracking

### Content Tables
- `pages` - CMS pages
- `landing_pages` - Marketing pages
- `posts` - Blog posts
- `blog_categories` - Blog categorization
- `faqs` - FAQ entries
- `faq_categories` - FAQ organization
- `testimonials` - Customer reviews
- `knowledgebase_articles` - Help articles
- `knowledgebase_categories` - KB organization

### Communication Tables
- `support_tickets` - Support system
- `support_ticket_replies` - Ticket responses
- `notifications` - System notifications
- `push_subscriptions` - Web Push subscriptions
- `push_notification_logs` - Push history
- `email_logs` - Email tracking
- `email_templates` - Email customization

### Career Tables
- `job_positions` - Job listings
- `resume_submissions` - Applications

### System Tables
- `activity_log` - User activity tracking
- `media` - Spatie media library
- `affiliates` - Affiliate management
- `sport_user` - User sport preferences
- `team_logos` - Team branding

## API Routes

### Public API
```
POST   /login                    - User authentication
POST   /register                 - User registration
POST   /logout                   - User logout
POST   /forgot-password          - Password reset

GET    /quick-checkout           - Quick checkout page (payment-first)
POST   /quick-checkout           - Process quick checkout
GET    /complete-registration    - Complete account setup
POST   /complete-registration    - Set password after payment

GET    /api/bets                 - List bets
POST   /api/bets                 - Create bet
GET    /api/games                - List games
GET    /api/sports               - List sports
GET    /api/user                 - Current user
PUT    /api/user/profile         - Update profile
POST   /api/user/subscription    - Manage subscription

POST   /api/push/subscribe       - Subscribe to push
DELETE /api/push/unsubscribe     - Unsubscribe from push
```

### Admin API
```
POST   /admin/cache/clear        - Clear all caches
GET    /admin/api/customers/search - Search customers
POST   /admin/notifications/push/send - Send push notification
POST   /admin/notifications/push/test - Test notification
```

## Environment Variables

### Required
```env
APP_NAME=WeWinGames
APP_ENV=local
APP_URL=http://wewingames.test
APP_DEBUG=false  # MUST be false in production

# Database
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=wewingames
DB_USERNAME=sail
DB_PASSWORD=password

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# Mail
MAIL_MAILER=smtp  # or 'log' for development
MAIL_HOST=smtp.sendgrid.net
MAIL_PORT=587
MAIL_USERNAME=apikey
MAIL_PASSWORD=your_sendgrid_api_key
MAIL_FROM_ADDRESS=noreply@wewingames.com

# Stripe
STRIPE_KEY=pk_live_xxx
STRIPE_SECRET=sk_live_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx

# Admin
ADMIN_PASSWORD=YourSecurePassword123!  # For production seeder
```

### Optional Services
```env
# Quick Checkout (Payment-First Registration)
QUICK_CHECKOUT_ENABLED=false  # Set to true to enable

# Push Notifications
VAPID_PUBLIC_KEY=your_public_key
VAPID_PRIVATE_KEY=your_private_key
VAPID_SUBJECT=mailto:admin@wewingames.com

# Analytics
GOOGLE_ANALYTICS_TAG_ID=G-ZTJTTQP72Q
GOOGLE_TAG_MANAGER_ID=GTM-PQDDCG6L

# Security
TURNSTILE_ENABLED=true
TURNSTILE_SITE_KEY=0x4AAAAAABjA9oaFF9BSsznw
TURNSTILE_SECRET_KEY=0x4AAAAAABjA9iC5axcso_Tat1vZ1G-JsZc

# Cloudflare
CLOUDFLARE_ENABLED=true
CLOUDFLARE_EMAIL=your_email
CLOUDFLARE_API_KEY=your_api_key
CLOUDFLARE_ZONE_ID=your_zone_id

# Notifications (Optional)
SLACK_BOT_USER_OAUTH_TOKEN=xoxb-xxx
SLACK_BOT_USER_DEFAULT_CHANNEL=#alerts
POSTMARK_TOKEN=xxx
RESEND_KEY=xxx

# SpringBig Integration (Member Sync)
SPRINGBIG_ENABLED=false
SPRINGBIG_BASE_URL=https://production.api.springbig.technology/pos/v1
SPRINGBIG_API_KEY=your_api_key
SPRINGBIG_AUTH_TOKEN=your_auth_token

# SpringBig External Group (Segment-based, disabled by default)
SPRINGBIG_EXTERNAL_GROUP_ENABLED=false
SPRINGBIG_EXTERNAL_GROUP_BASE_URL=https://production.api.springbig.technology/general/v1
SPRINGBIG_EXTERNAL_GROUP_ID=
SPRINGBIG_SEGMENT_FREE=
SPRINGBIG_SEGMENT_SILVER=
SPRINGBIG_SEGMENT_GOLD=
SPRINGBIG_SEGMENT_PLATINUM=
SPRINGBIG_SEGMENT_CANCELED=
```

## SpringBig Integration

SpringBig syncs user subscription tiers for marketing automation. Two approaches available:

### 1. Custom Group List (Default)
Uses `custom_group_list` field on member POST/PUT calls. Sends tier as comma-separated string.

**Env:**
```env
SPRINGBIG_ENABLED=true
SPRINGBIG_API_KEY=your_api_key
SPRINGBIG_AUTH_TOKEN=your_merchant_id
```

**Usage:** Automatic on user registration/subscription change via `SpringBigService`.

### 2. External Group Segments (Optional)
Uses separate External Group API with segments for each tier. Members added/removed from segments on subscription changes.

**Setup:**
```bash
# 1. List existing groups/segments
php artisan springbig:setup-external-group --list

# 2. Create external group (one-time)
php artisan springbig:setup-external-group --create-group
# Output: SPRINGBIG_EXTERNAL_GROUP_ID=123

# 3. Create tier segments (one-time)
php artisan springbig:setup-external-group --create-segments
# Output: SPRINGBIG_SEGMENT_FREE=456
#         SPRINGBIG_SEGMENT_GOLD=457
#         etc.

# Or do both at once
php artisan springbig:setup-external-group --all
```

**After setup, add to .env:**
```env
SPRINGBIG_EXTERNAL_GROUP_ENABLED=true
SPRINGBIG_EXTERNAL_GROUP_ID=123
SPRINGBIG_SEGMENT_FREE=456
SPRINGBIG_SEGMENT_SILVER=457
SPRINGBIG_SEGMENT_GOLD=458
SPRINGBIG_SEGMENT_PLATINUM=459
SPRINGBIG_SEGMENT_CANCELED=460
```

**Key Files:**
- `app/Services/SpringBigService.php` - All API methods
- `app/Console/Commands/SpringBigSetupExternalGroup.php` - Setup command
- `app/Console/Commands/SyncSpringBigMembers.php` - Bulk sync command
- `config/services.php` - Configuration

**Sync Command (custom_group_list):**
```bash
# Dry run - see what would sync
php artisan springbig:sync-members --dry-run

# Sync all members (lookup by email/phone, then update or create)
php artisan springbig:sync-members

# Force create new (skip lookup - use for fresh SpringBig account)
php artisan springbig:sync-members --create

# Filter by tier
php artisan springbig:sync-members --tier=gold

# Sync specific user
php artisan springbig:sync-members --user=123

# Limit batch size
php artisan springbig:sync-members --limit=100
```

**Sync Flow:**
1. `GET /members?email=xxx` - Lookup by email
2. If not found, `GET /members?phone_number=xxx` - Lookup by phone
3. If found → `PUT /members/{pos_user}` - Update with tier
4. If not found → `POST /members` - Create new member

**Service Methods:**
| Method | Purpose |
|--------|---------|
| `getMemberByEmail($email)` | Lookup member by email |
| `getMemberByPhone($phone)` | Lookup member by phone |
| `createMember($user)` | Create member with custom_group_list |
| `updateMember($user, $posUser)` | Update member by pos_user |
| `syncMember($user)` | Lookup by email/phone, then update or create |
| `createExternalGroup($name, $desc)` | Create external group |
| `createSegments()` | Create tier segments |
| `addUserToSegment($user, $tier)` | Add user to segment |
| `removeUserFromSegment($user, $tier)` | Remove user from segment |
| `updateUserSegment($user, $oldTier)` | Move user between segments |
| `syncUserSegment($user)` | Full sync (remove all, add current) |

**API Endpoints Used:**
- POS API: `POST/PUT /pos/v1/members` (custom_group_list)
- External Group: `POST/GET /general/v1/external_group`
- Segments: `POST/GET/PUT /general/v1/external_group_segments`

## Deployment

### Production Build & Deploy
```bash
# Fix npm PATH if needed
export PATH="/opt/nvm/versions/node/v22.17.0/bin:$PATH"

# Install with reduced concurrency (prevents EMFILE errors)
npm install --maxsockets 1

# Build assets
npm run build

# Optimize Laravel
php artisan optimize:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan icons:cache

# Run migrations
php artisan migrate --force

# Seed production data
php artisan db:seed --class=ProductionSeeder
```

### Server Requirements
- PHP 8.2+ with extensions: BCMath, Ctype, JSON, Mbstring, OpenSSL, PDO, Tokenizer, XML
- MySQL 8.0+ or MariaDB 10.3+
- Redis 6.0+
- Node.js 18+ & npm 8+
- Composer 2+
- Minimum 2GB RAM
- SSL certificate for HTTPS

## Coding Standards

### PHP
- PSR-12 coding standards
- Laravel Pint for formatting
- Type declarations required
- Service pattern for business logic
- Repository pattern for data access

### JavaScript/TypeScript
- TypeScript for all new code
- Vue 3 Composition API
- ESLint & Prettier formatting
- Component-based architecture
- Proper type interfaces

### Git Workflow
- Feature branches from `main`
- Descriptive commit messages
- PR reviews required
- Run tests before merging
- Squash merge for features

## Security Best Practices

- Input validation on all forms
- CSRF protection enabled
- XSS prevention via Vue
- SQL injection prevention via Eloquent
- API rate limiting configured
- Secure session management
- Environment variables for secrets
- Regular dependency updates

## Performance Optimization

- Redis caching for frequent queries
- Database indexing on foreign keys
- Lazy loading for Vue components
- Image optimization with Vite
- Query optimization with eager loading
- CDN for static assets
- Gzip compression enabled
- Browser caching headers

## Common Tasks

### Adding a New Feature
1. Create migration: `php artisan make:model ModelName -mfsc`
2. Define relationships and fillable
3. Create policy: `php artisan make:policy ModelPolicy`
4. Add routes in appropriate file
5. Create controller with CRUD actions
6. Build Vue components
7. Add to admin navigation if needed
8. Write tests

### Managing Subscriptions
1. Create/update Stripe products in `/admin/stripe-products`
2. Set pricing and features
3. Enable/disable products
4. Monitor in subscription dashboard

### Sending Push Notifications
1. Access `/admin/notifications/push`
2. Compose notification
3. Select target audience
4. Send or schedule
5. Monitor delivery

## Migration Best Practices

### Naming Conventions
- Use descriptive names: `2025_01_15_create_testimonials_table.php`
- Name indexes under 64 chars: `disc_redemptions_unique`
- Check column existence before adding
- Write proper rollback methods

### Common Pitfalls to Avoid
- Long auto-generated index names
- Composite indexes on string columns exceeding key length
- Missing down() methods
- Raw SQL for enum changes
- Incorrect migration ordering

## Testing

### Run Tests
```bash
# All tests
php artisan test

# Specific suite
php artisan test --testsuite=Feature

# With coverage
php artisan test --coverage

# Specific test
php artisan test --filter=SubscriptionTest
```

### Test Database
```bash
# Use separate test database
DB_CONNECTION=mysql_test
DB_DATABASE=wewingames_test
```

## Troubleshooting

### Common Issues

1. **419 CSRF Error**
   ```bash
   php artisan optimize:clear
   # Clear browser cookies
   # Check SESSION_DOMAIN
   ```

2. **Vite Connection Error**
   ```bash
   npm run build
   # Or restart: npm run dev
   ```

3. **Migration Errors**
   ```bash
   # Check migration status
   php artisan migrate:status
   
   # Fix key length for old MySQL
   # Already handled in AppServiceProvider
   ```

4. **Email Not Sending**
   ```bash
   # Check MAIL_MAILER in .env
   # Use 'log' for testing
   # Check storage/logs/laravel.log
   ```

5. **Stripe Webhook Errors**
   ```bash
   # Verify webhook secret
   # Check Stripe logs
   # Test with Stripe CLI
   ```

## Recent Updates (April 2026)

### New Features
1. **Quick Checkout**: Payment-first registration flow for higher conversion
2. **OneSignal Integration**: Push notifications for non-admin pages
3. **Enhanced Media Library**: Centralized file management
4. **Knowledgebase System**: Help articles and categories
5. **Job Board**: Career opportunities management
6. **Affiliate System**: Commission tracking
7. **Activity Logging**: Comprehensive user tracking

### Improvements
1. **Performance**: Reduced query counts with eager loading
2. **Security**: Enhanced middleware and validation
3. **UX**: Bootstrap 5 migration for consistency
4. **Admin**: Redesigned dashboard with better analytics
5. **SEO**: Improved meta tags and structured data

### Bug Fixes
1. Fixed 419 CSRF errors on file uploads
2. Resolved email verification 500 errors
3. Corrected TypeScript null checks
4. Fixed admin cache clearing 403 error
5. Improved form validation messages

## Support & Documentation

- Laravel Docs: https://laravel.com/docs/12.x
- Vue.js Docs: https://vuejs.org/
- Inertia Docs: https://inertiajs.com/
- Bootstrap Docs: https://getbootstrap.com/docs/5.3/
- Stripe Docs: https://stripe.com/docs

## Contact

For questions or issues:
- Use in-app support system at `/support`
- Check activity logs in admin dashboard
- Review Laravel Telescope for debugging

===

<laravel-boost-guidelines>
=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to enhance the user's satisfaction building Laravel applications.

## Foundational Context
This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.3.30
- inertiajs/inertia-laravel (INERTIA) - v2
- laravel/framework (LARAVEL) - v12
- laravel/nightwatch (NIGHTWATCH) - v1
- laravel/prompts (PROMPTS) - v0
- tightenco/ziggy (ZIGGY) - v2
- laravel/pint (PINT) - v1
- @inertiajs/vue3 (INERTIA) - v2
- vue (VUE) - v3


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
- The 'search-docs' tool is perfect for all Laravel related packages, including Laravel, Inertia, Livewire, Filament, Tailwind, Pest, Nova, Nightwatch, etc.
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


=== herd rules ===

## Laravel Herd

- The application is served by Laravel Herd and will be available at: https?://[kebab-case-project-dir].test. Use the `get-absolute-url` tool to generate URLs for the user to ensure valid URLs.
- You must not run any commands to make the site available via HTTP(s). It is _always_ available through Laravel Herd.


=== inertia-laravel/core rules ===

## Inertia Core

- Inertia.js components should be placed in the `resources/js/Pages` directory unless specified differently in the JS bundler (vite.config.js).
- Use `Inertia::render()` for server-side routing instead of traditional Blade views.

<code-snippet lang="php" name="Inertia::render Example">
// routes/web.php example
Route::get('/users', function () {
    return Inertia::render('Users/Index', [
        'users' => User::all()
    ]);
});
</code-snippet>


=== inertia-laravel/v2 rules ===

## Inertia v2

- Make use of all Inertia features from v1 & v2. Check the documentation before making any changes to ensure we are taking the correct approach.

### Inertia v2 New Features
- Polling
- Prefetching
- Deferred props
- Infinite scrolling using merging props and `WhenVisible`
- Lazy loading data on scroll

### Deferred Props & Empty States
- When using deferred props on the frontend, you should add a nice empty state with pulsing / animated skeleton.


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


=== inertia-vue/core rules ===

## Inertia + Vue

- Vue components must have a single root element.
- Use `router.visit()` or `<Link>` for navigation instead of traditional links.

<code-snippet lang="vue" name="Inertia Client Navigation">
    import { Link } from '@inertiajs/vue3'

    <Link href="/">Home</Link>
</code-snippet>

- For form handling, use `router.post` and related methods. Do not use regular forms.


<code-snippet lang="vue" name="Inertia Vue Form Example">
    <script setup>
    import { reactive } from 'vue'
    import { router } from '@inertiajs/vue3'
    import { usePage } from '@inertiajs/vue3'

    const page = usePage()

    const form = reactive({
      first_name: null,
      last_name: null,
      email: null,
    })

    function submit() {
      router.post('/users', form)
    }
    </script>

    <template>
        <h1>Create {{ page.modelName }}</h1>
        <form @submit.prevent="submit">
            <label for="first_name">First name:</label>
            <input id="first_name" v-model="form.first_name" />
            <label for="last_name">Last name:</label>
            <input id="last_name" v-model="form.last_name" />
            <label for="email">Email:</label>
            <input id="email" v-model="form.email" />
            <button type="submit">Submit</button>
        </form>
    </template>
</code-snippet>


=== tests rules ===

## Test Enforcement

- Every change must be programmatically tested. Write a new test or update an existing test, then run the affected tests to make sure they pass.
- Run the minimum number of tests needed to ensure code quality and speed. Use `php artisan test` with a specific filename or filter.
</laravel-boost-guidelines>

---
> Source: [WeWinGames1/SITE-WeWinGames](https://github.com/WeWinGames1/SITE-WeWinGames) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
