## quest100

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Ruby on Rails 8.0.2 application called "My Academy Quest" - a quest management system where users can create, manage, and track their academic progress. The app uses Turbo Streams for real-time updates and includes a brag page for celebrating completed quests.

## Development Commands

### Testing
- `bundle exec rspec` - Run RSpec test suite
- `bundle exec rspec spec/models` - Run model specs only
- `bundle exec rspec spec/requests` - Run request specs only
- `bundle exec rspec spec/routing` - Run routing specs only
- `bundle exec cucumber` - Run Cucumber acceptance tests

### Linting and Code Quality
- `bundle exec rubocop` - Run RuboCop linter (configured with Rails, Performance, RSpec plugins)
- `bundle exec rubocop -a` - Auto-correct RuboCop violations
- `bundle exec brakeman` - Run security analysis

### Database
- `bundle exec rails db:migrate` - Run database migrations
- `bundle exec rails db:setup` - Setup database from scratch
- `bundle exec rails db:reset` - Drop, recreate and seed database

### Asset Pipeline
- `bundle exec rails tailwindcss:build` - Build Tailwind CSS
- `bundle exec rails tailwindcss:watch` - Watch and rebuild Tailwind CSS

### Development Server
- `bundle exec rails server` - Start development server
- `bin/dev` - Start development with Procfile (if using foreman)

## Architecture

### Models
- **Quest** (`app/models/quest.rb`): Core model with name validation and boolean status field. Automatically sets status to false for new records via after_initialize callback.

### Controllers
- **QuestsController** (`app/controllers/quests_controller.rb`): Handles CRUD operations for quests with Turbo Stream responses. Includes toggle_status action for status updates.
- **BragController** (`app/controllers/brag_controller.rb`): Simple controller for the brag/celebration page.

### Key Features
- **Turbo Streams**: Real-time UI updates without page refreshes for create, destroy, and toggle operations
- **Responsive Design**: Uses Tailwind CSS with mobile-first approach
- **Quest Management**: Create quests, toggle completion status, delete quests

### Routes Structure
- Root route: `quests#index`
- Quest resources: index, create, destroy only
- Quest member route: `toggle_status` (PATCH)
- Brag page: `/brag` -> `brag#index`

### Database Schema
- **quests** table: id, name (string, required), status (boolean, defaults to false), timestamps

### Frontend Stack
- **Stimulus**: JavaScript framework for interactive behaviors
- **Turbo**: SPA-like navigation and real-time updates
- **Tailwind CSS**: Utility-first CSS framework
- **Importmap**: JavaScript module management

## Test Coverage

The application has comprehensive test coverage:
- **RSpec**: Unit tests for models, request tests, routing tests
- **Cucumber**: Acceptance tests for user scenarios including responsive design
- **SimpleCov**: Code coverage reporting

## Code Standards

- Ruby 3.2+ target version
- RuboCop configuration with Rails, Performance, RSpec, Capybara plugins
- Documentation requirement disabled
- Frozen string literals enabled across codebase
- Strong parameters using Rails 8's `expect` method

## Development Notes

- The app uses Rails 8's solid adapters (Solid Cache, Solid Queue, Solid Cable) for caching, background jobs, and ActionCable
- PostgreSQL database in all environments
- Security scanning with Brakeman
- No custom Cursor rules or GitHub Copilot instructions found

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KanditT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
