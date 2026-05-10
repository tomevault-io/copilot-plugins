## maecenas

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vietnamese manga/comic reading platform (truyện tranh) built with Laravel 10 + Livewire 2 + Tailwind CSS. The app supports comic browsing, reading chapters, user accounts with leveling/XP system, comments, voting, following, and an admin panel with crawling capabilities from external sources (OTruyen API).

## Common Commands

```bash
# Start development server
php artisan serve

# Frontend assets (Vite)
npm run dev          # Dev server with HMR
npm run build        # Production build

# Admin assets (Laravel Mix - for admin panel SCSS)
npx mix              # Compile admin SCSS to public/assets/css/argon-dashboard.css

# Database
php artisan migrate
php artisan migrate:fresh --seed

# Create admin user
php artisan create:admin

# Crawl comics from OTruyen API
php artisan crawl:otruyen {fromPage} {toPage}

# Cache management
php artisan cache:clear && php artisan route:clear && php artisan config:clear && php artisan view:clear

# Tests
php artisan test
./vendor/bin/phpunit --filter=TestName

# Code style (Laravel Pint)
./vendor/bin/pint
```

## Architecture

### Two-Panel Structure
- **User-facing frontend**: `resources/views/users/` — Blade views with Tailwind CSS, Livewire components
- **Admin panel**: `resources/views/pages/` + `resources/views/layouts/` — Uses Argon Dashboard theme (SCSS compiled via Laravel Mix)

### Key Models & Relationships
- `Comic` → hasMany `Chapter`, belongsToMany `Category` (pivot: `comic_categories`), belongsToMany `Author` (pivot: `author_comic`), hasMany `Comment`, `Vote`, `Follow`, `History`
- `Chapter` → grouped by `chapter_number` with multiple `server` variants (multi-server chapter images)
- `User` → role-based (`role=1` for admin), XP/leveling system via `Level` model

### Routing Pattern
- Admin routes: `/admin/*` — protected by `checkLogin` + `checkAdmin` middleware chain
- Public routes: Vietnamese slug-based URLs (`/the-loai/{slug}`, `/{slug}/{chapter}`)
- Catch-all `/{slug}` route for comic detail — must be registered last in `routes/web.php`
- API routes: `/api/*` — search, sitemap generation, chapter reporting

### Crawling System
- `Admin\CrawlController` — crawls comics/chapters from OTruyen API via admin panel
- `CrawlOTruyen` command — CLI batch crawl with page range
- External API configured via `LINK_OTRUYEN_API` env var
- Uses Goutte (web scraping) and Guzzle (HTTP client) for content fetching

### Asset Pipeline
- **Vite**: Compiles `resources/css/app.css` (Tailwind) + `resources/js/app.js` for frontend
- **Laravel Mix (webpack.mix.js)**: Compiles `resources/scss/argon-dashboard.scss` for admin panel
- **Static theme assets**: `public/assets/theme/css/style.css` + `public/assets/theme/js/custom.js` — loaded directly

### SEO
- Uses `artesaos/seotools` package — SEO settings stored in `seo` database table
- `Comic` model has `generateSeoTags()` method for structured data (JSON-LD, OpenGraph, Twitter Cards)
- Sitemap generation via `spatie/laravel-sitemap`

### Caching Strategy
- File-based caching (`CACHE_DRIVER=file`)
- Home page data (hot mangas, categories, rankings) cached with `Cache::remember()`
- Cache TTL configured via `TIME_CACHE` env var (default 3600s)

### Storage
- Supports local and AWS S3 (`league/flysystem-aws-s3-v3`) for image storage
- Configured via `FILESYSTEM_DISK` and `AWS_*` env vars

## Environment Variables (Key Custom Ones)

| Variable | Purpose |
|---|---|
| `LINK_OTRUYEN_API` | OTruyen API base URL for crawling |
| `TIME_CACHE` | Default cache TTL in seconds |
| `APP_STORAGE` | Storage path configuration |
| `GOOGLE_CLIENT_ID/SECRET` | Google OAuth (via Socialite) |

## Dark Mode

Tailwind dark mode uses `selector` strategy (`darkMode: 'selector'` in tailwind.config.js). Toggle by adding/removing `dark` class on a parent element.

## Custom Middleware

- `CheckLogin` — redirects unauthenticated users to home
- `CheckAdmin` — returns 404 if user `role != 1`

---
> Source: [jhin1m/Maecenas](https://github.com/jhin1m/Maecenas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
