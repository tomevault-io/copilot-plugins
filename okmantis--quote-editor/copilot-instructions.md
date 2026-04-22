## quote-editor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Quote Editor is a Rails 7.2 Progressive Web App using PostgreSQL, Hotwire (Turbo + Stimulus), and esbuild/Sass for asset bundling.

- **Ruby**: 3.3.5 (see `.ruby-version`)
- **Node**: 20.19.6 (see `.node-version`)
- **Database**: PostgreSQL

## Essential Commands

```bash
# Initial setup
bin/setup

# Start development (Rails server + JS/CSS watchers on port 3000)
bin/dev

# Run all tests
bin/rails test test:system

# Run unit tests only
bin/rails test

# Run system tests only (requires Chrome)
bin/rails test:system

# Run a single test file
bin/rails test test/models/some_test.rb

# Linting
bin/rubocop        # Check style
bin/rubocop -A     # Auto-fix

# Security scan
bin/brakeman --no-pager

# Database
bin/rails db:migrate
bin/rails db:test:prepare
```

## Architecture

### Frontend Stack
- **Turbo**: SPA-like navigation, imported in `app/javascript/application.js`
- **Stimulus**: Controllers in `app/javascript/controllers/`, auto-registered via `index.js`
- **Assets**: esbuild outputs to `app/assets/builds/application.js`, Sass to `application.css`

### Key Directories
- `app/javascript/controllers/` - Stimulus controllers (generate with `bin/rails generate stimulus name`)
- `app/views/pwa/` - PWA manifest and service worker templates
- `config/routes.rb` - URL routing (health check at `/up`)

### Browser Requirements
ApplicationController enforces modern browsers supporting: webp, web push, badges, import maps, CSS nesting, CSS :has.

## CI/CD

GitHub Actions runs on push to `main` and PRs:
1. **scan_ruby** - Brakeman security scan
2. **lint** - RuboCop with Omakase styling
3. **test** - Full test suite with PostgreSQL service

## Code Style

Uses `rubocop-rails-omakase` for opinionated Rails styling. Configuration in `.rubocop.yml`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OKMantis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
