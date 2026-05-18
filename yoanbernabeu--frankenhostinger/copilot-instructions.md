## frankenhostinger

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FrankenHostinger is a Symfony 8.0 web application using PHP 8.4+. It uses Doctrine ORM with PostgreSQL, Twig templating, and Stimulus/Turbo for frontend interactivity via Symfony's Asset Mapper (no Node.js/webpack required).

## Common Commands

```bash
# Install dependencies
composer install

# Start development server
symfony server:start
# or without Symfony CLI:
php -S localhost:8000 -t public

# Database
docker compose up -d                          # Start PostgreSQL
bin/console doctrine:database:create          # Create database
bin/console doctrine:migrations:migrate       # Run migrations

# Testing
bin/phpunit                                   # Run all tests
bin/phpunit tests/path/to/TestFile.php        # Run single test file
bin/phpunit --filter testMethodName           # Run single test method

# Cache
bin/console cache:clear

# Code generation
bin/console make:controller ControllerName
bin/console make:entity EntityName
bin/console make:migration

# Debugging
bin/console debug:router                      # List routes
bin/console debug:container                   # List services
```

## Architecture

**Stack**: Symfony 8.0 / PHP 8.4+ / PostgreSQL 16 / Doctrine ORM / Twig / Stimulus + Turbo

**Key directories**:
- `src/Controller/` - HTTP request handlers
- `src/Entity/` - Doctrine ORM entities
- `src/Repository/` - Data access layer
- `config/packages/` - Bundle configurations (YAML)
- `templates/` - Twig templates
- `assets/` - Frontend JS/CSS (Stimulus controllers in `assets/controllers/`)
- `migrations/` - Database migrations
- `var/share/` - Application data storage (configurable via `APP_SHARE_DIR`)

**Entry points**:
- `public/index.php` - Web entry point
- `bin/console` - CLI commands
- `assets/app.js` - Frontend entry (imports Stimulus bootstrap)

**Environment configuration**: Uses `.env` files with precedence: `.env` → `.env.local` → `.env.{APP_ENV}` → `.env.{APP_ENV}.local`

## Testing

PHPUnit 12.5 configured with strict error handling (fails on deprecations, notices, warnings). Test environment uses `APP_ENV=test`. Tests live in `tests/` directory.

---
> Source: [yoanbernabeu/FrankenHostinger](https://github.com/yoanbernabeu/FrankenHostinger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
