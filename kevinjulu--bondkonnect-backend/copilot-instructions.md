## bondkonnect-backend

> The **BondKonnect API** is a Laravel 11 based service providing the core business logic, trading engine, and financial integrations for the BondKonnect platform.

# GEMINI.md - Backend Context: BondKonnect API

## Project Overview
The **BondKonnect API** is a Laravel 11 based service providing the core business logic, trading engine, and financial integrations for the BondKonnect platform.

## Core Technologies
- **Framework:** Laravel 11 (PHP 8.3+)
- **Server:** FrankenPHP (for production performance and integrated HTTPS).
- **Database:** MariaDB (Primary) & Redis (Cache, Session, Queue).
- **Auth:** Laravel Sanctum (Stateful SPA authentication).
- **Real-time:** Pusher/WebSockets integration.
- **Testing:** PHPUnit (Unit & Feature tests).

## Architecture & Infrastructure

### 1. Production Hardening (Railway)
- **Trusted Proxies:** Configured to trust all proxies (`*`) to handle Railway's edge SSL termination.
- **Session Security:**
  - `SESSION_DRIVER=redis` for persistent state across deployments.
  - `SESSION_DOMAIN=bondkonnect.up.railway.app` (Exact match for PSL compliance).
  - `SANCTUM_STATEFUL_DOMAINS` includes both public and internal mesh URLs.
- **Entrypoint:** Uses `docker/entrypoint.sh` for automated migrations and storage linking.

### 2. Key Modules
- **Rating System:** Peer-to-peer trading party validation and weighted credibility scores.
- **Financials:** 
  - Subscriptions: `/V1/financials`
  - Services: `/V1/services`
  - Payments: `/V1/payments` (M-Pesa/PayPal)
- **Credibility Service:** Dynamic score calculation based on professionalism, reliability, and settlement history.

## Building and Running
- **Local Dev:** `php artisan serve` (Port 8000)
- **Production:** Docker/FrankenPHP (Port 80)
- **Queue Worker:** `php artisan queue:work`
- **Scheduler:** `php artisan schedule:work`

## API Conventions
- **Normalization:** Backend expects normalized keys (e.g., `user_email`, `trans_id`).
- **Response Format:** Standard JSON envelopes with appropriate HTTP status codes.
- **Versioning:** Primary endpoints follow `/api/V1/` prefix.

---
*Last Updated: April 2026*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kevinjulu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kevinjulu)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
