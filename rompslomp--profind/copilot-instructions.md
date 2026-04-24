## profind

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Application Overview

**proFind** is a Rails 8.1 platform connecting homeowners with qualified service professionals. Core concepts:
- Users can be `basic`, `provider`, or `admin` (role enum)
- Providers go through an approval workflow (`provider_status` enum: not_requested → pending → approved/rejected)
- Only users where `approved_provider?` is true (role=provider AND provider_status=approved) can create Services; basic users can request Quotes
- Admin backoffice (`/admin`) manages users, provider approvals, and tags

## Running the Application

**Via Docker (recommended):**
```bash
cp .env.example .env
# Set RAILS_MASTER_KEY in .env from config/master.key
docker compose up
docker compose run --rm web bin/rails db:seed  # first run only
```

**Locally:**
```bash
bundle install && yarn install && yarn build:css
bin/rails db:create db:migrate db:seed
bin/dev  # starts Rails + CSS watcher via Procfile.dev
```

App runs at `http://localhost:3000`.

## Commands

```bash
# Tests
bundle exec rspec                    # full RSpec suite
bundle exec rspec spec/models/       # specific directory
bundle exec rspec spec/models/user_spec.rb  # single file
bundle exec rspec spec/models/user_spec.rb:42  # single example

# Linting & security
bin/rubocop -f github               # code style
bin/brakeman --no-pager             # Rails security scan
bin/bundler-audit                   # gem vulnerability audit

# Assets
yarn build:css                      # compile SCSS once
```

## Architecture

### Authorization Flow
- **Devise** handles authentication; `ApplicationController` runs `before_action :authenticate_user!` except on `index`. Controllers with public `show` actions (`ServicesController`, `TagsController`) add `skip_before_action :authenticate_user!, only: :show` explicitly
- **Pundit** policies (`app/policies/`) enforce authorization. `ServicePolicy` gates create/edit/destroy to approved providers and admins; `QuotePolicy` gates quote visibility to quote owner
- Admin controllers inherit from `Admin::BaseController` which checks `current_user.admin?`

### Key Models
- **User** — role (basic/provider/admin) + provider_status (not_requested/pending/approved/rejected) enums control feature access
- **Service** — belongs to a provider User; has many Tags (through ServiceTag); supports multiple image attachments via Active Storage
- **Quote** — links a basic User to a Service; status enum (pending/responded/closed)
- **Tag** — service categories (Electrician, Plumber, etc.); managed by admin

### Service Search (public, not Ransack)
`Service` has three custom scopes used in `ServicesController#index`:
- `.by_approved_providers` — filters to services from approved providers only
- `.search_by_tag(tag_id)` — filters by tag association
- `.search_by_text(query)` — case-insensitive search on title/description

Admin uses **Ransack** for user search in the backoffice.

### Admin Namespace
All routes under `/admin` map to `app/controllers/admin/` with `layout "admin"`. Key flows:
- `Admin::ProviderRequestsController` — approve sets `role=provider, provider_status=approved`; reject sets `provider_status=rejected`
- `Admin::TagsController` — CRUD for service categories

### CSS / Assets
- Bootstrap 5 + SCSS via `cssbundling-rails`; compiled via `yarn build:css`
- Propshaft (not Sprockets) for asset pipeline
- Hotwire (Turbo + Stimulus) for interactive behavior

## Test Setup

- RSpec with FactoryBot and Faker; factories in `spec/factories/`
- `spec/support/devise.rb` adds Devise test helpers (needed for controller/request specs)
- Transactional fixtures for DB cleanup
- System tests use Selenium/Capybara

**Seed credentials** (password: `password123`):
- `admin@profind.dev` — admin
- `john@prohandyman.dev` — approved provider
- `sarah@example.com` — basic user
- `mike@wannabepro.dev` — pending provider approval

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rompslomp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
