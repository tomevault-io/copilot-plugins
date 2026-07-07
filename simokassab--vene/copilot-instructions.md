## vene

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Rails 8.0 e-commerce application for Vene Jewelry, featuring a bilingual (English/Arabic) storefront and admin panel. 
The application uses PostgreSQL, Tailwind CSS, and Hotwire (Turbo + Stimulus).

**Ruby Version:** 3.4.1
**Rails Version:** 8.0.3

## Development Commands

### Setup
```bash
bundle install
bin/rails db:create db:migrate db:seed
```

### Running the Application
```bash
# Start all services (web, CSS watching, Sidekiq worker)
bin/dev

# Or individually:
bin/rails server          # Web server on port 3000
bin/rails tailwindcss:watch  # CSS compilation
bundle exec sidekiq       # Background jobs
```

### Database
```bash
bin/rails db:migrate              # Run migrations
bin/rails db:rollback            # Rollback last migration
bin/rails db:seed                # Load seed data (jewelry products)
bin/rails db:reset               # Drop, create, migrate, seed
```

### Testing
```bash
bin/rails test                   # Run all tests
bin/rails test test/models/      # Run model tests only
bin/rails test test/models/product_test.rb  # Run specific test file
```

### Code Quality
```bash
bundle exec rubocop              # Check code style
bundle exec brakeman             # Security vulnerability scan
```

### Rails Console
```bash
bin/rails console               # Development console
bin/rails console --sandbox     # Sandbox mode (auto-rollback)
```

## Architecture Overview

### Multi-Locale Support
The application supports English (en) and Arabic (ar) via scoped routes with `(:locale)` prefix. Locale is determined by:
1. URL parameter `:locale`
2. Session `session[:locale]`
3. Default locale (I18n.default_locale)

Most models have `name_en`/`name_ar` and `description_en`/`description_ar` fields with helper methods `name(locale)` and `description(locale)`.

### Authentication & Authorization
- **Devise** for authentication with two separate user flows:
  - Customer registration/login: `/sign_up`, `/sign_in` (via `Customers::RegistrationsController`, `Customers::SessionsController`)
  - Admin login only: `/admin/sign_in` (via `Admin::SessionsController`)
- Admin users have `admin: true` boolean flag on User model
- Admin controllers inherit from `Admin::BaseController` which authenticates admin users
- Sidekiq Web UI mounted at `/admin/sidekiq` (requires admin authentication)

### Product Variants System
The application supports two variant approaches (transitional state):

1. **New Approach** (preferred): Uses `VariantType` and `VariantOption` models
   - `VariantType` (e.g., "Size", "Color") has many `VariantOption` (e.g., "Small", "Large", "Gold", "Silver")
   - `ProductVariant` links to both `variant_type` and `variant_option`
   - Managed via `/admin/variant_types` resource

2. **Legacy Approach**: Direct `name` and `value` fields on `ProductVariant`
   - Used for backward compatibility
   - Still validated and functional

Both approaches are validated in `ProductVariant#validate_variant_data`. Each variant has independent `stock_quantity` tracking.

### Stock Management
- Stock tracked at both `Product` level and `ProductVariant` level
- When creating an order:
  - Stock is decremented from the variant (if selected) or product
  - Done via `OrderItem` callbacks or checkout logic
- When canceling an order via `Order#cancel_order!`:
  - Stock is restored to variants/products using `increment!(:stock_quantity, quantity)`
- Always use database-level optimization for stock queries when dealing with large datasets

### Shopping Cart
Implemented as a session-based service (`Cart` class in `app/services/cart.rb`):
- Stores cart data in `session[:cart]` as a hash with keys like `"product_id"` or `"product_id_variant_id"`
- `Cart::Item` is a Data class (immutable struct) with `product`, `product_variant`, and `quantity`
- Cart methods: `add`, `update`, `remove`, `clear`, `items`, `subtotal`
- Cart loads products/variants in bulk to optimize database queries (uses `index_by(&:id)`)

### Order Processing Flow
1. User adds items to session-based cart
2. At checkout, `Storefront::CheckoutsController#create`:
   - Creates `Order` with order items
   - Calculates tax based on country (local vs international rates from `Setting`)
   - Calls `Order#update_totals!(settings)` to calculate subtotal, tax, shipping, total
   - Enqueues `OrderConfirmationJob` for email notification
   - Clears cart from session
   - Initiates payment via `Montypay::Client#start_payment`
