## victory

> This file provides guidance for AI assistants working on this codebase.

# CLAUDE.md — АН "Виктори" Real Estate Platform

This file provides guidance for AI assistants working on this codebase.

## Project Overview

**АН "Виктори"** is a Ruby on Rails 7.1 digital platform for a Russian real estate agency. It provides property listings, search, client dashboards, online valuations, mortgage calculators, and CRM-style functionality.

- **Language**: Russian (primary UI), with locale support for `ru` and `en`
- **Framework**: Rails 7.1, Ruby 3.2.2
- **Database**: PostgreSQL 15+
- **App Module**: `ViktoryRealty`

## Current State

The application is in a **partially functional state** with several features temporarily disabled:

- **Active**: Landing page (`LandingController`), Properties CRUD, Property Valuations, basic routing
- **Disabled (commented out)**: Devise authentication, ActiveAdmin panel, Sidekiq background jobs, Rack::Attack rate limiting, Pundit authorization
- **Stub methods**: `user_signed_in?` always returns `false`; `current_user` always returns `nil`
- **Root route**: `landing#index` — a standalone minimalist landing page (no application layout, no gem dependencies)
- **Server port**: 5000 (configured for Replit environment)

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Web framework | Rails 7.1 |
| Language | Ruby 3.2.2 |
| Database | PostgreSQL (adapter: `pg`) |
| Server | Puma 6.x |
| CSS | Tailwind CSS (`tailwindcss-rails`) |
| JS bundling | Importmap + Stimulus + Turbo (Hotwire) |
| Auth | Devise (currently disabled) |
| Search | PgSearch (full-text, Russian dictionary) |
| Pagination | Kaminari |
| Background jobs | Sidekiq (disabled) / Active Job with `:async` adapter |
| API | Rack::CORS, jbuilder |
| Testing | RSpec, FactoryBot, Shoulda Matchers, DatabaseCleaner, Capybara |
| Linting | RuboCop (rubocop-rails, rubocop-rspec, rubocop-performance) |
| Geocoding | Geocoder gem |
| Scheduling | Whenever (cron) |
| WebSockets | Action Cable |
| File uploads | Active Storage |

## Directory Structure

```
victory/
├── app/
│   ├── controllers/
│   │   ├── application_controller.rb   # Base controller with many helper methods
│   │   ├── landing_controller.rb       # Active standalone landing page
│   │   ├── properties_controller.rb    # Property CRUD + map/search/compare
│   │   ├── property_valuations_controller.rb
│   │   ├── contact_forms_controller.rb
│   │   ├── dashboard_controller.rb
│   │   ├── home_controller.rb
│   │   ├── pages_controller.rb
│   │   └── api/
│   │       └── v1/                     # JSON API endpoints
│   ├── models/
│   │   ├── property.rb                 # Core listing model (rich scopes, enums, validations)
│   │   ├── user.rb                     # Devise model with roles (client/agent/admin)
│   │   ├── inquiry.rb
│   │   ├── favorite.rb
│   │   ├── message.rb
│   │   ├── review.rb
│   │   ├── document.rb
│   │   ├── price_history.rb
│   │   ├── property_valuation.rb
│   │   ├── property_type.rb
│   │   ├── property_view.rb
│   │   ├── saved_search.rb
│   │   └── viewing_schedule.rb
│   ├── services/
│   │   ├── property_evaluation_service.rb  # Market value calculation
│   │   ├── recommendation_service.rb
│   │   └── pdf_generator_service.rb
│   ├── jobs/
│   │   ├── send_viewing_reminders_job.rb
│   │   ├── update_property_statistics_job.rb
│   │   ├── inquiry_notification_job.rb
│   │   ├── viewing_notification_job.rb
│   │   ├── property_valuation_completed_job.rb
│   │   └── property_valuation_follow_up_job.rb
│   ├── mailers/
│   │   ├── inquiry_mailer.rb
│   │   ├── property_valuation_mailer.rb
│   │   └── viewing_mailer.rb
│   ├── channels/
│   │   └── chat_channel.rb
│   ├── javascript/
│   │   └── controllers/                # Stimulus controllers
│   │       ├── app_controller.js       # Global UI (mobile menu, scroll)
│   │       ├── chat_controller.js
│   │       ├── favorite_controller.js
│   │       ├── mortgage_calculator_controller.js
│   │       └── yandex_map_controller.js
│   └── views/
│       ├── layouts/                    # application, dashboard, mailer
│       ├── landing/index.html.erb      # Standalone landing page (complete HTML)
│       ├── properties/
│       ├── property_valuations/
│       └── shared/
├── config/
│   ├── routes.rb                       # Comprehensive route definitions
│   ├── application.rb                  # App-level config (timezone: Moscow, locale: ru)
│   ├── database.yml                    # PostgreSQL config via ENV vars
│   ├── puma.rb                         # Port 5000, 0.0.0.0 binding
│   ├── sidekiq.yml                     # 5 concurrency, queues: critical/mailers/default/scheduled/low_priority
│   ├── schedule.rb                     # Whenever cron jobs
│   └── environments/
│       ├── development.rb              # hosts.clear for Replit; letter_opener mailer
│       ├── test.rb
│       └── production.rb
├── db/
│   └── migrate/                        # 13 migrations covering all core tables
├── spec/
│   ├── rails_helper.rb                 # RSpec + FactoryBot + DatabaseCleaner setup
│   ├── spec_helper.rb
│   └── support/
│       ├── factory_bot.rb
│       ├── request_helpers.rb
│       └── shared_examples.rb
├── Gemfile
├── .rubocop.yml
└── .rspec
```

