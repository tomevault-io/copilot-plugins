## hermen-mes

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Manufacturing Execution System (MES) for Hermen Ltda., a Bolivian textile company. Native PHP 7.4+ application with MySQL database, focused on polyamide production line management (weaving, dyeing, quality control, inventory).

**Language:** All code comments and UI are in Spanish.

## Development Environment

- **Stack:** XAMPP (Apache + MySQL + PHP 7.4+)
- **Timezone:** America/La_Paz (UTC-4) - configured at both PHP and MySQL levels
- **Database:** mes_hermen (MySQL 5.7+, utf8mb4_unicode_ci)
- **No build system:** PHP files served directly, no bundling or minification

**Local URL:** `http://localhost/mes_hermen`
**Default login:** admin / admin123

## Architecture

### Database Connection (Singleton Pattern)
```php
require_once 'config/database.php';
$db = getDB();  // Returns PDO connection
```

### Helper Functions (defined in config/database.php)
- `getDB()` - Get PDO connection
- `isLoggedIn()` - Check if user is authenticated
- `hasRole($roles)` - Check user role(s): admin, coordinador, gerencia, tejedor, revisor, tintorero
- `redirect($url)` - Redirect relative to SITE_URL
- `sanitize($data)` - Clean user input
- `jsonResponse($data, $status)` - Return JSON and exit

### Session Variables
```php
$_SESSION['user_id']
$_SESSION['user_role']
$_SESSION['user_area']
$_SESSION['user_name']
```

### API Response Format
All APIs return JSON:
```json
{"success": true/false, "message": "...", "data": ...}
```

### Inventory Types (Critical IDs)
| ID | Code | Description |
|----|------|-------------|
| 1 | MP | Materias Primas (Raw Materials) |
| 2 | CAQ | Colorantes y Aux. Químicos (Dyes) |
| 3 | EMP | Empaque (Packaging) |
| 4 | ACC | Accesorios (Accessories) |
| 6 | PT | Productos Terminados (Finished Products) |
| 7 | REP | Repuestos (Spare Parts) |

## Critical System: Document Numbering

**DO NOT MODIFY** without reading `api/DOCUMENTACION_NUMERACION.md`.

Document numbers are generated via `api/obtener_siguiente_numero.php`:
- Format: `{OPERATION}-{INVENTORY}-{TYPE}-{YYYYMM}-{SEQUENCE}`
- Example: `IN-CAQ-C-202601-0001` (Ingreso, Colorantes, Compra)

**Two modes:**
- `modo=preview` - Shows next number WITHOUT consuming sequence (use in UI)
- `modo=commit` - Increments sequence (use only when saving)

**Implementation pattern for new inventory modules:**
```php
// In ingresos_*.php or salidas_*.php
case 'siguiente_numero':
    $_GET['tipo_inventario'] = '2';  // inventory ID
    $_GET['operacion'] = 'INGRESO';   // or 'SALIDA'
    $_GET['tipo_movimiento'] = $tipo;
    $_GET['modo'] = 'preview';
    include 'obtener_siguiente_numero.php';
    exit();
```

**Never use `file_get_contents()` for internal API calls** - use `include` with `$_GET` variables.

## Module Structure

Each module follows this pattern:
- `modules/{module_name}/` - UI pages
- `api/ingresos_{type}.php` - Inventory receipts
- `api/salidas_{type}.php` - Inventory dispatches
- `api/kardex_{type}.php` - Movement history
- `api/reportes_{type}.php` - Reports

**Adding new modules:**
1. Add navigation entry in `includes/header.php`
2. Create module folder under `modules/`
3. Create corresponding API files
4. Add inventory type to `$codigosInventario` in `obtener_siguiente_numero.php`

## Key Files

- `config/database.php` - DB connection, session config, helper functions
- `includes/header.php` - Sidebar navigation (update for new modules)
- `includes/footer.php` - Footer template
- `api/obtener_siguiente_numero.php` - Centralized document numbering
- `mes_hermen.sql` - Complete database schema with seed data

## Frontend

- Vanilla JavaScript (no frameworks)
- Chart.js for dashboard analytics
- Font Awesome 6.4 for icons
- Modal dialogs for forms
- Assets in `assets/css/`, `assets/js/`, `assets/img/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/g-meneses) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
