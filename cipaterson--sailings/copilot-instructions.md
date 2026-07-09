## sailings

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

`sailings` is a Rails 8.1 application using SQLite, Hotwire (Turbo + Stimulus), importmap, Propshaft, Solid Queue, Solid Cache, and Solid Cable. Ruby version: 3.4.2.

## Common Commands

```bash
# Start server
bin/rails server

# Run all tests
bin/rails test

# Run a single test file
bin/rails test test/models/foo_test.rb

# Run a single test by line number
bin/rails test test/models/foo_test.rb:42

# Run system tests
bin/rails test:system

# Database
bin/rails db:create db:migrate
bin/rails db:migrate

# Console
bin/rails console

# Lint
bin/rubocop

# Security audit
bin/brakeman
bundle exec bundler-audit
```

## Architecture

**Stack:** Rails 8.1, SQLite (all environments), Propshaft (asset pipeline), importmap (no Node/bundler), Hotwire (Turbo + Stimulus).

**Background jobs:** Solid Queue (runs inside Puma in development/production via `SOLID_QUEUE_IN_PUMA`). Define jobs in `app/jobs/`.

**Caching:** Solid Cache (database-backed). **WebSockets/ActionCable:** Solid Cable (database-backed).

**Production:** Four SQLite databases — primary, cache, queue, and cable — all stored in `storage/`. Deployed via Kamal (Docker).

**JavaScript:** ES modules via importmap (`config/importmap.rb`). Stimulus controllers live in `app/javascript/controllers/`. No npm/yarn/bun.

**Styling:** `app/assets/stylesheets/application.css`. No CSS preprocessor configured by default.

**Testing:** Minitest (Rails default), parallel test execution via `parallelize(workers: :number_of_processors)`, fixtures enabled. System tests use Capybara + Selenium.

**Linting:** RuboCop with `rubocop-rails-omakase` (Rails' opinionated style guide). Config in `.rubocop.yml`.

---
> Source: [cipaterson/sailings](https://github.com/cipaterson/sailings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
