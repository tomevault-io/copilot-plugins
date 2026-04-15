## coffeeriders

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CoffeeRiders is a Laravel 12 + Vue 3 + Inertia.js application for a cycling business offering:
- **Bike Rental** (`/rent-bikes`) - bicycle rental with booking system
- **Cycling Studio** (`/cycling-studio`) - indoor cycling activity booking (Zwift-style)
- **Admin Panel** (`/adminka`) - management dashboard for bikes, bookings, users, and coupons

## Common Commands

```bash
# Development (starts server, queue, logs, and vite concurrently)
composer dev

# Development with SSR
composer dev:ssr

# Run tests
composer test

# Lint and format frontend
npm run lint          # ESLint with auto-fix
npm run format        # Prettier
npm run format:check  # Prettier check only

# Build frontend
npm run build
npm run build:ssr

# Laravel code style (PHP)
./vendor/bin/pint
```

## Architecture

### Backend (Laravel 12)
- **Routes**: Split across multiple files in `routes/` - `web.php` includes other route files (`admin.php`, `cycling-studio.php`, `rent-bikes.php`, `tinkoff.php`)
- **Admin routes**: Prefixed with `/adminka`, protected by `admin` middleware
- **Auth middleware**: Custom `authed` middleware for authenticated users, `admin` middleware for admin-only access
- **User roles**: Defined in `App\Enums\Role` (ADMIN, REGULAR_USER)
- **Custom helpers**: `app/helpers.php` - auto-loaded, contains `verifyTelegramInitData()`

### Key Services
- `TinkoffService` - Payment processing via Tinkoff API (Russian payment provider)
- `AdminTelegram` - Telegram bot notifications for bookings (separate bots for prokat/studio/admin)
- `PricingService` / `CouponService` - Pricing and discount logic
- `BikeRentService` / `ImageService` - Bike management utilities

### Frontend (Vue 3 + TypeScript)
- **Entry**: `resources/js/app.ts` (SSR: `resources/js/ssr.ts`)
- **Path alias**: `@` maps to `resources/js/`
- **Pages**: `resources/js/pages/` - Inertia pages matching route structure
- **Layouts**: `resources/js/layout/` - `AppLayout.vue`, `AuthLayout.vue`
- **UI Components**: `resources/js/components/ui/` - shadcn/ui style components (reka-ui based)
- **Shared data**: Auth user, flash messages, Ziggy routes available via Inertia

### Database
- SQLite for testing, typically MySQL/PostgreSQL in production
- Models: `User`, `Bike`, `BikeBooking`, `BikeImage`, `CyclingActivity`, `CyclingStation`, `CyclingOrder`, `Coupon`, `CouponUsage`

## Environment & Config
- Telegram bot tokens: `TELEGRAM_BOT_TOKEN` and configs in `config/telegram.php`
- Tinkoff payment: `services.tinkoff.terminal_key`, `services.tinkoff.secret_key`
- Payments logged to `payments` channel

## Language
- The application UI and content is primarily in Russian
- Admin notifications via Telegram are in Russian

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PttRulez)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PttRulez)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