3. Order statuses: `pending`, `payment_pending`, `paid`, `shipped`, `delivered`, `canceled`
4. Payment statuses: `pending`, `paid`, `failed`

### Payment Integration
- MontyPay integration via `app/services/montypay/client.rb`
- Payment webhook handled at `POST /storefront/payments/webhook`
- Success/failure redirect pages at `/storefront/payments/success` and `/storefront/payments/failure`

### Background Jobs
- **Sidekiq** for background job processing
- Jobs: `OrderConfirmationJob` (sends order confirmation emails)
- Configure Sidekiq in `config/initializers/sidekiq.rb`

### File Uploads
- **CarrierWave** for file uploads with **MiniMagick** for image processing
- Uploaders in `app/uploaders/`
- Models with uploads:
  - `Product` has `video` (via `ProductVideoUploader`)
  - `ProductImage` and `Category` likely have image uploaders
  - `Banner` model for promotional banners

### Admin Panel Structure
All admin controllers in `app/controllers/admin/`:
- Inherit from `Admin::BaseController` (authentication/authorization)
- Resources: products, categories, banners, variant_types, orders, pages, settings
- Product images managed as nested resource under products

### Ransack for Search/Filter
- Search gem integrated for filtering (see `Product.ransackable_attributes` and `Product.ransackable_associations`)
- Used in admin panels and potentially storefront search (`/search` route via `Storefront::SearchesController`)

### Pagination
- **Pagy** gem (v9.0) for pagination
- Configured in `config/initializers/pagy.rb`

### PDF Generation
- **Prawn** gem for generating PDF invoices
- Invoice routes: `GET /orders/:id/invoice` (storefront) and `GET /admin/orders/:id/invoice`
- Implemented via `InvoiceGenerator` service (`app/services/invoice_generator.rb`)

### Related Products
Products can have related products via `ProductRelation` join model:
- `Product#featured_related(limit = 4)` returns related products or falls back to same-category products
- Useful for "You may also like" sections

## Database Optimization Guidelines

When writing queries, always optimize for scale (assume 1M+ records):
- Use `includes`, `preload`, or `eager_load` to avoid N+1 queries
- Add database indexes for frequently queried columns
- Use `select` to limit columns when fetching large datasets
- Use `find_each` or `in_batches` for batch processing
- Use `pluck` when you only need specific column values
- Add `counter_cache` for associations that are counted frequently

## Code Principles

Follow SOLID, DRY, and KISS principles:
- Extract complex logic into service objects (`app/services/`)
- Keep controllers thin (delegate to models/services)
- Use scopes for reusable query logic
- Prefer composition over inheritance where appropriate

## Testing Strategy

- Standard Rails minitest framework
- Tests run in parallel (configured in `test/test_helper.rb`)
- Fixtures in `test/fixtures/`
- Model tests verify validations, associations, and key business logic
- System tests use Capybara + Selenium WebDriver

## Environment Variables

Required environment variables (see `config/database.yml` and Gemfile):
- `DATABASE_URL` or `DATABASE_HOST`, `DATABASE_PORT`, `DATABASE_USERNAME`, `DATABASE_PASSWORD`, `DATABASE_NAME`
- Configure via `.env` file (using `dotenv-rails` gem)

## Key Models Reference

- **User**: Devise authentication, has `admin` boolean, has_many orders
- **Product**: Belongs to category, has_many product_images, product_variants, order_items; has `featured` and `on_sale` flags
- **ProductVariant**: Variant with stock tracking, belongs to product, variant_type, variant_option
- **VariantType**: Has many variant_options (e.g., "Size", "Color")
- **VariantOption**: Belongs to variant_type, has many product_variants (e.g., "Small", "Gold")
- **Order**: Has many order_items, belongs to user; tracks status, payment_status, totals
- **OrderItem**: Join table with quantity, unit_price; belongs to order, product, optional product_variant
- **Category**: Has many products, has `image` field
- **Banner**: Promotional banners, optional link to product
- **Page**: CMS pages with slugs for static content
- **Setting**: Singleton model for site settings (tax rates, shipping, local country)

## Important Notes

- Always include locale in redirects/links: `redirect_to cart_path(locale: I18n.locale)`
- Use `current_settings` helper (from ApplicationController) to access site settings
- Use `cart_item_count` helper to display cart badge count
- Product prices: use `Product#current_price` (handles sale_price logic)
- Test emails in development open via letter_opener (no actual sending)

---
> Source: [simokassab/vene](https://github.com/simokassab/vene) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
