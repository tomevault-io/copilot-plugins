## olaindex

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OLAINDEX is a OneDrive directory index web application built on Laravel 8. It uses the Microsoft Graph API to browse and preview OneDrive content, supporting multiple accounts, SharePoint integration, and multiple themes.

## Commands

### Development

```bash
# Install dependencies
composer install

# Initialize application (creates database, .env, runs migrations)
composer run install-app
# Or: php artisan install

# Start development server
php artisan serve

# Clear cache
php artisan config:cache
php artisan cache:clear
```

### Testing

```bash
# Run all tests
vendor/bin/phpunit

# Run specific test suite
vendor/bin/phpunit --testsuite=Unit
vendor/bin/phpunit --testsuite=Feature
```

### Code Style

```bash
# Check code style (PSR-2)
composer run cs-check

# Fix code style
composer run cs-fix
```

## Architecture

### Core Service Layer

The `App\Service` namespace contains the Microsoft Graph API integration:

- **`OneDrive.php`** - Core service for all Graph API operations (list, fetch, upload, copy, move, delete, share)
- **`GraphClient.php`** - HTTP client wrapper for Graph API requests
- **`GraphRequest.php`** / **`GraphResponse.php`** - Request/response handling
- **`Client.php`** - OAuth configuration for COM (global) and CN (China) OneDrive accounts
- **`Constants.php`** - API endpoints for different OneDrive regions

### Controllers

- **`DriveController`** - Main controller handling file browsing, preview, download, search, and pagination
- **`AccountController`** - OneDrive account management
- **`AdminController`** - Admin panel settings
- **`ManageController`** - File management operations (upload, mkdir, delete)
- **`AuthController`** - OAuth callback handling

### Models

- **`Account`** - OneDrive account storage with access token management and auto-refresh
- **`Setting`** - Application settings with caching
- **`ShortUrl`** - Short URL mapping for sharing

### Key Patterns

1. **Account Token Refresh**: `Account::getAccessToken()` automatically refreshes expired tokens
2. **Caching**: Heavy use of Laravel Cache for API responses (configurable via `cache_expires` setting)
3. **Multi-tenancy**: Supports single and multi-account modes via `single_account_mode` setting
4. **Path Protection**: Supports password-protected directories via `encrypt_path` config and cookie-based auth

### Helper Functions

Global helpers in `app/Helpers/Functions.php`:
- `setting($key, $default)` - Get cached setting value
- `trans_request_path($path)` - Transform path encoding
- `trans_absolute_path($path)` - Normalize path format

## Database

Default: SQLite (`database/data/database.sqlite`). MySQL supported via `.env` configuration.

The install command copies `database.sample.sqlite` to `database.sqlite`.

## OAuth Flow

1. User binds OneDrive account via admin panel
2. OAuth callback handled by `AuthController@callback`
3. Tokens stored in `accounts` table with auto-refresh on expiry

## Themes

Two built-in themes in `resources/views/`:
- **default** - Standard Bootstrap-based theme
- **mdui** - Material Design UI theme

Configured via `main_theme` setting.

---
> Source: [WangNingkai/OLAINDEX](https://github.com/WangNingkai/OLAINDEX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