## Key Models

### Property
The central model. Key characteristics:
- **Enums**: `deal_type` (sale/rent/daily), `status` (draft/pending/active/sold/rented/archived/rejected), `condition` (needs_repair/normal/renovated/euro/designer)
- **Soft delete**: `deleted_at` column; `default_scope { not_deleted }`
- **Full-text search**: PgSearch with Russian dictionary on title/description/address/district
- **Geocoding**: Auto-geocodes on address change
- **FriendlyId**: URL slugs from title
- **Active Storage**: `images` and `floor_plans` attachments
- **Scopes**: `published`, `featured`, `for_sale`, `for_rent`, `by_price_asc`, `within_radius`, etc.
- **Key methods**: `publish!`, `unpublish!`, `soft_delete!`, `price_formatted`, `rooms_info`, `similar_to`

### User
Devise-based with roles:
- **Roles**: `client` (0), `agent` (1), `admin` (2) — enum with `_prefix: true`
- **OAuth**: Google OAuth2 and Yandex via `from_omniauth`
- **Soft delete**: `deleted_at` + `active` boolean
- **JSONB columns**: `notification_settings`, `preferences`
- **Associations**: properties, favorites, inquiries, saved_searches, messages, property_views, reviews, viewing_schedules

## Routing Structure

The routes file is extensive. Key namespaces:

| Namespace | Purpose |
|-----------|---------|
| `/` (root) | `landing#index` |
| `/properties` | Property CRUD + map/compare/search |
| `/dashboard/` | User dashboard (profile, favorites, inquiries, messages, settings) |
| `/sell/` | Property listing/evaluation for sellers |
| `/valuations/` | Online property valuation tool |
| `/services/` | Mortgage calculator, legal services, virtual tours |
| `/forms/`, `/contact_forms/` | Form submission endpoints |
| `/chat/` | Conversation and messaging |
| `/chatbot/` | AI chatbot messages |
| `/api/v1/` | JSON API (properties, auth, favorites, inquiries, saved_searches, recommendations) |
| `/webhooks/` | AmoCRM, Telegram, YooKassa webhooks |
| `/cable` | Action Cable WebSocket mount |
| `/health` | Health check endpoints |

## Development Workflow

### Running the App

```bash
# Start the Rails server (port 5000)
bundle exec rails server -p 5000 -b 0.0.0.0

# Or via bin/rails
bin/rails server
```

