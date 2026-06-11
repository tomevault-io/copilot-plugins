## sploder-revival

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sploder Revival is a PHP-based web application that powers an online Flash game creation platform. Users can create various types of games (arcade, platformer, shooter, physics puzzles, etc.) and share them with the community.

## Development Commands

### Initial Setup
```bash
composer install              # Install PHP dependencies
make build                    # Build the Docker image for development
make dev.bootstrap            # Bootstrap database with test data (run once)
make dev.hook                 # Install pre-commit formatting hooks
```

### Running the Application
```bash
make dev                      # Start development environment (detached)
make dev.watch                # Start development environment (attached, see logs)
make dev.down                 # Stop development environment
```

Development server runs at: http://127.0.0.1:8010

### Testing and Code Quality
```bash
make test                     # Run PHPUnit tests (./vendor/bin/phpunit)
./vendor/bin/phpcs            # Run PHP CodeSniffer (PSR-12 standard)
./vendor/bin/phpcbf           # Auto-fix code style issues
```

### Database Operations
```bash
make dev.bash.db              # Access database container
make dev.backup.db            # Create schema backup to db/sploder.sql
```

**Important**: After schema changes, always run `make dev.backup.db` to update the schema dump used for bootstrapping.

### Container Access
```bash
make dev.bash.site            # Access web application container
make dev.bash.db              # Access PostgreSQL container
```

### Production Commands
```bash
make build.prod               # Build production image
make prod.bootstrap           # Bootstrap production database (schema only)
make prod                     # Start production environment
make prod.down                # Stop production environment
make prod.backup.db           # Create full database backup
make prod.logs                # View production logs
```

Production server runs at: http://127.0.0.1:8020

## Architecture

### Database Layer

The application uses a **Repository Pattern** with a centralized singleton manager:

- **DatabaseManager** (`src/database/databasemanager.php`): Manages database connections
  - PostgreSQL (primary database for all data)
  - SQLite (legacy members database for migration)
  - Accessed via `DatabaseManager::get()`

- **RepositoryManager** (`src/repositories/repositorymanager.php`): Provides access to all repositories
  - Singleton pattern: `RepositoryManager::get()`
  - Available repositories:
    - `getGameRepository()` - Game data and metadata
    - `getUserRepository()` - User accounts and profiles
    - `getGraphicsRepository()` - Custom graphics/sprites
    - `getAwardsRepository()` - User awards and achievements
    - `getContestRepository()` - Contest information
    - `getFriendsRepository()` - Friend relationships
    - `getChallengesRepository()` - User challenges

All repositories implement interfaces (prefixed with `I`) and accept an `IDatabase` instance.

### Directory Structure

- `src/php/` - API endpoints for Flash games (save/load game data, user status, leaderboards)
- `src/repositories/` - Database repository layer (interfaces and implementations)
- `src/services/` - Business logic services (rendering, game feeds, challenges)
- `src/database/` - Database abstraction and connection management
- `src/content/` - Shared PHP includes for page components (headers, navigation, initialization)
- `src/make/` - Game creation interfaces for different game types
- `src/games/` - Game playing and browsing pages
- `src/accounts/` - User authentication and account management
- `src/dashboard/` - User dashboard and profile pages
- `src/swf/` - Flash SWF files for game creators and players
- `tests/` - PHPUnit tests (suffix: `*Tests.php`)

### Configuration

Environment configuration is managed through `.env` files:
- Copy `.env.example` to `src/.env` for manual deployments
- Docker deployments use environment variables from docker-compose files
- All configuration is loaded via `src/config/env.php`

Key environment variables:
- `POSTGRES_*` - Database connection settings
- `ORIGINAL_MEMBERS_DB` - Legacy SQLite database name
- `PHP_ENVIRONMENT` - `development` or `production` (affects error handling)
- `SPLODERHEADS_ENABLED` - Feature flag for multiplayer games
- `SWITCH` / `SWITCH_TIMER` - Maintenance mode killswitch

### Testing

- Tests are located in `tests/` directory
- Test files must end with `Tests.php` (e.g., `RepositoryManagerTests.php`)
- Bootstrap: `vendor/autoload.php`
- Run with: `make test` or `./vendor/bin/phpunit`

### Code Style

- Standard: PSR-12 (enforced via `phpcs.xml`)
- Pre-commit hook automatically runs `phpcbf` to fix formatting issues
- Install hook with: `make dev.hook`

## Common Workflows

### Adding a New Repository

1. Create interface in `src/repositories/i{name}repository.php`
2. Create implementation in `src/repositories/{name}repository.php`
3. Add to `RepositoryManager` class:
   - Add private readonly property
   - Initialize in constructor
   - Add getter method
   - Add `require_once` at top of file

### Working with the Database

Access repositories through the singleton manager:

```php
require_once(__DIR__ . '/repositories/repositorymanager.php');

$repos = RepositoryManager::get();
$gameRepo = $repos->getGameRepository();
```

### Running a Single Test

```bash
./vendor/bin/phpunit tests/repositories/RepositoryManagerTests.php
```

## Notes

- The codebase is **maintenance-only** - new features are limited, focus is on PHP version compatibility
- Flash Player is required for development (see README.md for compatible browsers)
- Pre-commit hooks ensure code style consistency
- Always update database schema dump after migrations

---
> Source: [Sploder-Saptarshi/sploder-revival](https://github.com/Sploder-Saptarshi/sploder-revival) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
