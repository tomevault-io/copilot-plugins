## requi

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Sistema de Requisiciones y Órdenes de Compra - A PHP-based purchase requisition and approval workflow system. Built with a custom MVC framework (not Laravel), using Azure AD for authentication and MySQL for persistence.

**Language**: PHP 8.0+  
**Database**: MySQL (via XAMPP)  
**Auth**: Azure AD OAuth2

## Commands

### Development Server
```powershell
composer serve  # PHP built-in server at localhost:8000
```

### Run Tests
```powershell
composer test  # Runs PHPUnit (tests in tests/ directory)
```

### Database
MySQL runs through XAMPP. Connection config in `config/database.php`, credentials in `.env`.

## Architecture

### Request Flow
1. All requests route through `public/index.php` (front controller)
2. Router (`App\Core\Router`) dispatches to controllers based on `routes/web.php`
3. Middlewares execute before controller actions (auth, CSRF, roles)
4. Controllers call Services for business logic
5. Services interact with Models for data persistence
6. Views are PHP templates rendered by controllers

### Directory Structure
- `app/Controllers/` - Handle HTTP requests, call services, render views
- `app/Services/` - Business logic layer (validation workflows, notifications)
- `app/Models/` - ActiveRecord-style data models extending `Model` base class
- `app/Middlewares/` - Request filters (AuthMiddleware, CsrfMiddleware, RoleMiddleware)
- `app/Helpers/` - Utility classes (Config, Session, View, Redirect)
- `app/Views/` - PHP template files
- `routes/web.php` - All route definitions with middleware groups
- `config/` - Configuration files and SQL migrations

### Key Patterns

**Models** extend `App\Models\Model`:
- Define `$table` and `$fillable` properties
- Use `find()`, `where()`, `create()`, `save()`, `delete()` for CRUD
- Static `query()` for raw SQL, `beginTransaction()`/`commit()`/`rollback()` for transactions

**Controllers** extend `App\Controllers\Controller`:
- Use `$this->view('path', $data)` to render views
- Use `$this->jsonResponse($data)` for API responses
- Use `$this->getUsuarioId()`, `$this->getUsuarioEmail()` for auth context
- Use `$this->isAdmin()`, `$this->isRevisor()` for role checks

**Routes** use fluent registration:
```php
$router->group(['prefix' => '/admin', 'middlewares' => ['RoleMiddleware:admin']], function($router) {
    $router->get('/usuarios', [AdminController::class, 'usuarios']);
    $router->post('/usuarios/{id}', [AdminController::class, 'updateUsuario']);
});
```

### Core Domain

**Requisición** (Purchase Requisition): Created by users, contains items, distributed across cost centers, goes through approval workflow.

**Flujo de Autorización** (Approval Flow):
1. `pendiente_revision` - Awaiting reviewer approval
2. `pendiente_autorizacion` - Awaiting cost center authorizer approval
3. `autorizado` - Fully approved
4. `rechazado` - Rejected at any stage

**Key Services**:
- `RequisicionService` - Create/update requisitions with items and distributions
- `AutorizacionService` - Handle approval workflow actions
- `FlujoValidacionService` - Manage validation flow state machine
- `NotificacionService` / `EmailService` - Send notifications

### Authorization Types
- **Centro de Costo** - Cost center managers approve spending from their budget
- **Método de Pago** - Payment method authorizers (for specific payment types)
- **Cuenta Contable** - Account authorizers (for specific expense accounts)
- **Revisor** - Initial reviewers who validate requisition data

## Conventions

- All text/labels in Spanish (Guatemala locale)
- Date format: `Y-m-d` (ISO)
- Timezone: `America/Guatemala`
- Currency: Supports multiple via `moneda` field
- CSRF protection required for all POST/PUT/DELETE routes (use `CsrfMiddleware`)
- Views should include CSRF token: `<input type="hidden" name="_token" value="<?= $_SESSION['csrf_token'] ?>">`
- PUT/DELETE simulated via POST with `<input type="hidden" name="_method" value="PUT">`

---
> Source: [b-gutierrez1/requi](https://github.com/b-gutierrez1/requi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