### Database

```bash
# Create databases
bin/rails db:create

# Run migrations
bin/rails db:migrate

# Seed data
bin/rails db:seed

# Reset (drop + create + migrate + seed)
bin/rails db:reset
```

Environment variables for database:
- `DATABASE_URL` (production, takes precedence)
- `DATABASE_HOST`, `DATABASE_PORT`, `DATABASE_USERNAME`, `DATABASE_PASSWORD`
- Database names: `viktory_realty_development`, `viktory_realty_test`, `viktory_realty_production`

### Running Tests

```bash
# Run all specs
bundle exec rspec

# Run specific file
bundle exec rspec spec/models/property_spec.rb

# Run with documentation format
bundle exec rspec --format documentation
```

Test setup:
- Uses `DatabaseCleaner` (transaction strategy, truncation for JS tests)
- `Faker::Config.locale = 'ru'` — Russian locale for test data
- FactoryBot syntax methods included globally
- Capybara with `selenium_chrome_headless` for JS specs

### Linting

```bash
# Run RuboCop
bundle exec rubocop

# Auto-correct offenses
bundle exec rubocop -a

# Auto-correct all (including unsafe)
bundle exec rubocop -A
```

RuboCop configuration highlights:
- Target Ruby 3.2
- Max line length: 120
- Max method length: 25
- String literals: single quotes
- `frozen_string_literal: true` required in all files
- `rubocop-rails`, `rubocop-rspec`, `rubocop-performance` plugins enabled
- `db/schema.rb`, migrations, and bin/ excluded

### Background Jobs (Sidekiq — currently disabled)

```bash
# Start Sidekiq (when enabled)
bundle exec sidekiq -C config/sidekiq.yml
```

Queues (priority order): `critical` → `mailers` → `default` → `scheduled` → `low_priority`

### Cron Jobs (Whenever)

```bash
# Update crontab
bundle exec whenever --update-crontab

# Clear crontab
bundle exec whenever --clear-crontab
```

Scheduled jobs:
- Every hour: `SendViewingRemindersJob`
- Daily at 03:00: `UpdatePropertyStatisticsJob`
- Daily at 10:00: `PropertyValuationFollowUpJob`

## Code Conventions

### Ruby / Rails

1. **Frozen string literals**: Every `.rb` file must start with `# frozen_string_literal: true`
2. **Single quotes**: Use single quotes for strings (RuboCop enforced)
3. **Service objects**: Plain Ruby classes in `app/services/`, called via `.call` or `#call` instance method
4. **Enums**: Always use `_prefix: true` option; comment Russian translations inline
5. **Scopes**: Chain-safe; use lambda syntax `-> { }` always
6. **Soft delete**: Use `deleted_at` + `default_scope { not_deleted }` pattern (not paranoia gem)
7. **Callbacks**: Keep them minimal; prefer explicit method calls in controllers/services
8. **Localization**: All user-facing strings should use `I18n.t()`. Default locale is `:ru`
9. **Comments**: Section headers use the `# ====...====` banner style found throughout models

### Controllers

- `ApplicationController` provides many helper methods: pagination (`per_page`), breadcrumbs, UTM capture, device detection, JWT decoding
- Respond to both HTML and JSON formats using `respond_to` blocks
- Use `render_success` / `render_error` helpers for consistent responses
- Authorization via `require_admin!` / `require_agent!` guards (Pundit disabled but stubs exist)

### JavaScript (Stimulus)

- Controllers live in `app/javascript/controllers/`
- Follow Stimulus conventions: `static targets`, `connect()`, `disconnect()` lifecycle
- Use `data-controller`, `data-action`, `data-target` attributes in ERB views

### Database

