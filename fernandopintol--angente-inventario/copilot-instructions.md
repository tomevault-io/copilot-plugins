## angente-inventario

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Laravel 12 + React + Inertia.js** inventory management system for "Distribuidora Paucara" with multi-channel sales support (Web, Mobile App, Presential). The system handles inventory, sales, purchases, customer management, and logistics with automated stock management and accounting integration.

## Development Commands

### Backend (Laravel)
- **Serve application**: `composer run dev` (starts Laravel server + queue worker + Vite dev server concurrently)
- **Alternative serve**: `php artisan serve`
- **Run tests**: `composer run test` (clears config and runs tests)
- **Queue worker**: `php artisan queue:listen --tries=1`
- **Run single test**: `php artisan test --filter=<TestName>`
- **Clear cache**: `php artisan config:clear`
- **Database migrations**: `php artisan migrate`
- **Database seeding**: `php artisan db:seed`

### Frontend (React + TypeScript)
- **Development**: `npm run dev` (Vite dev server)
- **Build production**: `npm run build`
- **Build SSR**: `npm run build:ssr`
- **Lint and fix**: `npm run lint`
- **Format code**: `npm run format`
- **Type checking**: `npm run types`
- **Format check**: `npm run format:check`

## Architecture Overview

### Backend Structure
- **Framework**: Laravel 12 with Inertia.js
- **Database**: SQLite for development (database/database.sqlite)
- **Authentication**: Laravel Sanctum + Spatie Permission
- **Queue System**: Database-driven queues for background tasks
- **Testing**: Pest PHP testing framework

### Multi-Channel Sales System
The system supports three sales channels:
- **WEB**: Admin web interface for direct sales
- **APP_EXTERNA**: Mobile app external sales (proforma → approval → venta)
- **PRESENCIAL**: Point-of-sale direct sales

### Key Business Logic

#### Sales Flow (Flujo de Ventas)
1. **Proforma Creation** (Optional): Customers can create quotes via mobile app
2. **Approval Process**: Admin approves/rejects proformas
3. **Conversion to Sale**: Approved proformas convert to sales
4. **Stock Management**: Automatic stock movements based on shipping requirements
5. **Payment Processing**: Multiple payment types (cash, transfer, credit, etc.)

#### Stock Management
- **Automatic Number Generation**: Sales get auto-generated numbers like `VEN202509150001`
- **Default Currency**: BOB (Bolivianos) is automatically set (ID: 1)
- **Stock Reservation**: Proformas reserve stock temporarily
- **Stock Validation**: Always validate before sale confirmation
- **Movement Tracking**: All stock changes tracked in MovimientoInventario

### Frontend Structure
- **Framework**: React 19 + TypeScript
- **Routing**: Inertia.js for SPA-like experience
- **UI Components**: Custom components + Radix UI + Tailwind CSS
- **State Management**: React hooks + Inertia.js shared state
- **Build Tool**: Vite with Laravel integration

### Important Models and Relationships
- **Venta** (Sales): Core sales model with automatic stock processing
- **Proforma**: Quote system for mobile app integration
- **StockProducto**: Product inventory per warehouse
- **MovimientoInventario**: All stock movements tracking
- **Cliente**: Customer management
- **Producto**: Product catalog with multiple pricing tiers

### API Design
- Dual response system: Web requests return Inertia responses, API requests return JSON
- Controllers check `request()->expectsJson()` or `request()->is('api/*')` to determine response type
- RESTful endpoints with consistent ApiResponse helper for JSON responses

### Key Services
- **StockService**: Centralized stock validation and movement processing
- **VentaService**: Sales business logic and validation
- **DashboardService**: Metrics and analytics
- **SidebarService**: Dynamic navigation management

### Permission System
Uses Spatie Laravel Permission with role-based access control:
- Middleware protection on all routes
- Granular permissions for each module (ventas.index, productos.manage, etc.)
- Dynamic sidebar based on user permissions

### Important Conventions
- **Automated Fields**: `numero` and `moneda_id` are auto-generated/set in sales
- **Hidden Form Fields**: Number and currency fields are hidden in frontend forms
- **Stock Validation**: Always validate stock before any transaction
- **Dual Language**: Spanish language pack installed (laraveles/spanish)
- **Currency System**: Multi-currency support with BOB as default

### Development Notes
- Form requests validate optional `numero` and `moneda_id` fields
- Sales controller automatically generates numbers and sets default currency
- Stock movements are processed in model events (created, deleting)
- Background jobs handle stock alerts and expiration notifications
- Comprehensive logging for all stock operations and errors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FernandoPintoL) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