- Use `bigint` primary keys (Rails default)
- Add indexes for all foreign keys, enum columns, and commonly filtered columns
- Prefer `decimal` over `float` for monetary values
- Use `jsonb` for flexible preference/settings storage
- Geographic queries use PostGIS `earth_distance` / `ll_to_earth` functions

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `RAILS_ENV` | `development` | Environment |
| `PORT` | `5000` | Server port |
| `DATABASE_URL` | — | Full PostgreSQL URL (production) |
| `DATABASE_HOST` | `localhost` | DB host |
| `DATABASE_PORT` | `5432` | DB port |
| `DATABASE_USERNAME` | `postgres` | DB username |
| `DATABASE_PASSWORD` | `` | DB password |
| `REDIS_URL` | `redis://localhost:6379/0` | Redis URL |
| `RAILS_MAX_THREADS` | `5` | Thread count / DB pool size |
| `WEB_CONCURRENCY` | `1` | Puma worker count |
| `ACTION_CABLE_URL` | `ws://localhost:3000/cable` | WebSocket URL |
| `CORS_ORIGINS` | `*` | Allowed CORS origins (comma-separated) |
| `APP_HOST` | `localhost` | Application hostname |
| `APP_PROTOCOL` | `http` | Application protocol |
| `JWT_SECRET_KEY` | (credentials) | JWT signing key |
| `ASSET_HOST` | — | CDN host for assets |
| `LOG_LEVEL` | `debug`/`info` | Log verbosity |
| `SESSION_TIMEOUT` | `1800` | Session expiry in seconds |

## API Structure

REST JSON API under `/api/v1/`:

- `POST /api/v1/auth/login` — authenticate and receive JWT
- `GET /api/v1/properties` — list with search/filter
- `GET /api/v1/properties/:id` — single property
- `GET /api/v1/properties/featured` — featured listings
- `GET /api/v1/properties/:id/similar` — similar properties
- `GET/POST/DELETE /api/v1/favorites` — manage favorites
- `GET/POST/SHOW /api/v1/inquiries` — inquiries
- `GET/POST/DELETE/PATCH /api/v1/saved_searches` — saved searches
- `POST /api/v1/mortgage_calculator/calculate` — mortgage calculation
- `GET /api/v1/recommendations` — personalized recommendations

## Important Notes for AI Assistants

1. **Authentication is disabled**: `current_user` returns `nil` and `user_signed_in?` returns `false`. Do not assume Devise is active. Before re-enabling, add the Devise gem back to `Gemfile`.

2. **Many routes point to non-existent controllers**: The routes file is aspirational. Controllers like `BlogController`, `NewsController`, `SitemapController`, `RobotsController`, `PwaController`, `HealthController`, `ErrorsController`, `ChatController`, `ChatbotController`, `SellController`, `ServicesController`, `FormsController`, `WebhooksController` are defined in routes but may not have corresponding controller files yet.

3. **The landing page is self-contained**: `app/views/landing/index.html.erb` contains complete HTML (no application layout). It uses Tailwind CSS from CDN.

4. **Replit-specific config**: `config.hosts.clear` in development allows all hosts. Port 5000 is hardcoded for the Replit proxy. Keep these when working in that environment.

5. **Russian locale**: All user-facing text, error messages, and I18n keys are in Russian. Locale files are in `config/locales/ru.yml` and `config/locales/devise.ru.yml`.

6. **Ransack for search**: `PropertiesController#index` uses Ransack (`@q = Property.published.ransack(params[:q])`). The `ransackable_attributes`, `ransackable_associations`, and `ransackable_scopes` class methods on `Property` control what is searchable.

7. **Soft deletes**: Both `User` and `Property` use `deleted_at` with `default_scope { not_deleted }`. Use `unscoped` to include deleted records when needed.

8. **Price formatting**: Use `property.price_formatted` (returns formatted string with `₽`). Never format prices manually.

9. **Service objects pattern**: Services in `app/services/` take their main argument in `initialize` and expose a `call` method. Use `service.call` and check the `result` hash.

10. **No Pundit yet**: Authorization guards (`require_admin!`, `require_agent!`) exist in `ApplicationController` but Pundit is commented out. Policy objects do not exist yet.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nick4man)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nick4man)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
